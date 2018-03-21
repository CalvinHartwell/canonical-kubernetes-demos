# canonical-kubernetes-demos

This repository contains demo workloads for customers, PoC, demonstrations and conferences for use with Canonical Kubernetes (CDK).

## Deploying Canonical Kubernetes on AWS

Deploying Canonical Kubernetes is really easy, I assume you are running on an Ubuntu machine and you have an AWS account.  Grab a new API key from AWS and put that into:

```
~/.aws/credentials
```

In this format:

```
[default]
aws_access_key_id=<access_id>
aws_secret_access_key=<secret_key>
```

Next install juju on your machine, we will use the snap:

```
sudo apt-get install snap
snap install juju
```

Next we bootstrap juju so it's ready to use aws:

```
juju bootstrap
```

We follow the steps here for bootstrapping our cloud for AWS:

```
calvinh@ubuntu-ws:~/.aws$ juju bootstrap
Clouds
aws
aws-china
aws-gov
azure
azure-china
cloudsigma
google
joyent
localhost
oracle
rackspace
salesmaas

Select a cloud [localhost]: aws

Regions in aws:
ap-northeast-1
ap-northeast-2
ap-south-1
ap-southeast-1
ap-southeast-2
ca-central-1
eu-central-1
eu-west-1
eu-west-2
sa-east-1
us-east-1
us-east-2
us-west-1
us-west-2

Select a region in aws [us-east-1]: eu-west-1

Enter a name for the Controller [aws-eu-west-1]: canonical-kubernetes

Creating Juju controller "canonical-kubernetes" on aws/eu-west-1
Looking for packaged Juju agent version 2.3.2 for amd64
Launching controller instance(s) on aws/eu-west-1...
 - i-08ce69142f943b5a4 (arch=amd64 mem=4G cores=2)eu-west-1a"
Installing Juju agent on bootstrap instance
Fetching Juju GUI 2.11.3
Waiting for address
Attempting to connect to 172.31.20.176:22
Attempting to connect to 34.244.155.220:22
Connected to 34.244.155.220
Running machine configuration script...


Bootstrap agent now started
Contacting Juju controller at 34.244.155.220 to verify accessibility...
Bootstrap complete, "canonical-kubernetes" controller now available
Controller machines are in the "controller" model
Initial model "default" added
```

Once the controller has been configured, we now deploy CDK:

```
calvinh@ubuntu-ws:~/.aws$ juju deploy canonical-kubernetes
Located bundle "cs:bundle/canonical-kubernetes-150"
Resolving charm: cs:~containers/easyrsa-27
Resolving charm: cs:~containers/etcd-63
Resolving charm: cs:~containers/flannel-40
Resolving charm: cs:~containers/kubeapi-load-balancer-43
Resolving charm: cs:~containers/kubernetes-master-78
Resolving charm: cs:~containers/kubernetes-worker-81
Executing changes:
- upload charm cs:~containers/easyrsa-27 for series xenial
- deploy application easyrsa on xenial using cs:~containers/easyrsa-27
  added resource easyrsa
- set annotations for easyrsa
- upload charm cs:~containers/etcd-63 for series xenial
- deploy application etcd on xenial using cs:~containers/etcd-63
  added resource etcd
  added resource snapshot
- set annotations for etcd
- upload charm cs:~containers/flannel-40 for series xenial
- deploy application flannel on xenial using cs:~containers/flannel-40
  added resource flannel-amd64
  added resource flannel-s390x
- set annotations for flannel
- upload charm cs:~containers/kubeapi-load-balancer-43 for series xenial
- deploy application kubeapi-load-balancer on xenial using cs:~containers/kubeapi-load-balancer-43
- expose kubeapi-load-balancer
- set annotations for kubeapi-load-balancer
- upload charm cs:~containers/kubernetes-master-78 for series xenial
- deploy application kubernetes-master on xenial using cs:~containers/kubernetes-master-78
  added resource cdk-addons
  added resource kube-apiserver
  added resource kube-controller-manager
  added resource kube-scheduler
  added resource kubectl
- set annotations for kubernetes-master
- upload charm cs:~containers/kubernetes-worker-81 for series xenial
- deploy application kubernetes-worker on xenial using cs:~containers/kubernetes-worker-81
  added resource cni-amd64
  added resource cni-s390x
  added resource kube-proxy
  added resource kubectl
  added resource kubelet
- expose kubernetes-worker
- set annotations for kubernetes-worker
- add relation kubernetes-master:kube-api-endpoint - kubeapi-load-balancer:apiserver
- add relation kubernetes-master:loadbalancer - kubeapi-load-balancer:loadbalancer
- add relation kubernetes-master:kube-control - kubernetes-worker:kube-control
- add relation kubernetes-master:certificates - easyrsa:client
- add relation etcd:certificates - easyrsa:client
- add relation kubernetes-master:etcd - etcd:db
- add relation kubernetes-worker:certificates - easyrsa:client
- add relation kubernetes-worker:kube-api-endpoint - kubeapi-load-balancer:website
- add relation kubeapi-load-balancer:certificates - easyrsa:client
- add relation flannel:etcd - etcd:db
- add relation flannel:cni - kubernetes-master:cni
- add relation flannel:cni - kubernetes-worker:cni
- add unit easyrsa/0 to new machine 0
- add unit etcd/0 to new machine 1
- add unit etcd/1 to new machine 2
- add unit etcd/2 to new machine 3
- add unit kubeapi-load-balancer/0 to new machine 4
- add unit kubernetes-master/0 to new machine 5
- add unit kubernetes-worker/0 to new machine 6
- add unit kubernetes-worker/1 to new machine 7
- add unit kubernetes-worker/2 to new machine 8
Deploy of bundle completed.
```

You can check the deployment status using the following command:

```
 watch --color juju status --color
```

Note that this will give you the default bundle for CDK which is made up of 9 machines, flannel networking and no RBAC. This is based on the default bundle found here: [https://jujucharms.com/canonical-kubernetes/](https://jujucharms.com/canonical-kubernetes/).

For a more tailored build with Canal or Calico, you can use the bundle builder: [https://github.com/juju-solutions/bundle-canonical-kubernetes](https://github.com/juju-solutions/bundle-canonical-kubernetes). This will generate a bundle file, which is just a big piece of yaml which describes the configuration for the entire cluster, similar to an Ansible Playbook or Puppet Manifest.

If you have a custom bundle, you would deploy that using a command like this instead:

```
 juju deploy bundle.yaml
```

Eventually the colours will all turn green and your cluster is good to go. To access the cluster, we need to install the kubectl command line client and copy the kubernetes configuration file over for it to use:

```
 # If this does not work, try adding the --classic option on the end.
 snap install kubectl --classic
```

Next we copy over the configuration file:

```
  juju scp kubernetes-master/0:/home/ubuntu/config ~/.kube/config
```

Finally, using kubectl we can check that kubernetes cluster interaction is possible:

```
Kubernetes master is running at https://34.253.164.197:443

Heapster is running at https://34.253.164.197:443/api/v1/namespaces/kube-system/services/heapster/proxy
KubeDNS is running at https://34.253.164.197:443/api/v1/namespaces/kube-system/services/kube-dns/proxy
kubernetes-dashboard is running at https://34.253.164.197:443/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy
Grafana is running at https://34.253.164.197:443/api/v1/namespaces/kube-system/services/monitoring-grafana/proxy
InfluxDB is running at https://34.253.164.197:443/api/v1/namespaces/kube-system/services/monitoring-influxdb/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'
```

**__Note: There are two other ways CDK would be deployed, either using [Conjure-up](https://tutorials.ubuntu.com/tutorial/install-kubernetes-with-conjure-up#0) or using the graphical juju-as-a-service tool provided at [https://jujucharms.com/](https://jujucharms.com/).__**

## Deploying Canonical Kubernetes on Azure

Make sure you have installed juju and snap:

```
sudo apt-get install snap
snap install juju
```

We can first check that juju supports azure through the following command:

```
juju show-cloud azure
```

We can also update the list of clouds juju has available using the following command:

```
juju update-clouds
```

The azure CLI tool needs to be installed using the following command. You may want to download and review the script first for security reasons rather than piping directly into bash:

```
 curl -L https://aka.ms/InstallAzureCli | bash
```

If this does not work, try installing through apt:

```
# first we setup the apt for the repos
AZ_REPO=$(lsb_release -cs)
echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | \
     sudo tee /etc/apt/sources.list.d/azure-cli.list

#  Install the azure-ci packages
sudo apt-key adv --keyserver packages.microsoft.com --recv-keys 52E16F86FEE04B979B07E28DB02C46DF417A0893
sudo apt-get install apt-transport-https
sudo apt-get update && sudo apt-get install azure-cli
```

During the script, your path is updated and your terminal should be restarted:

```
exec -l $SHELL
```

After this has been installed, you can verify the installation it using the command:

```
 az --version
```

The next step is to add the azure credential to juju so we can use it to provision infrastructure:

```
juju add-credential azure
Enter credential name: cpe-azure

Auth Types
  interactive
  service-principal-secret

Select auth type [interactive]:

Enter subscription-id (optional):

To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code DJCVFCQZ4 to authenticate.
[
  {
    "cloudName": "AzureCloud",
    "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "isDefault": true,
    "name": "Pay-As-You-Go",
    "state": "Enabled",
    "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "user": {
      "name": "calvin.hartwell@canonical.com",
      "type": "user"
    }
  }
]
Credentials added for cloud azure.
```

**__Note that credentials may expire after some time and should be renewed if problems arise with juju when you use Azure.__**

The next step is to bootstrap Azure, change 'mycloud' to something more meaningful, this will be the name of the contoller node on azure:

```
  juju bootstrap azure mycloud
  juju bootstrap azure mycloud
Creating Juju controller "mycloud" on azure/centralus
Looking for packaged Juju agent version 2.3.4 for amd64
Launching controller instance(s) on azure/centralus...
 - machine-0 (arch=amd64 mem=3.5G cores=1)
Installing Juju agent on bootstrap instance
Fetching Juju GUI 2.12.1
Waiting for address
Attempting to connect to 192.168.16.4:22
Attempting to connect to 52.173.249.30:22
Connected to 52.173.249.30
Running machine configuration script...


Bootstrap agent now started
Contacting Juju controller at 192.168.16.4 to verify accessibility...
Bootstrap complete, "mycloud" controller now available
Controller machines are in the "controller" model
Initial model "default" added
```

Once we have the controller node, we are ready to deploy Kubernetes:

```
 juju deploy canonical-kubernetes
```

Or if you have a bundle file:

```
 juju deploy bundle.yaml
```

Finally, you can check the status using:

```
 juju status
```

or

```
 watch --color juju status --color
```

# Useful Links
- [https://jujucharms.com/docs/2.3/help-aws](https://jujucharms.com/docs/2.3/help-aws)
- [https://jujucharms.com/docs/2.2/help-azure](https://jujucharms.com/docs/2.3/help-azure)
- [https://docs.microsoft.com/en-gb/cli/azure/install-azure-cli-apt?view=azure-cli-latest#install](https://docs.microsoft.com/en-gb/cli/azure/install-azure-cli-apt?view=azure-cli-latest#install)
