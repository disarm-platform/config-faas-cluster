# config-faas-cluster
Config and instructions for a Locational OpenFaas cluster from scratch.

## Setup

1. Create new server!
1. Clone this repo (see next point - if repo is private?): `git clone https://github.com/locational/config-faas-cluster`
1. `cd` into `config-faas-cluster` folder
1. Get the external IP (not floating or static) of the machine
1. Create a swarm: `docker swarm init --advertise-addr XXX.XXX.XXX.XXX`
1. Ensure firewall is open: `ufw allow 2377` or similar
1. Note down the 'join a swarm' command from step above - will be needed to add more nodes to the cluster
1. Create network traefik with overlay driver: `docker network create -d overlay --attachable traefik-net`
1. Start the stack: `docker stack deploy -c docker-compose.yml rp`
1. Confirm https://traefik.srv.locational.io is live and reachable, with a couple of _Frontends_ and a couple of _Backends_
1. Visit https://port.srv.locational.io to create initial username and password
1. Add secrets for openfaas:
    ```sh
    echo "admin" | docker secret create basic-auth-user -
    echo "verysecretpassword" | docker secret create basic-auth-password -
    ```
1. Login to https://port.srv.locational.io and create and deploy an **OpenFaaS** stack from the _app templates_.
1. On the **OpenFaaS** stack _editor_, replace the `docker-compose.yml` content with the `openfaas-docker-compose.yml` file in this repo

## Cleanup
1. Once confirmed stack is up and Traefik is running fine, remove the port entry from `docker-compose.yml` to avoid leaving traefik dashboard exposed, and redeploy stack as above(`docker stack deploy...`).

## Monitoring  
1. For monitoring, add the datadog agent from the portainer templates with an API key from [this](https://app.datadoghq.com/account/settings#api) page of the datadog website. There is a customised `datadog.yml` file included in this repo that also activates logging.
1. Optionally, add the Portainer Agent stack from the _app templates_, to help Portainer see what's happening on all nodes in the cluster.
  
