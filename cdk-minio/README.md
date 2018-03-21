# Minio storage with Canonical Kubernetes

This demo shows how to use Minio storage with Canonical Kubernetes. Minio storage allows you to turn K8s PV into S3 Compatible storage ([https://www.minio.io/kubernetes.html](https://www.minio.io/kubernetes.html)).

It is a pretty useful storage mechanism which gives you Amazon S3 like storage wherever your kubernetes cluster is running. If you're running Kubernetes on-premise within your own data-center, on Google Cloud or Azure, you can avoid the problem of vendor lock-in with AWS by using Minio.

This document assumes you have already deployed Canonical Kubernetes and have a cluster running already with some storage available for the creation of PV. If you want something quick to test with, I recommend you follow the cdk-ceph demo here to deploy Canonical Kubernetes with Ceph [https://github.com/CalvinHartwell/canonical-kubernetes-demos/tree/master/cdk-ceph](https://github.com/CalvinHartwell/canonical-kubernetes-demos/tree/master/cdk-ceph).

## Deploying the Standalone Workload

On the minio website, there is a page for generating the kubernetes payload. This repository includes two examples from that site which have been modified to work out of the box, if you've deployed CDK with Ceph using the steps mentioned above.

The standalone instance is designed to provide minio as a single instance, rather than a distributed cluster. This is useful for testing locally or prototyping, before moving to a bigger deployment.

First make sure you have a pv available:

```
calvinh@ubuntu-ws:~/Source/canonical-kubernetes-demos$ kubectl get pv
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM     STORAGECLASS   REASON    AGE
test      50M        RWO            Retain           Available             rbd                      1h
```

After you have the pv, modify the minio-standalone.yaml kubernetes payload to use that PV. Inside this yaml is a pvc definition, you need to adjust the pvc to use this PV you created.

It is possible to check the details of the PV using kubectl:

```
kubectl edit pv test
# ... this will show you the pv details with full name, it can be changed this way.
```

Inside the minio-standalone.yaml file, there is a pvc (persistent volume claim) defined at the top, this must be modified:

```
# This PV has been added to the regular example, you should change this depending on your architecture
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  # This name uniquely identifies the PVC. Will be used in deployment below.
  name: minio-pv-claim
  labels:
    app: minio-storage-claim
spec:
  # Read more about access modes here: http://kubernetes.io/docs/user-guide/persistent-volumes/#access-modes
  accessModes:
    - ReadWriteOnce
  resources:
    # This is the request for storage. Should be available in the cluster.
    requests:
      storage: 10Mi
  # Uncomment and add storageClass specific to your requirements below. Read more https://kubernetes.io/docs/concepts/storage/persistent-volumes/#class-1
  storageClassName: rbd

```

In this example, we need to match the storageClassName (rbd) with the STORAGECLASS name given in the output of kubectl get pv command (rbd). Once it has been modified, apply the payload:

```
kubectl apply -f minio-standalone.yaml
```

If there are problems, it can be delete and re-applied:

```
kubectl delete -f minio-standalone.yaml
```

Doing this should resolve the pvc:

```
calvinh@ubuntu-ws:~/Source/canonical-kubernetes-demos/cdk-minio$ kubectl get pvc
NAME             STATUS    VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   AGE
minio-pv-claim   Bound     test      50Mi       RWO            rbd            4s
```

Which should now resolve:

## Deploying the Dedicated Workload
## Writing to the Minio Storage
## Reading from the Minio Storage

## Conclusion

## Useful Links
- [https://www.minio.io/kubernetes.html](https://www.minio.io/kubernetes.html)
