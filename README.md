# Overview

We're running all MVP Studio projects in Kubernetes on [Rancher](https://rancher.com/docs/rancher/v2.x/en/overview/).
This repository contains non-project specific configuration scripts, Kubernetes configuration files, etc. that allow you
to provision a cluster to work with the rest of the MVP Studio system.

# Quick Start for New Devs

The document below contains some details on how things are set up, our VPNs, etc. However, the most common task we have
is onboarding a new developer so they can contribute to an MVP Studio project. If that's all you need to do the easiest
way to start is with the [quick start guide](./docs/QUICK_START.md).

## Accessing Rancher and Setting up Kubectl

In order to access the cluster you must first VPN into the Continu cluster. See the
[VPN_README.md](./docs/VPN_README.md) for details. Assuming you have a running VPN connection then you can connect to
the master node.

The master node is also running the main rancher container at `https://100.64.0.1:8443/` (or
`https://172.19.250.22:8443/` if you're on the Continu OpenVpn rather than the WireGuard VPN).

Authentication is via github so log in via github using any github user that is part of the MVPStudio organization. Once
you've logged in an administrator will add you to our cluster and then whatever projects you are working on. Once in
Rancher you can click the `Kubeconfig file` link to get a `~/.kube/config` file. Note that that file contains settings
for two different clusters, `mvp-studio` and `mvp-studio-KerbHost2`, both of which correspond to our actual cluster. The
difference is the first has the IP address the user used to connect to the cluster while the latter has the server's own
idea of its IP address.  The former is therefore the right one to use. However, if you connected to the cluster while on
the OpenVpn connection both will be identical and you will need to modify the config to use it via the WireGuard VPN by
changing the line `server: https://172.19.250.22:8443/k8s/clusters/c-pv74p` to `server:
https://100.64.0.1:8443/k8s/clusters/c-pv74p`.

### Admin Access

If necessary, the admin credentials to access Rancher can be found in the standard MVP Studio secrets bucket
(`passwords.mvpstudio.org` bucket on GCE).  You can also find the ssh credentials for the nodes in that bucket in the
`continu-k8.txt` file.

## Directories

The subdirectories in here are:

* bootstrap: these are configurations and scripts that need to be run exactly once when a new cluster is created.
* running: anything that is running in our production cluster belongs here. That is, if you haven't changed anything
  then running `kubectl apply -f .` from this directory shouldn't change anything.

**Never** modify cluster state or resources manually: that makes it very hard to recreate the cluster state for a new
cluster and leaves no audit trail. Instead, add, modify, or delete files here. If done correctly we should be able to
exactly re-create the state of our cluster (in another region for example in case of an outage) but `kubectl apply -f .`
in the `boostrap` directory followed by `kubectl apply -f .` in the `running` directory.

# Cluster Components

The cluster contains many components. These are documented is the following sub-documents:

* [Nginx Ingress](./docs/INGRESS.md) is how we route all traffic in the cluster.
* [Hardware](./docs/HARDWARE.md) documents the hardware provided to us by Continu and how we've configured it.
* [Storage](./docs/STORAGE.md) documentation on how we've configured the SAN array and created a
  [Ceph](https://docs.ceph.com/docs/master/start/intro/) cluster from it that can be used for persistent volumes.

