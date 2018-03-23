# Canonical Kubernetes on Azure

Checking for errors/failed deployments:

- log into portal
- go to all services
- go to monitor
- go to overview
- click activity log errors
- check for errors around limits.
- create support request and make a case to increase the limits.

You will need to increase your limits to deploy CDK on Azure, you will need to increase your limits for cores and virtual machines.  

azure-storageclasses.yaml is designed to easily provide some azure-native storage if you deploy CDK on Azure. 
