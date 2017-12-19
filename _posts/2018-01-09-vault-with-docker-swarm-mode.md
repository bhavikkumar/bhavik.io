---
layout: post
title:  "Hashicorp Vault on Docker Swarm Mode"
date:   2018-01-09 10:00:00 +1200
---
In my last [post]({{ site.baseurl }}{% link _posts/2017-12-19-consul-with-docker-swarm-mode.md %}) the guide was for Hashicorp Consul running on a Docker Swarm Mode cluster. This post we are going to deploy a HA vault cluster using the Consul cluster as the backend storage.

At the end of this guide, we will have a 3 node Vault cluster deployed which has the following characteristics:
- Failover when active node goes offline
- Vault servers running on all manager nodes
- Vault using Consul cluster as the backend

## Vault Server Configuration
The following configuration must be in the same location on all of your manager nodes. I put it under `/opt/vault/vault.hcl` in my setup. This configuration allows Vault to connect to the local Consul server for the backend and listens on port 8200.

Do not use this configuration in production. In production enable TLS to protect the data in transit.
{%raw%}
```
backend "consul" {
  address = "127.0.0.1:8500"
  path = "vault"
  scheme="http"
}
listener "tcp" {
  address = "0.0.0.0:8200"
  tls_disable = 1
}
```
{%endraw%}

## Load Balancer Configuration
I am just going to give a high level configuration of what must be done when deploying the Vault cluster with failover. TCP Ports 8200 and 8201 must be routed to the Vault nodes, the health check can be performed on port 8200 with the path of /v1/sys/health. Healthy nodes will return back 200 or 429 if they are the standby node.

Vault may issue HTTP 307 redirects to the actual Vault node depending on your configuration. Your firewall rules must allow applications using the Vault cluster to access the load balancer and the individual Vault nodes on port 8200.

## Deploy Vault Server
Unlike the Consul cluster, I strongly recommend that Vault is not deployed as a Docker service as the container requires the ability to lock memory to prevent sensitive values from being swapped to disk. This is achieved by setting the IPC_LOCK flag when starting the container.

The variable of `$VAULT_CLUSTER_ADDR` is the DNS address of the load balancer which I am using to route traffic to the unsealed Vault server.
```
docker run -d --name vault --net=host --restart=always -e 'VAULT_REDIRECT_INTERFACE=eth0' -e VAULT_CLUSTER_ADDR=$VAULT_CLUSTER_ADDR -v /opt/vault:/config --cap-add IPC_LOCK vault server -config=/config/vault.hcl
```

## Vault Initialisation and Unsealing
When the Vault servers are first started, they will not be in a usable state. The Vault cluster needs to be initialised and each node unsealed. Depending on your security requirements it is possible to automate this process, I recommend storing the unseal and root token encrypted if this is the route you are going to take.

It is possible to do this via the HTTP API as [documented here](https://www.vaultproject.io/api/system/init.html) and [here](https://www.vaultproject.io/api/system/unseal.html).
