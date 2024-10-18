# capi

Cluster-API PoC

## Motivation

Managing dozens of clusters is challenging, especially if you treat everyone of them like a pet. This PoC tries to solve that problem by showcasing a solution to treat clusters like cattle.

## Management Cluster

The cluster where one or more Infrastructure Providers run, and where resources (e.g. Machines) are stored. Typically referred to when you are provisioning multiple workload clusters.

See [capi](./capi) for resources about my Management Cluster (called `capi`).