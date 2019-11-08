# config-faas-cluster
Config and instructions for a DiSARM OpenFaas cluster from scratch.

## Setup

1. Create new server(s)! Everything below is to be run on the _manager_ node.
1. Clone this repo: `git clone https://github.com/disarm-platform/config-faas-cluster` (will need to use your GitHub login)
1. `cd` into `config-faas-cluster` folder
1. Get the internal IP of the machine (e.g. from Google VM dashboard) or external (ideally is static)
1. Create a swarm with address from previous step: `docker swarm init --advertise-addr XXX.XXX.XXX.XXX`
1. Note down the 'join a swarm' command from step above - will be needed to add more nodes to the cluster
1. Ensure firewall is open for required ports: see [Firewall](#Firewall) below
1. Create network `traefik-net` with overlay driver: `docker network create -d overlay --attachable traefik-net`
1. Ensure `docker-compose.yml`, `openfaas-docker-compose.yml` and `traefik.toml` contain references to the correct domain/subdomains.
1. Start the stack: `docker stack deploy -c docker-compose.yml rp`
1. Confirm https://traefik.srv.disarm.io is live and reachable, with a couple of _Frontends_ and a couple of _Backends_
1. Visit https://port.srv.disarm.io to create initial username and password
1. Add secrets for openfaas:
    ```sh
    echo "admin" | docker secret create basic-auth-user -
    echo "verysecretpassword" | docker secret create basic-auth-password -
    ```
1. Login to https://port.srv.disarm.io and create and deploy an **OpenFaaS** stack from the _app templates_.
1. On the **OpenFaaS** stack _editor_, replace the `docker-compose.yml` content with the `openfaas-docker-compose.yml` file in this repo
1. Add `AUTH_URL` environmental variable to the **OpenFaas** stack, set value to: http://basic-auth-plugin:8080/validate

## Firewall

```
22/tcp
2376/tcp
2377/tcp
7946/tcp
7946/udp
4789/udp
```

At least 2377, 4789, 7946 (from [here](https://www.digitalocean.com/community/tutorials/how-to-configure-the-linux-firewall-for-docker-swarm-on-centos-7)) and also maybe list [here](https://gist.github.com/BretFisher/7233b7ecf14bc49eb47715bbeb2a2769)

## Cleanup
1. Once confirmed stack is up and Traefik is running fine, remove the port entry from `docker-compose.yml` to avoid leaving traefik dashboard exposed, and redeploy stack as above(`docker stack deploy...`).

## Monitoring  
1. Optionally, add the Portainer Agent stack from the _app templates_, to help Portainer see what's happening on all nodes in the cluster.

