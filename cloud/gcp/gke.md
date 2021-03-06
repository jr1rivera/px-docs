---
layout: page
title: "Dynamic Provisioning on Google Kubernetes Engine (GKE)"
keywords: portworx, container, Kubernetes, storage, Docker, k8s, pv, persistent disk, gke, gce
sidebar: home_sidebar
meta-description: "This page describes how to setup a production ready Portworx cluster in a Google Kubernetes Engine (GKE)."
---

* TOC
{:toc}

The steps below will help you enable dynamic provisioning of Portworx volumes in your Google Kurbenetes Engine (GKE) cluster.

## Prerequisites

{% include px-k8s-prereqs.md skip_ntp="true" skip_firewall="true" %}

## Create a GKE cluster

Following points are important when creating your GKE cluster.

1. Portworx is supported on GKE cluster provisioned on [Ubuntu Node Images](https://cloud.google.com/kubernetes-engine/docs/node-images). So it is important to specify the node image as **Ubuntu** when creating clusters.

2. To manage and auto provision GCP disks, Portworx needs access to the GCP Compute Engine API. For GKE 1.10 and above, Compute Engine API access is disabled by default. This can be enabled in the "Project Access" section when creating the GKE cluster. You can either allow full access to all Cloud APIs or set access for each API. When settting access for each API, make sure to select **Read Write** for the **Compute Engine** dropdown.

3. Portworx requires a ClusterRoleBinding for your user. Without this `kubectl apply ...` command fails with an error like ```clusterroles.rbac.authorization.k8s.io "portworx-pvc-controller-role" is forbidden```.

    Create a ClusterRoleBinding for your user using the following commands:
    ```
    # get current google identity
    $ gcloud info | grep Account
    Account: [myname@example.org]

    # grant cluster-admin to your current identity
    $ kubectl create clusterrolebinding myname-cluster-admin-binding \
        --clusterrole=cluster-admin --user=myname@example.org
    Clusterrolebinding "myname-cluster-admin-binding" created
    ```

More information about creating GKE clusters can be found [here](https://cloud.google.com/kubernetes-engine/docs/clusters/operations).

## Install

Portworx gets deployed as a [Kubernetes DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/). We will use below sections to generate the specs.

### Generate the spec

{% include k8s-spec-generate.md %}

### Applying the spec

Once you have generated the spec file, deploy Portworx.

```bash
$ kubectl apply -f px-spec.yaml
```

{% include k8s-monitor-install.md %}

## Deploy a sample application

Now that you have Portworx installed, checkout various examples of [applications using Portworx on Kubernetes](/scheduler/kubernetes/k8s-px-app-samples.html).

## Troubleshooting Notes

* GKE instances under certain scenarios do not automatically re-attach the persistent disks used by PX.
   - Under the following scenarios, GKE will spin up a new VM as a replacement for older VMs with the same node name:
      * Halting a VM in GKE cluster
      * Upgrading GKE between different kubernetes version
      * Increasing the size of the node pool
   - However, in these cases the previously attached persistent disks will not be re-attached automatically.
   - Currently you will have to manually re-attach the persistent disk to the new VM and then restart portworx on that node.
