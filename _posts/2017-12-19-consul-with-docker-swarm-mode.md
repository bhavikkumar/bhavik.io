---
layout: post
title:  "Hashicorp Consul on Docker Swarm Mode"
date:   2017-12-19 10:00:00 +1200
---
I've been playing around with Docker Swarm Mode since it is simpler to understand than Kubernetes. I have always wanted to deploy Hashicorp Consul for service registration, discovery and configuration management. While researching how to deploy a Consul cluster using Docker Swarm Mode, it became apparent most of the guides were for Docker 1.12 or required the use of a seed Consul server. Majority of the time, this seed server was treated like a pet which I definitely not what I wanted.

At the end of this guide, we will have a 3 node Consul cluster deployed which has the following characteristics:
- No seed Consul server
- Self-Healing (depending on how you deployed the infrastructure)
- Consul Servers on all manager nodes
- Consul Agents on all worker nodes

The easiest way to try this guide out is by deploying [Docker for AWS](https://docs.docker.com/docker-for-aws/) with 3 Managers and 1 Worker.

## Consul Server Configuration
The following configuration must be in the same location on all of your manager nodes. I put it under `/opt/consul/config.json`, the configuration below comes from my CloudFormation template and will need to be adjusted for your use case. I strongly recommend setting up TLS, ACLs and backup for `/consul/data` in production setups.

{% raw %}
```
{
  "advertise_addr" : "{{ GetInterfaceIP \"eth0\" }}",
  "bind_addr": "{{ GetInterfaceIP \"eth0\" }}",
  "client_addr": "0.0.0.0",
  "data_dir": "/consul/data",
  "datacenter": "${AWS::Region}",
  "leave_on_terminate" : true,
  "retry_join" : [
    "consul.server"
  ],
  "server_name" : "server.${AWS::Region}.consul",
  "skip_leave_on_interrupt" : true,
  "bootstrap_expect": ${ManagerClusterSize},
  "server" : true,
  "ui" : true,
  "autopilot": {
    "cleanup_dead_servers": true
  },
  "disable_update_check": true,
  "log_level": "warn",
  "encrypt": "${EncryptionToken}"
}
```
{% endraw %}

## Consul Agent Configuration
The following configuration is for agents which run on the worker nodes. As with the Consul server configuration, I put this under `/opt/consul/config.json` on my worker nodes. The encryption token here has to be the same as one configured in the Consul servers.
{% raw %}
```
{
  "advertise_addr" : "{{ GetInterfaceIP \"eth0\" }}",
  "bind_addr": "{{ GetInterfaceIP \"eth0\" }}",
  "client_addr": "0.0.0.0",
  "data_dir": "/consul/data",
  "datacenter": "${AWS::Region}",
  "leave_on_terminate" : true,
  "retry_join" : [
    "consul.server"
  ],
  "server_name" : "agent.${AWS::Region}.consul",
  "skip_leave_on_interrupt" : false,
  "server" : false,
  "ui" : false,
  "disable_update_check": true,
  "log_level": "warn",
  "encrypt": "${EncryptionToken}"
}
```
{% endraw %}


## Overlay Network
We need to create our own overlay network, I found the default swarm overlay network produced inconsistent results when starting the cluster. The following command will create another overlay network for us to use.
```
sudo docker network create -d overlay --subnet=192.168.0.0/16 default_net
```

## Docker Compose File
This is the key to a successful deployment of the Consul cluster. It uses the network which we just created and a alias `consul.server` for our consul servers. Running `nslookup consul.server` inside any container which is attached to `default_net` will result in us getting a list of all of the containers running as Consul servers.

The Consul servers and agents are deployed in global mode. If you spin up more managers or workers after the cluster is established, the additional consul nodes will automatically connect to the cluster.
```
---
version: '3.3'
networks:
  default_net:
    external: true
services:
  # Deploy the consul server instances
  server:
    image: consul:latest
    networks:
      default_net:
        aliases:
          - consul.server
    # Start the consul server with the given configuration
    command: "consul agent -config-file /consul/config/config.json"
    # Expose port 8500 so we can access the UI and allow connections across datacenters.
    ports:
      - target: 8500
        published: 8500
        mode: host
    # Mount the configuration and data volumes to the container.
    volumes:
      - /opt/consul:/consul/config
      - /opt/consul/data:/consul/data
    # Deploy the consul server on all servers which are managers.
    # Use DNS Round Robin instead VIP for discovery. This ensures we get all running
    # consul server instances when querying consul.server
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
      placement:
        constraints:
          - node.role == manager
  # Deploy the consul agent instances
  agent:
    image: consul:latest
    networks:
      default_net:
        aliases:
          - consul.server
     # Start the consul agent with the given configuration          
    command: "consul agent -config-file /consul/config/config.json"
    ports:
      - target: 8500
        published: 8500
        mode: host
    # Mount the configuration and data volumes to the container.
    volumes:
      - /opt/consul:/consul/config
      - /opt/consul/data:/consul/data
    # Deploy the consul agent on all servers which are workers.
    # Use DNS Round Robin instead VIP for discovery.  
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
      placement:
        constraints:
          - node.role == worker
```          
## Deploy the Consul Cluster
The following command will deploy the entire Consul cluster.
```
docker stack deploy -c /opt/consul/compose.yaml consul
```
## Accessing the Consul Cluster
The last part to the guide is how to access the various components of the cluster.
### Consul UI
The UI can be access by connecting to any of the managers using `http://<manager-ip>:8500/ui`.
### Consul API
The API can be access by connecting to any of the nodes using `http://<node-ip>:8500/v1/`

## Service Registration/Discovery
The normal setup is the application accessing the Consul node which is running on the same machine. However, with containers in the mix, the way to connect differs. If the application is not deployed as a docker service, then you can still register the service in Consul using `<node-ip>:8500` or `localhost:8500` but health checks will fail.

If the deployed as a service which is most likely scenario when using Docker Swarm Mode. Then you can use the ingress gateway to connect to Consul, which for my configuration was `172.17.0.1:8500`. The service must be attached to `default_net` for health checks to work correctly.
