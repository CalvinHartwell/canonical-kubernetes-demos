# Canonical Kubernetes Logging and Monitoring docs

bootstrap cloud and run juju deploy cdk-logging-monitoring.yaml

once deployed, run the scripts to enable the customisations:

```
# run customisation scripts
chmod 700 scripts/monitoring-config.sh
./monitoring-config.sh
```

It is also possible to use conjure-up to add the logging and monitoring tools. 
