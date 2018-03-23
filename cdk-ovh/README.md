# Canonical  Kubernetes on OVH
------
Provision phase.

Provision 2 servers.
# Import juju key into OVH (/home/calvinh/.local/share/juju/ssh)
# manual cloud users current user keys NOT juju key, at least initially for bootstrap
Import SSH key for current user, generate via ssh-keygen
Spin up one server and use OVH provided ubuntu 16.04 image, also use the juju key, use one OS disk.
The second disk will be used for ceph, on larger deployment or if more disks available, they can be used for ceph.
Spin up second server using same template. Make sure you install using original kernel.
----------

juju phase

```
juju bootstrap manual/149.202.91.28 ovh

calvinh@ubuntu-ws:~$ juju bootstrap manual/root@149.202.91.28 ovh
The authenticity of host '149.202.91.28 (149.202.91.28)' can't be established.
ECDSA key fingerprint is SHA256:KbjNRmXxC+uU3KdAp37pjbkdD3h0KD0JtxDTEyXPw+0.
Are you sure you want to continue connecting (yes/no)? yes
Creating Juju controller "ovh" on manual
Looking for packaged Juju agent version 2.3.4 for amd64
Installing Juju agent on bootstrap instance
Fetching Juju GUI 2.12.1
Running machine configuration script...
Bootstrap agent now started
Contacting Juju controller at 149.202.91.28 to verify accessibility...
Bootstrap complete, "ovh" controller now available
Controller machines are in the "controller" model
Initial model "default" added
```

now we have a controller machine, add another manual manually

```
calvinh@ubuntu-ws:~$ juju add-machine ssh:root@51.255.83.160
The authenticity of host '51.255.83.160 (51.255.83.160)' can't be established.
ECDSA key fingerprint is SHA256:U38M0cafSn12klSZ490Ea8NyAvG4c+DVXdQe8lX8Do8.
Are you sure you want to continue connecting (yes/no)? yes
created machine 0
```

wait a little while and you should see this:

```
calvinh@ubuntu-ws:~/Source/canonical-kubernetes-demos$ juju status
Model    Controller  Cloud/Region  Version  SLA
default  ovh         manual        2.3.4    unsupported

App  Version  Status  Scale  Charm  Store  Rev  OS  Notes

Unit  Workload  Agent  Machine  Public address  Ports  Message

Machine  State  DNS            Inst id               Series  AZ  Message
0        down   51.255.83.160  manual:51.255.83.160  xenial      Manually provisioned machine

calvinh@ubuntu-ws:~/Source/canonical-kubernetes-demos$ juju status
Model    Controller  Cloud/Region  Version  SLA
default  ovh         manual        2.3.4    unsupported

App  Version  Status  Scale  Charm  Store  Rev  OS  Notes

Unit  Workload  Agent  Machine  Public address  Ports  Message

Machine  State    DNS            Inst id               Series  AZ  Message
0        started  51.255.83.160  manual:51.255.83.160  xenial      Manually provisioned machine
```

To destroy it:

```
 juju destroy-model default
```

To add a new default model again:

```
juju add-model default 
```


---------
# Links

https://pastebin.canonical.com/p/t4ptzVwd23/
https://jujucharms.com/docs/2.2/clouds-manual
