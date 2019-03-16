# config-faas-cluster
Config and instructions for a Locational OpenFaas cluster from scratch.

## Setup

1. Create new server!
1. Clone this repo: `git clone https://github.com/locational/config-faas-cluster` (will need to use your GitHub login)
1. `cd` into `config-faas-cluster` folder
1. Get the external IP (not floating or static) of the machine
1. Create a swarm: `docker swarm init --advertise-addr XXX.XXX.XXX.XXX`
1. Ensure firewall is open for ports: 2377, 4789, 7946 (from [here](https://www.digitalocean.com/community/tutorials/how-to-configure-the-linux-firewall-for-docker-swarm-on-centos-7)): see Firewall below
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

## Firewall

```
ufw allow 22/tcp
ufw allow 2376/tcp
ufw allow 2377/tcp # Only for manager node
ufw allow 7946/tcp
ufw allow 7946/udp
ufw allow 4789/udp
ufw reload
systemctl restart docker
```

## Cleanup
1. Once confirmed stack is up and Traefik is running fine, remove the port entry from `docker-compose.yml` to avoid leaving traefik dashboard exposed, and redeploy stack as above(`docker stack deploy...`).

## Monitoring  
1. Optionally, add the Portainer Agent stack from the _app templates_, to help Portainer see what's happening on all nodes in the cluster.


## Datadog

Using it to monitor both system and Docker contexts, with processes, containers and logs.

1. Install agent: `DD_API_KEY=<DD_KEY!> bash -c "$(curl -L https://raw.githubusercontent.com/DataDog/datadog-agent/master/cmd/agent/install_script.sh)"` [instructions](https://app.datadoghq.com/account/settings#agent/ubuntu)
2. Add agent to docker and system-d groups: `usermod -a -G docker dd-agent` and `usermod -a -G systemd-journal dd-agent`
3. Configure logging: `mkdir conf.d/journal.d`, edit `conf.yaml` [instructions](https://docs.datadoghq.com/integrations/journald/):
    ```yaml
    logs:
        - type: journald
        path: /var/log/journal/
    ```
4. Configure Docker: edit `conf.d/docker.d/docker_daemon.yaml`:
    ```yaml
    init_config:

    instances:
        - url: "unix://var/run/docker.sock"
        new_tag_names: true
        
    logs:
        - type: docker 
    ```
4. Edit `datadog.yaml`, adding to the top:
    ```yaml
    process_config:
        enabled: "true"

    logs_enabled: true

    listeners:
        - name: docker

    config_providers:
        - name: docker
          polling: true
    ```
5. Restart the agent and check status: `sudo service datadog-agent restart` and `sudo service datadog-agent status`
