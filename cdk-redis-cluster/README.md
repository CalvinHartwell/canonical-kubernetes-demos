# quick example of creating a redis cluster on cdk

Original docs: [https://github.com/kubernetes/examples/tree/master/staging/storage/redis](https://github.com/kubernetes/examples/tree/master/staging/storage/redis). 


# Create a bootstrap master
kubectl create -f examples/staging/storage/redis/redis-master.yaml

# Create a service to track the sentinels
kubectl create -f examples/staging/storage/redis/redis-sentinel-service.yaml

# Create a replication controller for redis servers
kubectl create -f examples/staging/storage/redis/redis-controller.yaml

# Create a replication controller for redis sentinels
kubectl create -f examples/staging/storage/redis/redis-sentinel-controller.yaml

# Scale both replication controllers
kubectl scale rc redis --replicas=3
kubectl scale rc redis-sentinel --replicas=3

# Delete the original master pod
kubectl delete pods redis-master
kubectl get po
# You will notice the master is killed and a new redis pod is created automatically because of the rc 

# Note: If you are running all the above commands consecutively including this one in a shell script, it may NOT work out. When you run the above commands, let the pods first come up, especially the redis-master pod. Else, the sentinel pods would never be able to know the master redis server and establish a connection with it. 
