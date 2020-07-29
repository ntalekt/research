# Entry Point

Verify that your gcloud CLI is pointing to the correct GCP project by running:

    gcloud config list

Run the following command to create a new jumpbox VM in your project.

    gcloud compute instances create "jumpbox" \
      --image-family "ubuntu-1804-lts" \
      --image-project "ubuntu-os-cloud" \
      --boot-disk-size "200" \
      --zone us-central1-a

SSH to jumpbox

    gcloud compute ssh ubuntu@jumpbox

Initialize the jumpbox for GCP

    gcloud config list
    gcloud auth login
    gcloud config list

Create a hidden file named .env

    PIVNET_TOKEN=CHANGE_ME_PIVNET_TOKEN   # see https://network.pivotal.io/users/dashboard/edit-profile
    DOMAIN_NAME=pal4pe.com
    ENV_NAME=CHANGE_ME_ENV_NAME           # e.g. instructor will assign you one
    OPSMAN_PASSWD=CHANGE_ME_OPSMAN_PASSWD # e.g. for simplicity, recycle your PIVNET_TOKEN

    PROJECT_ID=$(gcloud config get-value core/project)
    OPSMAN_FQDN=pcf.${ENV_NAME}.${DOMAIN_NAME}

Install tools

    sudo apt update
    sudo apt install unzip jq
