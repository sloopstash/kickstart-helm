# SloopStash CRM app

Organize and manage your customers with a sample CRM (Customer Relationship Management) app written on the Flask framework in Python backed by the Redis database.

## Prerequisites

- Kubernetes 1.28
- Helm 3.17.0

## Modify hosts for CRM application access

### Windows
```bash
# Modify hosts configuration.
$ notepad C:\Windows\System32\drivers\etc\hosts
```

```text
192.168.101.61  app.crm.sloopstash.stg app-static.crm.sloopstash.stg
192.168.101.61  app.crm.sloopstash.qaa app-static.crm.sloopstash.qaa
192.168.101.61  app.crm.sloopstash.qab app-static.crm.sloopstash.qab
```

### Mac and Linux
```bash
# Modify hosts configuration.
$ sudo nano /etc/hosts
```

```text
192.168.101.61  app.crm.sloopstash.stg app-static.crm.sloopstash.stg
192.168.101.61  app.crm.sloopstash.qaa app-static.crm.sloopstash.qaa
192.168.101.61  app.crm.sloopstash.qab app-static.crm.sloopstash.qab
```

## Bootstrap CRM stack environment
```bash
# Store environment variables.
$ export ENVIRONMENT=stg

# Allowed values for $ENVIRONMENT variable.
* stg
* qaa
* qab

# Create directories for Kubernetes persistent-volumes on worker nodes.
$ sudo mkdir -p /mnt/sloopstash/${ENVIRONMENT}/crm/redis/0/data
$ sudo mkdir -p /mnt/sloopstash/${ENVIRONMENT}/crm/redis/0/log
$ sudo mkdir -p /mnt/sloopstash/${ENVIRONMENT}/crm/app/log
$ sudo mkdir -p /mnt/sloopstash/${ENVIRONMENT}/crm/nginx/log
$ sudo chmod -R ugo+rwx /mnt/sloopstash

# Deploy CRM application using Helm from OCI registry with default values
helm install sloopstash-crm-v1 oci://registry-1.docker.io/divyapriyamuthuvel/sloopstash-crm --version 1.1.1 -n sloopstash-${ENVIRONMENT}-crm --create-namespace

# Upgrade CRM application using Helm release with updated Nginx replica count
helm upgrade sloopstash-crm-v1  oci://registry-1.docker.io/divyapriyamuthuvel/sloopstash-crm --set nginx_replicas=3 -n sloopstash-${ENVIRONMENT}-crm

# Rollback CRM application to previous stable version using Helm release
helm rollback sloopstash-crm-v1 1 -n sloopstash-${ENVIRONMENT}-crm
```

- STG environment: [http://app.crm.sloopstash.stg:30001/dashboard](http://app.crm.sloopstash.stg:30001/dashboard)
- QAA environment: [http://app.crm.sloopstash.qaa:30002/dashboard](http://app.crm.sloopstash.qaa:30002/dashboard)
- QAB environment: [http://app.crm.sloopstash.qab:30003/dashboard](http://app.crm.sloopstash.qab:30003/dashboard)


## Manage CRM stack across multiple environments
```bash
# Login to Docker Hub OCI registry.
$ helm registry login registry-1.docker.io -u <username>

# Pull the CRM Helm chart from the OCI registry.
$ helm pull oci://registry-1.docker.io/divyapriyamuthuvel/sloopstash-crm --version 1.1.1

# Extract the downloaded package.
$ tar -xvzf sloopstash-crm-1.1.1.tgz

# Store environment variables.
$ export ENVIRONMENT=qaa

# Allowed values for $ENVIRONMENT variable.
* stg
* qaa
* qab

# Deploy CRM application using Helm from OCI registry with environment-specific values
helm install sloopstash-crm-v2 oci://registry-1.docker.io/divyapriyamuthuvel/sloopstash-crm --version 1.1.1 -f sloopstash-crm/var/${ENVIRONMENT}.yml -n sloopstash-${ENVIRONMENT}-crm --create-namespace

# Upgrade CRM application using Helm release with updated Nginx replica count using environment-specific values
helm upgrade sloopstash-crm-v2 oci://registry-1.docker.io/divyapriyamuthuvel/sloopstash-crm --version 1.1.1  -f sloopstash-crm/var/${ENVIRONMENT}.yml --set nginx_replicas=3 -n sloopstash-${ENVIRONMENT}-crm

# Rollback CRM application to previous stable version using Helm release
helm rollback sloopstash-crm-v2 1 -n sloopstash-${ENVIRONMENT}-crm

# List resources under Kubernetes namespace.
$ kubectl get pvc,cm,sts,deploy,rs,ds,po,svc,ep,ing -o wide -n sloopstash-${ENVIRONMENT}-crm

# Uninstall CRM application using Helm
$ helm uninstall sloopstash-crm-v1 -n sloopstash-${ENVIRONMENT}-crm
$ helm uninstall sloopstash-crm-v2 -n sloopstash-${ENVIRONMENT}-crm

# Delete Kubernetes namespace.
$ kubectl delete namespace sloopstash-${ENVIRONMENT}-crm
```