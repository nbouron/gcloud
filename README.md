# gcloud

## Env vars

```shell
export CLUSTER_NAME=""

echo "${CLUSTER_NAME}"
```

## Cluster

```shell
# create a cluster with default configuration
gcloud container clusters create "${CLUSTER_NAME}"

# get cluster credentials
gcloud container clusters get-credentials "${CLUSTER_NAME}"

# list all clusters
gcloud container clusters list

# describe a specific cluster
gcloud container cluster describe "${CLUSTER_NAME}"
```

## Static IP address

```shell
# Create static IP
gcloud compute addresses create helloweb-ip

# Describe static IP
gcloud compute addresses describe helloweb-ip

# List static IPs
gcloud compute addresses list
```

## hello-app

[hello-app](hello-app/README.md)