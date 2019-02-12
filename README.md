# config-faas-cluster
Config and instructions for a Locational OpenFaas cluster

1. create new server! maybe too obvious, but is a useful place to start!

1. clone this repo (see next point - if repo is private?)
`git clone https://github.com/locational/config-faas-cluster`

1. `cd` into `config-faas-cluster` folder

1. get the external IP (not floating or static) of the machine

1. create a swarm
`docker swarm init --advertise-addr XXX.XXX.XXX.XXX`

1. note down the 'join a swarm' command from step above
  
1. create network traefik with overlay driver
`docker network create -d overlay --attachable traefik-net`


1. start the revrse proxy and 
`docker stack deploy -c docker-compose.yml rp`

1. once confirmed traefik running fine, remove the port entry from `docker-compose.yml` to avoid leaving traefik dashboard exposed, and redeploy stack as above(`docker stack deploy...`)

1. visit port.srv.locational.io and create initial username and password

1. add secrets for openfaas
`echo "admin" | docker secret create basic-auth-user -`
`echo "verysecretpassword" | docker secret create basic-auth-password -`

1. Login to portainer and create and deploy a stack from the openfaas template

1. On the openfaas stack edit the docker-compose.yml and replace it with the openfaas docker compose file from this repo
  
1. For monitoring, add the datadog agent from the portainer templates with an API key from [this](https://app.datadoghq.com/account/settings#api) page of the datadog website
  
  
