## Document suggestions for OpenFaas upgrades
To improve the perfomance of the OpenFaaS server, the OpenFaaS framework will have to be updated to support scalling to and from zero. Scaling from zero is enabled by default on functions that have the `scale_from_zero` label. Scalling to zero is enabled by adding a `faas-idler` service into the docker compose.

### Notes on deploying functions that scale to and from zero
1. Deployment of functions should have the `scale_from_zero` and `com.openfaas.scale.zero` labels set to `true` or set both flags on the stack.yml under labels in each function.
1. `scale_from_zero` tells OpenFaaS to scale up the function when it is invoked and has been scaled to zero replicas.
1. `com.openfaas.scale.zero` tells the `faas-idler` service that the function can be scaled to zero when it is idle.

To implement the scaling on our current OpenFaaS instance , all the functions will have to be redeployed with the labels.
