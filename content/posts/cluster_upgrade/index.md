---
title: Upgrading a Kubernetes Cluster
date: 2022-10-10T07:30:00+00:00
description: Upgrading a Kubernetes cluster using kubeadm
menu:
    sidebar:
        name: K8s Cluster Upgrade
        identifier: k8s-cluster-upgrade
        weight: 15
author:
    name: Nick Ruffles
    image: /images/author/nick.png
tags: ["Security", "Kubernetes", "K8s", "Containers", "Upgrade"]
categories: ["Kubernetes", "Security"]
hero: kube-background.png
---

## Upgrading the controlplane node

On the controlplane node, run the following commands:  
This will update the package lists from the software repository.

```bash
apt update
# This will install the kubeadm version 1.24.0
apt-get install kubeadm=1.24.0-00

# Show the possible upgrades you can do
kubeadm upgrade plan

# This will upgrade Kubernetes controlplane node.
kubeadm upgrade apply v1.24.0

# This will update the kubelet with the version 1.24.0.
apt-get install kubelet=1.24.0-00

# You may need to reload the daemon and restart kubelet service after it has been upgraded.
systemctl daemon-reload
systemctl restart kubelet
```


## Upgrading the worker nodes

Before we upgrade the worker node, we need to drain it of pods, run the following on the `controlplane` node

```bash
kubectl drain node01 --ignore-daemonsets
```

On the `node01` node, run the following commands:

> If you are on the `controlplane` node, run `ssh node01` to log in to the `node01`.

This will update the package lists from the software repository.

```bash
# Update the package list from software repository
apt-get update

# Install the kubeadm version 1.24.0.
apt-get install kubeadm=1.24.0-00

# Upgrade the `node01` configuration.
kubeadm upgrade node

# Update the `kubelet` with the version `1.24.0`.
apt-get install kubelet=1.24.0-00 
```

You may need to reload the `daemon` and restart `kubelet` service after it has been upgraded.

```bash
systemctl daemon-reload
systemctl restart kubelet
```

> Type `exit` or `logout` or enter `CTRL + d` to go back to the `controlplane` node

Run the following command to uncordon the node, make sure this is run on the `controlplane` node

```bash
kubectl uncordon node01
```
