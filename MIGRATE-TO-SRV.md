# Migration steps

## Changing domain names

1. Update readme by changing all urls of the form `https://*.srv2.disarm.io` to `https://*.srv.disarm.io`

2. Update `docker-compose.yml` by changing the following lines:

    - `command: --api --docker --docker.swarmmode --docker.domain=srv2.disarm.io --docker.watch` to
    `command: --api --docker --docker.swarmmode --docker.domain=srv.disarm.io --docker.watch`

    - `command: --docker --docker.swarmmode --docker.domain=srv2.disarm.io --docker.watch` to
    `command: --docker --docker.swarmmode --docker.domain=srv.disarm.io --docker.watch`
    - `- "traefik.frontend.rule=Host:traefik.srv2.disarm.io"` to `- "traefik.frontend.rule=Host:traefik.srv.disarm.io"`

3. Update `openfaas-docker-compose.yml` by changing the following lines:

   `- "traefik.basic.frontend.rule=Host:faas.srv2.disarm.io"` to `- "traefik.basic.frontend.rule=Host:faas.srv.disarm.io"`

## Re-deploy functions

Run following bash script to deploy all functions in srv to srv2

```
   curl 'https://faas.srv.disarm.io/system/functions' -H 'authority: faas.srv.disarm.io' -H 'authorization: Basic dG9wc2VjcmV0dXNlcjphbmV3c2VjcmV0cGFzc3dvcmQ=' -H 'accept: application/json' | jq -c  '.[]| {name:.name,image:.image}' > functions.json
   jq -c '.name,.image' functions.json | xargs -n2 sh -c echo '$1 and $0'
```