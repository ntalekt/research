# Platform Infrastructure

## Pave the IaaS

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

Download GCP Terraform Templates (<https://github.com/pivotal-cf/terraforming-gcp/tree/master/terraforming-pks>)

    pivnet download-product-files -p elastic-runtime -i 697856 -r 2.9.5

Prepare the variables file for Terraform

    vi terraform.tfvars

Template

    env_name            = "ENV_NAME"
    project             = "PROJECT_ID"
    region              = "us-central1"
    zones               = ["us-central1-b", "us-central1-a", "us-central1-c"]
    dns_suffix          = "DOMAIN_NAME"
    opsman_image_url    = ""

    service_account_key = <<SERVICE_ACCOUNT_KEY
    YOUR_SERVICE_ACCOUNT_KEY #Entire contents of terraform.key.json
    SERVICE_ACCOUNT_KEY

Install terraform

    wget -O terraform.zip https://releases.hashicorp.com/terraform/0.11.14/terraform_0.11.14_linux_amd64.zip && \
      unzip terraform.zip && \
      sudo mv terraform /usr/local/bin

Run terraform

    cd ./terraforming-pks/
    ln -s ../terraform.tfvars .
    terraform init
    terraform plan
    terraform apply -auto-approve

## Deploy ops manager

List the files under this product and identify the file in question.

    pivnet product-files -p ops-manager -r 2.9.6

Download GCP Terraform Templates

    pivnet download-product-files -p ops-manager -i 726937 -r 2.9.6

Steps 2 & 3 to create image and VM: <https://docs.pivotal.io/platform/ops-manager/2-9/gcp/deploy-manual.html#create-image>

## Deploy BOSH director

Access ops manager: <https://docs.pivotal.io/platform/ops-manager/2-9/gcp/config-terraform.html#access-om>

Google Config: <https://docs.pivotal.io/platform/ops-manager/2-9/gcp/config-terraform.html#gcp-config>

Director Config: <https://docs.pivotal.io/platform/ops-manager/2-9/gcp/config-terraform.html#director-config>

Availability Zones: <https://docs.pivotal.io/platform/ops-manager/2-9/gcp/config-terraform.html#az>

Networks: <https://docs.pivotal.io/platform/ops-manager/2-9/gcp/config-terraform.html#network>

Assign AZs and networks: <https://docs.pivotal.io/platform/ops-manager/2-9/gcp/config-terraform.html#assign-azs>

Security: Be sure to select the checkbox Include OpsManager Root CA in Trusted Certs
