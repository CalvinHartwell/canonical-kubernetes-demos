# Minio storage with Canonical Kubernetes

This demo shows how to use Minio storage with Canonical Kubernetes. Minio storage allows you to turn K8s PV into S3 Compatible storage ([https://www.minio.io/kubernetes.html](https://www.minio.io/kubernetes.html)).

It is a pretty useful storage mechanism which gives you Amazon S3 like storage wherever your kubernetes cluster is running. If you're running Kubernetes on-premise within your own data-center, on Google Cloud or Azure, you can avoid the problem of vendor lock-in with AWS by using Minio.

## Prerequisites

This document assumes you have already deployed Canonical Kubernetes and have a cluster running already with some storage available for the creation of PV. If you want something quick to test with.

I recommend you follow the [cdk-ceph demo here to deploy Canonical Kubernetes with Ceph](https://github.com/CalvinHartwell/canonical-kubernetes-demos/tree/master/cdk-ceph) but create a default storage class based on the manual instructions. The PVC you create will need at least 1GB of storage so keep this in mind.

## Deploying the Standalone Minio Workload

On the minio website, there is a page for generating the kubernetes payload. This repository includes two examples from that site which have been modified to work out of the box with Canonical Kubernetes if you've deployed it with available PV.

The standalone instance is designed to provide minio as a single instance, rather than a distributed cluster. This is useful for testing locally or prototyping, before moving to a bigger deployment.

First make sure you have a pv available:

```
calvinh@ubuntu-ws:~/Source/canonical-kubernetes-demos/cdk-ceph$ kubectl get pvc
NAME             STATUS    VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   AGE
minio-pv-claim   Bound     test      2G         RWO            rbd            1d
```

After you have the PV, modify the minio-standalone.yaml kubernetes payload to use that PV. Inside this yaml file is a pvc definition, you need to adjust the pvc to use this PV you created.

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
      storage: 1Gi
  # Uncomment and add storageClass specific to your requirements below. Read more https://kubernetes.io/docs/concepts/storage/persistent-volumes/#class-1
  storageClassName: rbd # <- change this line

  # ... rest of the file is omitted
```

In this example, we need to match the storageClassName (rbd) with the STORAGECLASS name given in the output of kubectl get pv command (rbd). Once it has been modified, apply the payload:

```
kubectl apply -f minio-standalone.yaml
```

If there are problems, it can be deleted and the previous command can be re-run to re-deploy the solution:

```
kubectl delete -f minio-standalone.yaml
```

Doing this should resolve the pvc:

```
calvinh@ubuntu-ws:~/Source/canonical-kubernetes-demos/cdk-minio$ kubectl get pvc
NAME             STATUS    VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   AGE
minio-pv-claim   Bound     test      2G         RWO            rbd            1d
```

Which should now resolve the pod creation for minio:

```
calvinh@ubuntu-ws:~/Source/canonical-kubernetes-demos/cdk-ceph$ kubectl get po
NAME                                               READY     STATUS    RESTARTS   AGE
default-http-backend-h9vg4                         1/1       Running   0          1h
minio-deployment-6b7595956b-kzvsp                  1/1       Running   0          33s
nginx-ingress-kubernetes-worker-controller-8c44t   1/1       Running   0          1h
nginx-ingress-kubernetes-worker-controller-xgljb   1/1       Running   0          1h
nginx-ingress-kubernetes-worker-controller-zfqd7   1/1       Running   0          1h
```

And if you then tail the logs for the minio pod, you will end up with a nice welcome message:

```
calvinh@ubuntu-ws:~/Source/canonical-kubernetes-demos/cdk-ceph$ kubectl logs -f minio-deployment-6b7595956b-kzvsp
Created minio configuration file successfully at /root/.minio

Drive Capacity: 1.8 GiB Free, 1.8 GiB Total

Endpoint:  http://10.1.68.5:9000  http://127.0.0.1:9000
AccessKey: admin
SecretKey: password

Browser Access:
   http://10.1.68.5:9000  http://127.0.0.1:9000

Command-line Access: https://docs.minio.io/docs/minio-client-quickstart-guide
   $ mc config host add myminio http://10.1.68.5:9000 admin password

Object API (Amazon S3 compatible):
   Go:         https://docs.minio.io/docs/golang-client-quickstart-guide
   Java:       https://docs.minio.io/docs/java-client-quickstart-guide
   Python:     https://docs.minio.io/docs/python-client-quickstart-guide
   JavaScript: https://docs.minio.io/docs/javascript-client-quickstart-guide
   .NET:       https://docs.minio.io/docs/dotnet-client-quickstart-guide

```

Now our service is up, but we can't access it! Right at the end of the file is a nodeport entry which will be used to access the cluster.

What we need to do is open up this port. We could also use an ingress rule with a load-balancer instead with a DNS entry:

```
juju run --unit kubernetes-worker/0 "open-port 30900"
juju run --unit kubernetes-worker/1 "open-port 30900"
juju run --unit kubernetes-worker/2 "open-port 30900"
```

If you run the above commands and then run juju status, you should be able to grab the IP address of one of your worker nodes.

Hit the IP address in the browser using the port 30900, you should be greated by a nice welcoming web interface:

![minio login page](https://raw.githubusercontent.com/CalvinHartwell/cdk-rancher/master/images/rancher-login.png "Minio Storage Login Page")

But what are the credentials? If you go back and look at the minio-standalone.yaml, you will see a section that defines the admin user and password:

```
- name: MINIO_ACCESS_KEY
  value: "admin"
- name: MINIO_SECRET_KEY
  value: "password"
```

These can be removed, so no access key is required or left as default credentials. Finally We can now log in using these details:

![rancher login page](https://raw.githubusercontent.com/CalvinHartwell/cdk-rancher/master/images/rancher-login.png "Rancher Web GUI Login Page")

## Deploying the Dedicated Minio Workload



## Using the Minio Command Line Tool

To write and read data from we could use any of the available SDK(s), but we will use the minio client (https://docs.minio.io/docs/minio-client-quickstart-guide)[https://docs.minio.io/docs/minio-client-quickstart-guide].

Let's install it first using the snap:

```
 sudo snap install minio-client --edge --devmode
```

We add our newly provisioned minio server to our minio client as an end point:

```
 mc config host add minio http://192.168.1.51 BKIKJAA5BMMU2RHO6IBB V7f1CwQqAcwo80UEIJEjc5gVQUSSx5ohQ9GSrr12
```

## Troubleshooting & Errors

If your PV or PVC is too small, Minio will start correctly and it will throw the following error:

```
Created minio configuration file successfully at /root/.minio

Trace: 1: /q/.q/sources/gopath/src/github.com/minio/minio/cmd/server-main.go:247:cmd.serverMain()
       2: /q/.q/sources/gopath/src/github.com/minio/minio/vendor/github.com/minio/cli/app.go:499:cli.HandleAction()
       3: /q/.q/sources/gopath/src/github.com/minio/minio/vendor/github.com/minio/cli/command.go:214:cli.Command.Run()
       4: /q/.q/sources/gopath/src/github.com/minio/minio/vendor/github.com/minio/cli/app.go:260:cli.(*App).Run()
       5: /q/.q/sources/gopath/src/github.com/minio/minio/cmd/main.go:155:cmd.Main()
       6: /q/.q/sources/gopath/src/github.com/minio/minio/main.go:71:main.main()
[2018-03-21T03:24:34.902590795Z] [ERROR] Initializing object layer failed (disk path full)

Trace: 1: /q/.q/sources/gopath/src/github.com/minio/minio/cmd/server-main.go:249:cmd.serverMain()
       2: /q/.q/sources/gopath/src/github.com/minio/minio/vendor/github.com/minio/cli/app.go:499:cli.HandleAction()
       3: /q/.q/sources/gopath/src/github.com/minio/minio/vendor/github.com/minio/cli/command.go:214:cli.Command.Run()
       4: /q/.q/sources/gopath/src/github.com/minio/minio/vendor/github.com/minio/cli/app.go:260:cli.(*App).Run()
       5: /q/.q/sources/gopath/src/github.com/minio/minio/cmd/main.go:155:cmd.Main()
       6: /q/.q/sources/gopath/src/github.com/minio/minio/main.go:71:main.main()
[2018-03-21T03:24:34.902646769Z] [ERROR] Unable to shutdown http server (server not initialized)
```

To fix this issue, you need to create PV and PVC which provide at least 1GB of storage to Minio. Once you've done that, re-apply the kubernetes workload:

```
 # destroy and re-apply
 kubectl delete -f minio-dedicated.yaml
 kubectl apply -f minio-dedicated.yaml
```

If you get a constant 'ContainerCreating' status this means that the container cannot be started, either it is not in your registry or the PVC has issues:

```
calvinh@ubuntu-ws:~/Source/canonical-kubernetes-demos/cdk-minio$ kubectl get po
NAME                                               READY     STATUS              RESTARTS   AGE
default-http-backend-h9vg4                         1/1       Running             0          22m
minio-deployment-6b7595956b-p4nlj                  0/1       ContainerCreating   0          2m
nginx-ingress-kubernetes-worker-controller-8c44t   1/1       Running             0          20m
nginx-ingress-kubernetes-worker-controller-xgljb   1/1       Running             0          21m
nginx-ingress-kubernetes-worker-controller-zfqd7   1/1       Running             0          21m
```

If you run the kubectl get pvc command and see 'Pending' it is most likely caused by problems with the PVC/PV. You should adjust the size of your PVC so it is less than the total size of your PV and then re-run the container workload until you see the following message:

```
calvinh@ubuntu-ws:~/Source/canonical-kubernetes-demos/cdk-minio$ kubectl get pvc
NAME             STATUS    VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   AGE
minio-pv-claim   Bound     test      1G         RWO            rbd            13s
```

## Conclusion

We have covered the basics of deploying minio storage on-top of Canonical Kubernetes (CDK). The next steps would be to integrate the storage into your own application using one of the provided minio SDK's which can be found in the useful links seciton of this document.

## Useful Links
- [https://www.minio.io/kubernetes.html](https://www.minio.io/kubernetes.html)
- [https://docs.minio.io/docs/python-client-quickstart-guide](https://docs.minio.io/docs/python-client-quickstart-guide)
- [https://docs.minio.io/docs/java-client-quickstart-guide](https://docs.minio.io/docs/java-client-quickstart-guide)
- [https://docs.minio.io/docs/golang-client-quickstart-guide](https://docs.minio.io/docs/golang-client-quickstart-guide)
- [https://docs.minio.io/docs/javascript-client-quickstart-guide](https://docs.minio.io/docs/javascript-client-quickstart-guide)
- [https://docs.minio.io/docs/dotnet-client-quickstart-guide](https://docs.minio.io/docs/dotnet-client-quickstart-guide)
- [https://docs.minio.io/docs/minio-client-quickstart-guide](https://docs.minio.io/docs/minio-client-quickstart-guide)
