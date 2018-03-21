# cdk-ceph

This document describes how to deploy Canonical Kubernetes with Ceph storage using Juju.

We assume you have installed snap, juju and bootstrapped your cloud of choice (Azure, AWS) or on-premise infrastructure. Do not deploy the kubernetes model though yet.

The instructions for doing that can be found in the juju manuals or in the readme file here: [https://github.com/CalvinHartwell/canonical-kubernetes-demos/blob/master/README.md](https://github.com/CalvinHartwell/canonical-kubernetes-demos/blob/master/README.md).

## Deploy Canonical Kubernetes with Ceph

The repository includes a canonical kubernetes bundle file which also deploys Ceph: [https://github.com/CalvinHartwell/canonical-kubernetes-demos/blob/master/cdk-ceph/cdk-ceph.yaml](https://github.com/CalvinHartwell/canonical-kubernetes-demos/blob/master/cdk-ceph/cdk-ceph.yaml)

Once you have bootstrapped your cloud of choice, you shoud deploy it:

```
 juju deploy cdk-ceph.yaml
```

You can monitor the deployment status using the command:

```
 watch --color juju status --color
```

Eventually, this will all turn green and the workload status will be mostly 'active'. However, you will see an error or warning on the ceph-osd nodes, that no block storage is available.

Don't forget to also install kubectl and copy the kubeconfig file from the kubernetes-master server:

```
 sudo snap install kubectl --classic
 juju scp kubernetes-master/0:/home/ubuntu/config ~/.kube/config
```

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

# output for Azure
calvinh@calvinh-mbp:~$ juju storage-pools
AName Provider Attrs
azure azure
azure-premium azure account-type=Premium_LRS
loop loop
rootfs rootfs
tmpfs tmpfs
```

We can now add some storage to the ceph OSD nodes:

```
# commands for AWS
juju add-storage ceph-osd/0 osd-devices=ebs,10G,1
juju add-storage ceph-osd/1 osd-devices=ebs,10G,1
juju add-storage ceph-osd/2 osd-devices=ebs,10G,1

# commands for Azure
juju add-storage ceph-osd/0 osd-devices=azure,10G,1
juju add-storage ceph-osd/1 osd-devices=azure,10G,1
juju add-storage ceph-osd/2 osd-devices=azure,10G,1
```

You can now monitor the status again, you will see that the storage is being attached to the OSD nodes using:

```
 watch --color juju status --color
```

Finally, we can create a PV on kubernetes:

```
 juju run-action kubernetes-master/0 create-rbd-pv name=test size=50
```

We can check the pv has been crated

```
calvinh@ubuntu-ws:~/Source/canonical-kubernetes-demos$ kubectl get pv
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM     STORAGECLASS   REASON    AGE
test      50M        RWO            Retain           Available             rbd                      44m

```

The PV is now ready to be configured by the kubernetes-workers. Try spinning up a workload and creating a PVC which utilises the PV.


## Useful Links
- [https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-with-snap-on-ubuntu](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-with-snap-on-ubuntu)
- [https://jujucharms.com/docs/2.3/charms-storage-ceph](https://jujucharms.com/docs/2.3/charms-storage-ceph)
- [https://jujucharms.com/ceph-osd](https://jujucharms.com/ceph-osd)
- [https://jujucharms.com/ceph-mon](https://jujucharms.com/ceph-mon)
- [https://kubernetes.io/docs/getting-started-guides/ubuntu/storage/](https://kubernetes.io/docs/getting-started-guides/ubuntu/storage/)
