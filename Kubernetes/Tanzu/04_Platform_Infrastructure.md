# Platform Infrastructure

SSH to jumpbox

    gcloud compute ssh ubuntu@jumpbox

Create directory

    mkdir workspace
    cd workspace

Enable needed APIs

    gcloud services enable compute.googleapis.com && \
    gcloud services enable iam.googleapis.com && \
    gcloud services enable cloudresourcemanager.googleapis.com && \
    gcloud services enable dns.googleapis.com && \
    gcloud services enable sqladmin.googleapis.com

Create service account

    gcloud iam service-accounts create ACCOUNT-NAME

To create a key file for your service account

    gcloud iam service-accounts keys create "terraform.key.json" --iam-account "ACCOUNT-NAME@PROJECT-ID.iam.gserviceaccount.com"

To bind the service account to your project and give it the owner role

    gcloud projects add-iam-policy-binding PROJECT-ID --member 'serviceAccount:ACCOUNT-NAME@PROJECT-ID.iam.gserviceaccount.com' --role 'roles/owner'

Install pivnet

    wget https://github.com/pivotal-cf/pivnet-cli/releases/download/v1.0.4/pivnet-linux-amd64-1.0.4
    mv pivnet-linux-amd64-1.0.4 pivnet
    chmod +x pivnet
    sudo mv pivnet /usr/local/bin/
    pivnet --version

Login

    pivnet login --api-token=${PIVNET_TOKEN}

List the files under this product and identify the file in question.

    pivnet product-files -p elastic-runtime -r 2.9.5

Download GCP Terraform Templates

    pivnet download-product-files -p elastic-runtime -i 697856 -r 2.9.5
