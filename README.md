# OReilly: Introduction to Kubernetes, GitOps, and Observability

## Create your cluster

For this demo, you can use any Kubernetes cluster (1.22 - 1.25), including kind and colima clusters.

You can use [kind](https://kind.sigs.k8s.io/docs/user/quick-start/) (Kubernetes in Docker) to create your cluster by running:

```sh
kind create cluster --name oreilly-kubernetes
```

Alternatively, you can use Colima to create a local Kubernetes cluster:

```sh
brew install colima

colima start --kubernetes --cpu 4 --memory 8 --profile oreilly-kubernetes
```

Make sure your cluster is ready before proceeding:

```sh
# this should return a node marked Ready
kubectl get nodes --watch
```

## Bootstrap Flux to your cluster

Install the Flux CLI:

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
  --branch=oreilly-kubernetes \
  --path=./clusters/oreilly-cluster \
  --token-auth \
  --personal
```

The output from the bootstrap should look like:

```sh
► connecting to github.com
► cloning branch "main" from Git repository "https://github.com/tiffanywang3/oreilly-kubernetes.git"
✔ cloned repository
► generating component manifests
✔ generated component manifests
✔ component manifests are up to date
► installing components in "flux-system" namespace
✔ installed components
✔ reconciled components
► determining if source secret "flux-system/flux-system" exists
► generating source secret
► applying source secret "flux-system/flux-system"
✔ reconciled source secret
► generating sync manifests
✔ generated sync manifests
✔ sync manifests are up to date
► applying sync manifests
✔ reconciled sync configuration
◎ waiting for Kustomization "flux-system/flux-system" to be reconciled
✔ Kustomization reconciled successfully
► confirming components are healthy
✔ helm-controller: deployment ready
✔ kustomize-controller: deployment ready
✔ notification-controller: deployment ready
✔ source-controller: deployment ready
✔ all components are healthy
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
 clusters/oreilly-cluster/flux-system/gotk-components.yaml | 6129 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 clusters/oreilly-cluster/flux-system/gotk-sync.yaml       |   27 +
 clusters/oreilly-cluster/flux-system/kustomization.yaml   |    5 +
 3 files changed, 6161 insertions(+)
 create mode 100644 clusters/oreilly-cluster/flux-system/gotk-components.yaml
 create mode 100644 clusters/oreilly-cluster/flux-system/gotk-sync.yaml
 create mode 100644 clusters/oreilly-cluster/flux-system/kustomization.yaml
```

You'll also see the commits that Flux made to your repo:

```sh
git log --oneline
```

## Contents of the OReilly Kubernetes Repository

This repo includes manifests from the fluxcd/flux2 GitHub repository, modified for the purposes of this presentation. You can find docs [here](https://fluxcd.io/flux/guides/monitoring/#install-flux-grafana-dashboards). 

The observability stack workloads are deployed via Flux HelmReleases, which the Flux Helm Controller reconciles within your cluster. HelmReleases allow users to declaratively use Helm. 

This repo includes HelmReleases, HelmRepositories, and Kustomizations for our observability stack components. We'll be using Prometheus, Grafana, Loki, Fluent Bit, and Weave GitOps. 

View the Grafana dashboards:

Navigate to localhost:3000, with user `admin`, and password `prom-operator`:
```sh
kubectl -n observability port-forward svc/kube-prometheus-stack-grafana 3000:80
```

You can browse the Grafana Dashboards and look for the ones defined in clusters/oreilly-cluster/observability/observability-config (Cluster Logs, Flux Cluster Stats, and Flux Control Plane).


### Add the Weave GitOps HelmRelease + HelmRepository

Install the Weave GitOps CLI

```sh
brew install weaveworks/tap/gitops
```

Create the Weave GitOps Dashboard:

```sh
# this password will be used for accessing the GitOps Dashboard
export PASSWORD=password

# from the root of your repository, run the following to create the commit to add the Dashboard manifests
gitops create dashboard ww-gitops \
  --password=$PASSWORD \
  --export > ./clusters/oreilly-cluster/weave-gitops-dashboard.yaml
```

The GitOps CLI will have added a new file to the specified path; edit the contents of the commit and push that update to your repo:

```sh
git add .
git commit -m "Add weave gitops dashboard"
git push
```

We can use the Flux CLI to automatically reconcile the contents of our latest commit:

```sh
flux reconcile kustomization flux-system --with-source
```

Review the contents of the flux-system namespace:

```sh
kubectl get pods -n flux-system
NAME                                       READY   STATUS    RESTARTS   AGE
helm-controller-7d9bb444c7-2jjfl           1/1     Running   0          4m58s
kustomize-controller-5c84554f7b-49fdc      1/1     Running   0          4m58s
notification-controller-64695f5b65-7zrjf   1/1     Running   0          4m58s
source-controller-7859746949-2g7bp         1/1     Running   0          4m58s
ww-gitops-weave-gitops-6cfb57f656-2jhb9    1/1     Running   0          4s
```

Login to the GitOps Dashboard by exposing the service and using the default user (admin) and password (password):

```sh
kubectl port-forward svc/ww-gitops-weave-gitops -n flux-system 9001:9001
```

## Reproducible, Auditable, Reliably Delivered Workloads

You'll notice throughout this workshop that we're never calling `kubectl apply` to deploy workloads to the cluster; all of the changes we've made to the cluster have been via Git, with Flux reconciling the desired state you've defined in Git to your running cluster.

We could completely delete our cluster, create a new cluster (into which Flux is bootstrapped/deployed), and once Flux comes up successfully, all of our workloads will be deployed.

You can delete your kind cluster with:

```sh
kind delete cluster --name oreilly-kubernetes
```

If you are running a Colima cluster, you can stop Colima with:

```sh
colima stop --profile oreilly-kubernetes
```

And to delete the Colima cluster, you can run:

```sh
colima delete --profile oreilly-kubernetes
```

Once your cluster has been deleted, create a new one!

If you prefer kind:

```sh
kind create cluster --name new-oreilly-kubernetes
```

If you prefer colima:

```sh
colima start --kubernetes --cpu 4 --memory 8 --profile new-oreilly-kubernetes
```

Make sure your kind cluster is ready before re-bootstrapping Flux:

```sh
export GITHUB_USER=$YOUR_GITHUB_USER
export GITHUB_TOKEN=$YOUR_GITHUB_TOKEN

flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=oreilly-kubernetes \
  --branch=oreilly-kubernetes \
  --path=./clusters/oreilly-cluster \
  --token-auth \
  --personal
```

Once Flux is pointed back to this repository, it will reconcile the workloads comprising the desired state. 

