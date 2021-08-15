# flux-s3-exposer

Here resides all flux resources need to bring up a s3-exposer (with values for my own deployment)

## Usage

1. Fork this repository
1. Generate a github token and export it as an enviroment variable
1. Run the following command replacing the `<>` with the correct values:
```
flux bootstrap github --owner=<Your account> --repository=flux-s3-exposer --private=false --personal=true
```
After a couple of minutes you will be able to see the bucket content on the url specified

## Check
```
$ flux get sources git
NAME       	READY	MESSAGE                                                        REVISION                                     	SUSPENDED
flux-system	True 	Fetched revision: main/3164a370033061005f2e50c0d14e48c357772632main/3164a370033061005f2e50c0d14e48c357772632	False
s3-exposer 	True 	Fetched revision: main/9e0c73ee267d249ac0ef79dad843e42b51c6e040main/9e0c73ee267d249ac0ef79dad843e42b51c6e040	False
```
```
$ flux get helmreleases
NAME              	READY	MESSAGE                         	REVISION	SUSPENDED
cert-manager      	True 	Release reconciliation succeeded	v1.5.0  	False
external-dns      	True 	Release reconciliation succeeded	5.4.0   	False
ingress-controller	True 	Release reconciliation succeeded	0.10.0  	False
s3-exposer        	True 	Release reconciliation succeeded	0.1.0   	False
```

## What's going on under the hood

As soon as you install flux on the cluster it will do the following steps:
1. Will install all the resources and CRDS used by flux.
1. Create a new GitRepository pointing to this exact repository
1. Create a kustomization defined by kustomization.yaml inside this repository

When the flux pods are ready they will do the follwing steps:
1. Create HelmRepositories entrys for bitnami, nginx and jetstack (source of cert-manager chart)
1. Create a GitRepository pointing to https://github.com/Daklon/s3-exposer-helm where the s3-exposer helm chart resides
1. Install nginx ingress controller using the values defined on this repository (i.e. log-format-escape-json: "true")
1. Install external-dns using the values defined on this repository (i.e zoneType: public)
1. Install cert-manager to be able to issue certificates with let's encrypt

After all components are ready flux will install s3-exposer helm chart. Flux will wait for the other charts because they are defined as a dependecy for s3-exposer.

