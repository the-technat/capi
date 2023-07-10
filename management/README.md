# Management Cluster

We use the "Bootstrap & Pivot" approach for our management cluster.

## Infrastructure Provider Prerequisites

Before you start with anything, you need to decide for an infrastructure provider that will hosts your permanent management cluster. For me this is [Hetzner](https://hetzner.de), with it's corresponding [cluster-api-provider-hetzner](https://github.com/syself/cluster-api-provider-hetzner). 

All providers need some sort of prerequisites before you can use them, for CAPH, they can be found [here](https://github.com/syself/cluster-api-provider-hetzner/blob/main/docs/topics/preparation.md).


## Temporary Kind Cluster

If the prerequisites are met, we deploy a temporary kind cluster:

```
go install sigs.k8s.io/kind@latest
cat > /tmp/caph-config.yaml <<EOF 
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: caph
nodes:
- role: control-plane
- role: worker
EOF
kind create cluster --config /tmp/caph-config.yaml
```

Make sure all nodes are ready using `kubectl get nodes`.

## CAPI Init on Temp Cluster

Next we install CAPI with it's providers on kind. For this you need the [clusterctl](https://cluster-api.sigs.k8s.io/user/quick-start.html#install-clusterctl). 

Once installed, run the init:

```bash
kubectl create namespace capi-providers
clusterctl init --core cluster-api --bootstrap kubeadm --control-plane kubeadm --infrastructure hetzner -n capi-providers
kubectl create secret generic hetzner -n capi-providers --from-literal=hcloud=$HCLOUD_TOKEN
kubectl patch secret hetzner -n capi-providers -p '{"metadata":{"labels":{"clusterctl.cluster.x-k8s.io/move":""}}}'
```

This will install:
- cert-manager (bundeled with CAPI)
- capi controller 
- kubeadm bootstrap provider
- hetzner infrastructure provider

To check every worked, you can `kubectl get cluster` and should see no resources ;)

## Stage permanent management cluster

Now we can deploy our permanent management cluster. From our kind cluster's perspective it's a regular workload cluster, and that's how we install it.

Most providers have a quickstart guide for this section, as there are custom fields to a provider. For CAPH the quickstart is [here](https://github.com/syself/cluster-api-provider-hetzner/blob/main/docs/topics/quickstart.md). 

We use `clusterctl` to scaffold a bunch of YAML for us:

```
export HCLOUD_SSH_KEY="<ssh-key-name>" \
export CLUSTER_NAME="my-cluster" \
export HCLOUD_REGION="fsn1" \
export CONTROL_PLANE_MACHINE_COUNT=3 \
export WORKER_MACHINE_COUNT=3 \
export KUBERNETES_VERSION=1.25.2 \
export HCLOUD_CONTROL_PLANE_MACHINE_TYPE=cpx31 \
export HCLOUD_WORKER_MACHINE_TYPE=cpx31
clusterctl generate cluster my-cluster --kubernetes-version v1.25.2 --control-plane-machine-count=3 --worker-machine-count=3  > my-cluster.yaml
```

## Pivot CAPI to permanent management cluster

## Additional resources:
- https://blog.scottlowe.org/2020/12/02/bootstrapping-a-cluster-api-management-cluster/