# Kube-backup

This role is based on ark heptio

## Installation

### Prerequisites

You need to have a minio S3 server as service. Example: `http://backup.s3us.e-unicred.com.br`. 

Save its credentials in `groups/{{ env_site }}/backup.yml`. It will be similar to this:

```
backup_minio_host: backup.s3.mydomain.com:9000
backup_minio_access_key: SSJRJIMDKSMAO
backup_minio_secret_key: KJTVDHJNHKNGB
``` 

### Deploy

Step 1:

        kubectl apply -f ./00-prereqs.yaml

Step 2:

        kubectl apply -f ./00-minio.yaml
        kubectl apply -f ./10-ark-config.yaml
        kubectl apply -f ./20-ark-deployment.yaml
        kubectl apply -f ./30-restic-daemonset.yaml


## Documentation

### YAML files description

The file `00-prereqs.yaml contains all prerequisites necessary to run the Ark server, such as:

- `heptio-ark` namespace
- `ark` service account
- RBAC rules to grant permissions to the `ark` service account
- CRDs for the Ark-specific resources (Backup, Schedule, Restore, Config)

 The file `00-minio.yaml` contains all resources to configure the minio external services

Other complements files are:

	10-ark-config.yaml
	20-ark-deployment.yaml
	30-restic-daemonset.yaml

### How it works

#### Backup workflow


![](https://heptio.github.io/ark/v0.9.0/img/backup-process.png)

#### Backup 

Ex: Making a backup for the ns `demo`

	ark backup create backup-ns-demo --include-namespaces demo


#### Restore

Ex: Restore backup 

For example, delete all resources for the namespace `demo`

	kubectl delete ns demo`

Get all backup 

	ark backup get 

Restore:

	ark restore create --from-backup backup-ns-demo

Check:

	kubectl get all -n demo

#### Schedules

Ex: Create a schedule backup for runs every hour

	ark schedule create backup-k8s-all-cluster --schedule="0 * * * *"

