# config-faas-cluster
Config and instructions for a Locational OpenFaas cluster

- create new server! maybe too obvious, but is a useful place to start!

- clone this repo (see next point - if repo is private?)
`git clone https://github.com/locational/config-faas-cluster`

- `cd` into `config-faas-cluster` folder

- create ACME storage file for https certificates with 
  `touch acme.json`

- get the external IP (not floating or static) of the machine

- create a swarm
`docker swarm init --advertise-addr XXX.XXX.XXX.XXX`

- note down the 'join a swarm' command from step above
  
- create network traefik with overlay driver
`docker network create -d overlay --attachable traefik-net`


- start the revrse proxy and 
`docker stack deploy -c docker-compose.yml rp`

- once confirmed traefik running fine, remove the port entry from `docker-compose.yml` to avoid leaving traefik dashboard exposed

- add secrets for openfaas
`echo "admin" | docker secret create basic-auth-user -`
`echo "verysecretpassword" | docker secret create basic-auth-password -`

- Login to portainer and create a stack from the openfaas template
- On the openfaas stack edit the docker-compose.yml and replace it with the openfaas docker compose file from this repo
  and edit the `traefik.frontend.rule` label to be the openfaas domain name on the gateway service.
  
- For monitoring, add the datadog agent from the portainer templates with an API key from [this](https://app.datadoghq.com/account/settings#api) page of the datadog website
  
  
