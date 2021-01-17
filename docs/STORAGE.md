# Storage

Continue has a single large iSCSI NAS at `172.19.250.245`. That has been divided into 3 1TB iSCSI targets:

```
iSCSI Target: iqn.2001-05.com.equallogic:0-8a0906-26b870207-5ec43ae56b05e3b7-kuber1
iSCSI Target: iqn.2001-05.com.equallogic:0-8a0906-284870207-d6e43ae56b35e3b7-kuber2
iSCSI Target: iqn.2001-05.com.equallogic:0-8a0906-29b870207-e5343ae56b65e3b7-kuber3
```

We want to mount one of these onto each of our 3 nodes and then use them as the backing store for a
[Ceph](https://docs.ceph.com/docs/master/start/intro/) cluster. That cluster is set up and managed by
[Rook](https://rook.io/).

# Mounting the iSCSI Targets

Our [Ansible scripts](../ansible/README.md) install the necessary iSCSI tools. The [inventory
file](../ansible/inventory.yml) maps each host to a unique iSCSI target and the script then creates the corresponding
connection. The iSCSI devices are left as raw block devices so they can be managed by Ceph.

# Ceph Cluster Dashboard

There is a dashboard that's run via the `rook-ceph-mgr-dashboard` service. To use it you can port-forward to it and then
load it in a browser:

```
kubectl port-forward -n rook-ceph service/rook-ceph-mgr-dashboard 7000
```

And then point your browser at `localhost:7000`.


## Why Ceph

Oliver did a presentation in February, 2020 [comparing the various k8's storage
options](https://docs.google.com/presentation/d/1pFAZ0XRTvLqRx4Kpa7nI6g9s-I8EDCFumcfAur0mXPc/edit?usp=sharing). It was
decided that [Ceph](https://docs.ceph.com/docs/master/start/intro/) was our best option.
