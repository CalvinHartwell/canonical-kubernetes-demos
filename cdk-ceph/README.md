# cdk-ceph

This document describes how to deploy Canonical Kubernetes with Ceph storage using Juju.

We assume you have installed snap, juju and bootstrapped your cloud of choice (Azure, AWS) or on-premise infrastructure. Do not deploy the kubernetes model though yet.

The instructions for doing that can be found in the juju manuals or in the readme file here: [https://github.com/CalvinHartwell/canonical-kubernetes-demos/blob/master/README.md](https://github.com/CalvinHartwell/canonical-kubernetes-demos/blob/master/README.md).

## Deploy CDK with ceph storage:

The repository includes a canonical kubernetes bundle file which also deploys Ceph: [https://github.com/CalvinHartwell/canonical-kubernetes-demos/blob/master/cdk-ceph/cdk-ceph.yaml](https://github.com/CalvinHartwell/canonical-kubernetes-demos/blob/master/cdk-ceph/cdk-ceph.yaml)

Once you have bootstrapped your cloud of choice, you shoud deploy it:

```
 juju deploy cdk-ceph.yaml
```

You can monitor the deployment status using the command:

```
 watch --color juju status --color:
```
