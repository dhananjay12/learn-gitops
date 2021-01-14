# Flux Getting Started Guide

# 1 - Get a Kubernetes Cluster

Local or any cloud provider 

# 2 - Setup  fluxctl

[https://docs.fluxcd.io/en/latest/references/fluxctl/](https://docs.fluxcd.io/en/latest/references/fluxctl/)
Try
```
fluxctl
```

Make sure you are pointing to the kubernetes cluster you want to use
```
kubectl config current-context
kubectl get nodes
```
Create the flux namespace:
```
kubectl create ns flux

```
Install flux into this namespace. We want to hook up flux to the git-repo we want to control. 

```
export GHUSER="YOURUSER"
fluxctl install \
--git-user=${GHUSER} \
--git-email=${GHUSER}@users.noreply.github.com \
--git-url=git@github.com:${GHUSER}/learn-gitops \
--git-branch=master \
--git-path=demo-app,nginx \
--namespace=flux | kubectl apply -f -
```
`git-path` is the folder structure you want flux to synchronize with. Flux will recursively go through these directories
structure. It will see if there is any change in these directories and apply it to the cluster. 


Wait for Flux to start:
```
kubectl -n flux rollout status deployment/flux
```
When we run fluxctl, it will be run against a specific namespace and since we have intalled flux in the flux namespace.
To do that we created and env variable so that fluxctl commands runs on that namespace.

```
export FLUX_FORWARD_NAMESPACE=flux
fluxctl list-workloads -a
```
or supply the namespace flux is installed in via

```
fluxctl list-workloads --k8s-fwd-ns flux -a
```

At first there wont be any workloads yet, because you need to give write access to your Github repository. You can see
the same in the logs of flux pod.

Note: Be aware that exposing the Flux API in this way is a security hole, because it can be accessed without authentication. See the docs

We need to add the deployment key from Flux to the Github Deployed keys. Get the key from flux using:

```
fluxctl identity
```
Copy the above one to your Github. 

By default, flux will check this git repo every 5 mins. If you want to manually sync, run the following command:

```
fluxctl sync
```

# Remove flux pods from 

```
fluxctl install \
--git-user=${GHUSER} \
--git-email=${GHUSER}@users.noreply.github.com \
--git-url=git@github.com:${GHUSER}/learn-gitops \
--git-branch=master \
--git-path=demo-app,nginx \
--namespace=flux | kubectl delete -f -
```

# Delete a namespace

```
kubectl delete namespaces <insert-some-namespace-name>
```

# Flux for Docker Regsitry

Flux can also look into - Automated deployment of new container image, which means it can update the git and then deploy.
For every workload, you need to create a policy in order to tell flux how it should drive changes to the container 
registry. 

By default, the main feature of flux is to sync with git, but if you want to monitor the container registry as well, thats
also an option. You can do it using annotation in metadata of your deployment file. 

```
  annotations:
    fluxcd.io/automated: 'true'
```

You can follow the semversion too. Refer the docs.


```
fluxctl list-workloads -a
```

```
fluxctl list-workloads --namespace=frontend
```

[Slides](https://docs.google.com/presentation/d/1dTxUJL1hi9hnDvSF7HlW3aaoK2Q0GkW7dfQUYHlteJk/edit?usp=sharing)

