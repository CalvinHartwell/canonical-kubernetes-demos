# Canonical  Kubernetes on OVH
------
Provision phase. 

Provision 2 servers.
Import juju key into OVH. 
Spin up one server and use OVH provided ubuntu 16.04 image, also use the juju key, use one OS disk. 
The second disk will be used for ceph, on larger deployment or if more disks available, they can be used for ceph. 
Spin up second server using same template. Make sure you install using original kernel. 
----------

juju phase 



---------
# Links

https://jujucharms.com/docs/2.2/clouds-manual
