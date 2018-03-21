# cdk-ceph

This document describes how to deploy Canonical Kubernetes with Ceph storage using Juju.

We assume you have installed snap, juju and bootstrapped your cloud of choice (Azure, AWS) or on-premise infrastructure. Do not deploy the kubernetes model though yet.

The instructions for doing that can be found in the juju manuals or in the readme file here: [https://github.com/CalvinHartwell/canonical-kubernetes-demos/blob/master/README.md](https://github.com/CalvinHartwell/canonical-kubernetes-demos/blob/master/README.md).

## Deploy CDK with Ceph

The repository includes a canonical kubernetes bundle file which also deploys Ceph: [https://github.com/CalvinHartwell/canonical-kubernetes-demos/blob/master/cdk-ceph/cdk-ceph.yaml](https://github.com/CalvinHartwell/canonical-kubernetes-demos/blob/master/cdk-ceph/cdk-ceph.yaml)

Once you have bootstrapped your cloud of choice, you shoud deploy it:

```
 juju deploy cdk-ceph.yaml
```

You can monitor the deployment status using the command:

```
 watch --color juju status --color:
```

Eventually, this will all turn green and the workload status will be mostly 'active'. However, you will see an error or warning on the ceph-osd nodes, that no block storage is available.

The next step is to add some storage devices to the osd nodes which they can consume. You can use the command juju storage-pools to get a list of storage pools:

```
# output for AWS
calvinh@ubuntu-ws:~/Source/canonical-kubernetes-demos/cdk-ceph$ juju storage-pools
Name     Provider  Attrs
ebs      ebs       
ebs-ssd  ebs       volume-type=ssd
loop     loop      
rootfs   rootfs    
tmpfs    tmpfs
```





## Useful Links
- [https://jujucharms.com/docs/2.3/charms-storage-ceph](https://jujucharms.com/docs/2.3/charms-storage-ceph)
- [https://jujucharms.com/ceph-osd](https://jujucharms.com/ceph-osd)
- [https://jujucharms.com/ceph-mon](https://jujucharms.com/ceph-mon)
- [https://kubernetes.io/docs/getting-started-guides/ubuntu/storage/](https://kubernetes.io/docs/getting-started-guides/ubuntu/storage/)
