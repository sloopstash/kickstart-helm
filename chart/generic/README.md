# SloopStash Generic

The SloopStash Generic Helm Chart simplifies the deployment, configuration, and lifecycle management of databases. Currently we have support for Redis — an in-memory key-value data store—within Kubernetes environments. This chart supports environment-specific customizations.

## Prerequisites

- Kubernetes 1.28
- Helm 3.17.0

## Bootstrap Redis environment
```bash
# Store environment variables.
$ export ENVIRONMENT=stg

# Allowed values for $ENVIRONMENT variable.
* stg
* qaa
* qab

# Create directories for Kubernetes persistent-volumes on worker nodes.
$ sudo mkdir -p /mnt/sloopstash/${ENVIRONMENT}/redis/0/data
$ sudo mkdir -p /mnt/sloopstash/${ENVIRONMENT}/redis/0/log
$ sudo chmod -R ugo+rwx /mnt/sloopstash

# Deploy Redis using Helm from OCI registry with default values
helm install sloopstash-generic-redis-v1 oci://registry-1.docker.io/divyapriyamuthuvel/generic --version 1.1.1 -n sloopstash-${ENVIRONMENT}-generic --create-namespace

# Upgrade Redis using Helm release with updated Redis version
helm upgrade sloopstash-generic-redis-v1 oci://registry-1.docker.io/divyapriyamuthuvel/generic --set global.redis_version=7.2.1 -n sloopstash-${ENVIRONMENT}-generic
```

### Verify Redis deployment
```bash
# Access Bash shell of existing OCI container running Redis.
$ kubectl exec -ti -n sloopstash-${ENVIRONMENT}-generic redis-0 -c main -- /bin/bash

# Access Redis client.
$ redis-cli -p 3000

# Exit shell.
$ exit
```


## Manage Redis across multiple environments
```bash
# Store environment variables.
$ export ENVIRONMENT=qaa

# Allowed values for $ENVIRONMENT variable.
* stg
* qaa
* qab

# Deploy Redis using Helm from OCI registry with environment-specific values
helm install sloopstash-generic-redis-v2 oci://registry-1.docker.io/divyapriyamuthuvel/generic --version 1.1.1 --set global.environment=qaa -n sloopstash-${ENVIRONMENT}-generic --create-namespace

# Upgrade Redis using Helm release with updated Redis version using environment-specific values
helm upgrade sloopstash-generic-redis-v2 oci://registry-1.docker.io/divyapriyamuthuvel/generic --version 1.1.1 --set global.redis_version=7.2.1 -n sloopstash-${ENVIRONMENT}-generic

# List resources under Kubernetes namespace.
$ kubectl get pvc,cm,sts,deploy,rs,ds,po,svc,ep,ing -o wide -n sloopstash-${ENVIRONMENT}-generic

# Uninstall Redis deployment using Helm
$ helm uninstall sloopstash-generic-redis-v1 -n sloopstash-${ENVIRONMENT}-generic
$ helm uninstall sloopstash-generic-redis-v2 -n sloopstash-${ENVIRONMENT}-generic

# Delete Kubernetes namespace.
$ kubectl delete namespace sloopstash-${ENVIRONMENT}-generic
```