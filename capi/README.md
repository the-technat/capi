# Cluster-API Management Cluster - capi

[The cluster where one or more Infrastructure Providers run, and where resources (e.g. Machines) are stored. Typically referred to when you are provisioning multiple workload clusters.](https://cluster-api.sigs.k8s.io/reference/glossary#management-cluster).

## Hardware

Fujitsu Lifebook E714 with i7, 16GB Ram and 128GB SSD.

## OS

Installed Ubuntu Server 24.04 LTS manually, installed tailscale and joined the tailnet.

Passwordless sudo is enabled.

To ignore the display and suspending of the laptop, I added the following two lines to `/etc/systemd/logind.conf`:

```
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
```

Also ran: `setterm --blank 1 --powerdown 2`

A reboot is needed to apply these changes.

## K3s

Using [k3sup](https://github.com/alexellis/k3sup) this is very easy:

```console
k3sup install \
  --host capi \
  --context capi \
  --tls-san capi.crocodile-bee.ts.net \
  --k3s-extra-args "--flannel-iface tailscale0 \
        --node-ip $TAILSCALE_ADDR \
        --node-external-ip $TAILSCALE_ADDR"
```

## Argo CD

For deploying extra stuff on the Management Cluster I use Argo CD. Installing it is pretty simple:

```
helm repo add argo https://argoproj.github.io/argo-helm
helm upgrade -i argocd -n argocd --create-namespace argo/argo-cd -f argocd-values.yaml
```