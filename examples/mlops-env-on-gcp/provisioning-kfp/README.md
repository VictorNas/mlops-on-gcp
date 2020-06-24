# Provisioning MLOps services

In this lab, you will enable managed services used in MLOps environment and provision a [standalone deployment of Kubeflow Pipelines](https://www.kubeflow.org/docs/pipelines/installation/standalone-deployment/).

Provisioning of the environment has been automated with the `./install.sh` script. 

The script goes through the following steps:
1. Enables required cloud services
1. Provisions an MVP infrastructure to host a standalone deployment of Kubeflow Pipelines. This step has been automated with [Terraform](https://www.terraform.io/):
    1. Creates a VPC 
    1. Creates and configures a GKE cluster
    1. Creates an instance of Cloud SQL
    1. Creates a GCS bucket
    1. Creates a service account for GKE worker nodes and a service account to be used by KFP pipelines
1. Deploys Kubeflow Pipelines and configures the KFP services to use Cloud SQL for ML metadata management and GCS for artifact storage. This step has been automated with [Kustomize](https://kustomize.io/).

## Running the installation script

You will run the provisioning script using [Cloud Shell](https://cloud.google.com/shell/). 

**Terraform** is pre-installed in **Cloud Shell**. The version of `kubectl` installed by default in **Cloud Shell** does not support **Kustomize**. *When `kubectl` in **Cloud Shell** is upgraded to the version that supports **Kustomize** the step below will not be necessary*.

To install **Kustomize** in **Cloud Shell**:
```
cd /usr/local/bin
sudo wget https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2Fv3.3.0/kustomize_v3.3.0_linux_amd64.tar.gz
sudo tar xvf kustomize_v3.3.0_linux_amd64.tar.gz
sudo rm kustomize_v3.3.0_linux_amd64.tar.gz
cd
```
The above command installs **Kustomize** to the `/usr/local/bin` folder, which by default is on the `PATH`. **Kustomize** is a single executable. Note that this folder will be reset - and Kustomize removed - after you disconnect from **Cloud Shell**.


To start the provisioning script:

1. Open **Cloud Shell**
2. Clone this repo under your `home` folder.
```
cd
git clone https://github.com/GoogleCloudPlatform/mlops-on-gcp.git
cd mlops-on-gcp/examples/mlops-env-on-gcp/provisioning-kfp
```

3. Start installation
```
./install.sh [PROJECT_ID] [SQL_PASSWORD] [NAME_PREFIX] [REGION] [ZONE] [NAMESPACE]
```

Where:

|Parameter|Optional|Description|
|-------------|---------|-------------------------------|
|[PROJECT_ID]| Required|The project id of your project.|
|[SQL_PASSWORD]| Required|The password for the Cloud SQL root user|
|[NAME_PREFIX]|Optional|A name prefix tha will be added to the names of the provisioned resources. If not provided [PROJECT_ID] will be used as the prefix|
|[REGION]|Optional|The region for the Cloud SQL instance.  If not provided the `us-central1` region will be used|
|[ZONE]|Optional|The zone for the GKE cluster. If not provided the `us-central1-a` will be used.|
|[NAMESPACE]|Optional|The namespace to deploy KFP to. If not provided the `kubeflow` namespace will be used|

We recommend using the defaults for the region, the zone and the namespace.

4. Review the logs generated by the script for any errors.

## Accessing KFP UI

After the installation completes, you can access the KFP UI from the following URL. You may need to wait a few minutes before the URL is operational.

```
gcloud container clusters get-credentials $PREFIX-cluster --zone $ZONE
echo "https://"$(kubectl describe configmap inverse-proxy-config -n kubeflow | \
grep "googleusercontent.com")
```

## Cleaning up

Run the `./destroy.sh` script to turn down the MLOps services you provisioned.

```
./destroy.sh PROJECT_ID [NAME_PREFIX=PROJECT_ID] [REGION=us-central1] [ZONE=us-central1-a]
```