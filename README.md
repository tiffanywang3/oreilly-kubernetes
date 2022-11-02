# OReilly: Introduction to Kubernetes, GitOps, and Observability

## Create your cluster

For this demo, you can use any Kubernetes cluster (1.22 - 1.25).

You can use [kind](https://kind.sigs.k8s.io/docs/user/quick-start/) (Kubernetes in Docker) to create your cluster by running:

```sh
kind create cluster
```

Make sure your cluster is ready before proceeding:

```sh
# this should return a node marked Ready
kubectl get nodes
```

## Bootstrap Flux to your cluster

Install the Flux CLI

```sh
brew install fluxcd/tap/flux
```

Bootstrap Flux:

```sh
export GITHUB_USER=$YOUR_GITHUB_USER
export GITHUB_TOKEN=$YOUR_GITHUB_TOKEN

flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=oreilly-kubernetes \
  --branch=main \
  --path=./clusters/kind-cluster \
  --token-auth \
  --personal
```

## Add the Weave GitOps HelmRelease + HelmRepository

Install the Weave GitOps CLI

```sh
brew install weaveworks/tap/gitops
```

Bootstrap Weave GitOps Dashboard:

```sh
# this password will be used for accessing the GitOps Dashboard
export PASSWORD=password

# from the root of your repository, run the following to create the commit to add the Dashboard manifests
gitops create dashboard ww-gitops \
  --password=$PASSWORD \
  --export > ./clusters/kind-cluster/weave-gitops-dashboard.yaml
```

## Create Flux Kustomizations for your Observability Stack

