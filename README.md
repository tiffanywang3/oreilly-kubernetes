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

Pull commits made to your repository by Flux:

```sh
git pull 
```

You should see the Flux manifests got added to the path that you specified:

```sh
-> git pull
remote: Enumerating objects: 14, done.
remote: Counting objects: 100% (14/14), done.
remote: Compressing objects: 100% (8/8), done.
remote: Total 13 (delta 0), reused 13 (delta 0), pack-reused 0
Unpacking objects: 100% (13/13), 29.63 KiB | 5.93 MiB/s, done.
From https://github.com/tiffanywang3/oreilly-kubernetes
   31d23a9..4cc7965  main       -> origin/main
Updating 31d23a9..4cc7965
Fast-forward
 clusters/kind-cluster/flux-system/gotk-components.yaml | 6129 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 clusters/kind-cluster/flux-system/gotk-sync.yaml       |   27 +
 clusters/kind-cluster/flux-system/kustomization.yaml   |    5 +
 3 files changed, 6161 insertions(+)
 create mode 100644 clusters/kind-cluster/flux-system/gotk-components.yaml
 create mode 100644 clusters/kind-cluster/flux-system/gotk-sync.yaml
 create mode 100644 clusters/kind-cluster/flux-system/kustomization.yaml
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

The GitOps CLI will have added a new file to the specified path; commit and push that update to your repo:

```sh
git add .
git commit -m "Add weave gitops dashboard"
git push
```



## Create Flux Kustomizations for your Observability Stack

