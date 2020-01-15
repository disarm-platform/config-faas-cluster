# config-faas-cluster
Config and instructions for a DiSARM OpenFaas cluster from scratch.

## Setup

### Prepare server
1. Create new server(s)! Needs to be running Docker. Everything below is to be run on the _manager_ node.
1. Ensure firewall is open for required ports: see [Firewall](#Firewall) below
1. Clone and `cd` into this repo: `git clone https://github.com/disarm-platform/config-faas-cluster`
1. Ensure `docker-compose.yml`, `openfaas-docker-compose.yml` and `traefik.toml` contain references to the correct domain/subdomains.

### Configure Docker Swarm mode
1. Get the internal IP of the machine (e.g. from Google VM dashboard) or external (ideally is static)
1. Create a swarm with address from previous step: `docker swarm init --advertise-addr XXX.XXX.XXX.XXX`
1. Note down the 'join a swarm' command from step above - will be needed to add more nodes to the cluster

### Create Docker prerequisites
1. Create network `traefik-net` with overlay driver: `docker network create -d overlay --attachable traefik-net`
1. Add secrets for openfaas:
    ```sh
    echo "admin" | docker secret create basic-auth-user -
    echo "verysecretpassword" | docker secret create basic-auth-password -
    ```
    
### Start the stacks
1. Start the _proxy_ stack: `docker stack deploy -c docker-compose.yml rp`
1. Start the _OpenFaas_ stack: `docker stack deploy -c openfaas-docker-compose.yml func`
1. Add _secret_ called `ssl-cert`, to allow Squid's SSL certificate to be accesssed by functions: `docker secret create ssl-cert ./squid/cert/private.pem`. Must be run after certificate is created in `func` (OpenFaas) stack. **NOTE** If this fails, give `squid` a little time to startup.
### Logging

1. Install the [stackdriver monitoring agent ](https://cloud.google.com/monitoring/agent/install-agent) by running the commands: 

```
curl -sSO https://dl.google.com/cloudagents/install-logging-agent.sh
sudo bash install-logging-agent.sh
```

1. Create symbolic link  `traefik.conf` to the logging agent directory by running
    `sudo ln -s /home/disarm/config-faas-cluster/traefik.conf /etc/google-fluentd/config.d/traefik.conf`
    
1. Restart the agent by running the command `sudo service google-fluentd restart`


### Confirm is alive

1. Confirm https://traefik.srv.disarm.io is live and reachable, with a couple of _Frontends_ and a couple of _Backends_
1. Visit https://port.srv.disarm.io to create initial username and password

# Re-deploy functions from one installation to another

For example, to deploy all functions in `server1` to `server2`:

```
curl 'https://faas.server1.disarm.io/system/functions' -H 'authority: faas.server1.disarm.io' -H 'authorization: Basic dG9wc2VjcmV0dXNlcjphbmV3c2VjcmV0cGFzc3dvcmQ=' -H 'accept: application/json' > functions.json

# Deploy each with scaling
< functions.json | jq -r '.[] | .name,.image' | parallel -n 2 --dry-run 'faas deploy --image={2} --name={1} --gateway=https://faas.server2.disarm.io -l com.openfaas.scale.zero=true'

# Invoke each, to trigger the scaling-to-zero
< functions.json | jq -r '.[] | .name,.image' | parallel -n 2 --dry-run 'echo "" | faas invoke {1} --gateway=https://faas.server2.disarm.io'

```

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

