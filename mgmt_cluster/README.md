# Cluster-API Management Cluster

Terminology explained [here](https://cluster-api.sigs.k8s.io/reference/glossary#management-cluster).

## Hardware

Fujitsu Lifebook E714 with i7, 16GB Ram and 128GB OS Disk

## OS

Installed Ubuntu Server 24.04.01 LTS manually, installed tailscale and joined the tailnet.

Passwordless sudo is enabled.

Lastly to ignore the display and suspending of the laptop, I added the following two lines to `/etc/systemd/logind.conf`:

```
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
```

Also ran: `setterm --blank 1 --powerdown 2`

A reboot is needed to apply these changes.

## K3s

Using k3sup this is very easy:

```
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