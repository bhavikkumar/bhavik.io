---
layout: post
title:  "HAProxy with Consul Template"
date:   2018-03-20 10:00:00 +1200
---
The last two posts have been about Consul and Vault but the final piece missing is getting traffic in to the services which have are registered in Consul for service discovery. For this post we are going to be using HAProxy with Consul template to define the configuration required for ingress. There are examples for Apache, Nginx and Varnish in the consul template git repo. Traefik is also becoming a more popular reverse proxy and load balancer for microservices, it has built in support for a number of backends including Consul.

At the end of this guide, we will have HAProxy deployed on every node with the following:
- Services exposed by tags
- Routing done by versions
- Deployed as a Docker Swarm Mode service

## HAProxy Template Configuration
The following is the haproxy.json file which I created for consul template. It defines the location of the template, the destination of the HAProxy configuration and the command to run HAProxy with.
{%raw%}
```
template {
  source = "/tmp/haproxy.ctmpl"
  destination = "/etc/haproxy/haproxy.cfg"
  command = "/bin/sh -c 'haproxy -D -f /etc/haproxy/haproxy.cfg -p /run/haproxy-lb.pid -sf $(cat /run/haproxy-lb.pid)'"
}
```
{%endraw%}

## Template Configuration
The template gets information from Consul and builds a HAProxy configuration. The tags which are available for use can be found in the README.md on the [consul-template github page](https://github.com/hashicorp/consul-template). You can also define keys such as maximum connections and use them to build the configuration for a great amount of flexibility.

The following is haproxy.ctmpl I use for my ingress. The template creates a routing for services based on the version and edge tags and the service name for the path. For example, if we have genre service deployed with the tags of `version=1.1` and `edge` then the template will generate a rule which will route incoming traffic from http://hostname/v1.1/genre to http://container-name/genre. All services which are not tagged as edge are not exposed externally from the cluster.

{%raw%}
```
global
    log 127.0.0.1   local0
    log 127.0.0.1   local1 notice
    debug
    stats timeout 30s
    maxconn 1024

defaults
    log global
    option httplog
    option dontlognull
    mode http
    timeout connect 5000
    timeout client  50000
    timeout server  50000

frontend http-in
    bind 0.0.0.0:80
    monitor-uri /healthcheck{{ range $i, $service := services }}{{ range $tag := .Tags }}{{ if $tag | regexMatch "^version=.+" }}{{ $version := index (. | split "=") 1 }}{{ if $service.Tags | contains "edge" }}
    # Edge for {{ $service.Name }}, Version: {{ $version }}
    acl {{ $service.Name }}{{ $version }} path_beg /v{{ $version }}/{{ $service.Name }}
    use_backend {{ $service.Name }}{{ $version }} if {{ $service.Name }}{{ $version }}
    {{ end }}{{ end }}{{ end }}{{ end }}

{{ range $i, $service := services }}{{ range $tag := .Tags }}{{ if $tag | regexMatch "^version=.+" }}{{ $version := index (. | split "=") 1 }}{{ if $service.Tags | contains "edge" }}
# Backend for {{ $service.Name }}, Version: {{ $version }}
backend {{ $service.Name }}{{ $version }}
    mode http
    balance roundrobin
    option forwardfor
    option httpchk GET /healthcheck
    http-check expect ! rstatus ^5
    default-server inter 2s fall 1 rise 2
    reqrep ^([^\ ]*\ /)v{{ $version }}/{{ $service.Name }}[/]?(.*)     \1{{ $service.Name }}/\2{{range $c,$d:=service $service.Name}}{{ if $d.Tags | contains "edge" }}{{ if $d.Tags | contains (printf "%s%s" "version=" $version) }}
    server {{.Address}} {{.Address}}:{{.Port}} check
    {{ end }}{{ end }}{{ end }}{{ end }}{{ end }}{{ end }}{{ end }}

listen stats
    bind 0.0.0.0:1936
    stats enable
    stats uri /
    stats hide-version
    stats auth admin:changeme
```
{%endraw%}    

## HAProxy
### Dockerfile
I created my own HA Proxy container with consul template in it, the ones available were using out dated versions of consul template.

{%raw%}
```
FROM haproxy:1.8-alpine
LABEL Description="Runs consul template and runs haproxy based on the generated configuration, based on sirile/haproxy" Version="0.1"

ENV CONSUL_TEMPLATE_VERSION=0.19.4

# Update wget to get support for SSL
RUN apk --update add wget

# Download consul-template
RUN wget --no-check-certificate https://releases.hashicorp.com/consul-template/${CONSUL_TEMPLATE_VERSION}/consul-template_${CONSUL_TEMPLATE_VERSION}_linux_amd64.tgz \
  && tar xfz consul-template_${CONSUL_TEMPLATE_VERSION}_linux_amd64.tgz \
  && mv consul-template /usr/bin/consul-template \
  && rm consul-template_${CONSUL_TEMPLATE_VERSION}_linux_amd64.tgz

RUN touch /run/haproxy.sock
RUN chmod 777 /run/haproxy.sock

COPY haproxy.json /tmp/haproxy.json
COPY haproxy.ctmpl /tmp/haproxy.ctmpl

ENTRYPOINT ["consul-template"]
CMD ["-help"]
```
{%endraw%}

### Run Container with Template
The best way to run the container is by mounting the template configurations from the host machine into the container, this allows the template to be updated easily and then passing in the command `-config=/tmp/haproxy.json -consul-addr=consul.server:8500`. The consul address is only available to the swarm cluster as we set this alias up when deploying consul.

The configuration I use is the following which deploys the haproxy with consul template on all nodes. It exposes port 80 for traffic to go to services and port 1936 for the management interface.

My recommendation is using port 443 with the appropriate TLS certificate for your domain.
{%raw%}
```
---
version: '3.3'
networks:
  default_net:
    external: true
services:
  server:
    image: bhavikk/haproxy-consul-template
    networks:
      default_net:
        aliases:
          - haproxy.server
    command: "-config=/tmp/haproxy.json -consul-addr=consul.server:8500"
    ports:
      - target: 80
        published: 80
        mode: host
      - target: 1936
        published: 1936
        mode: host
    volumes:
      - /opt/haproxy:/tmp
    deploy:
      mode: global
      endpoint_mode: dnsrr
      update_config:
        parallelism: 1
        failure_action: rollback
        delay: 30s
      restart_policy:
        condition: any
        delay: 5s
        window: 120s
```        
{%endraw%}

## Load Balancer Configuration
The load balancer configuration is simple in this scenario, we allow all nodes to receive traffic in bound to port 80. For port 1936, I have another load balancer limits the IP addresses can access the management interface.
