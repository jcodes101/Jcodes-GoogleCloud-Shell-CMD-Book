# 🧰 Beginner Google Cloud Shell Commands

Welcome! This guide contains useful beginner-level commands for navigating and using **Google Cloud Shell**. These commands work in the terminal provided in your Google Cloud Console and help you manage your GCP resources and environment efficiently.

---

## 🗂️ File & Directory Management

```bash
pwd                         # Show current directory
ls                          # List files and folders
ls -la                      # Detailed list including hidden files
cd [folder]                 # Change directory
mkdir [folder-name]         # Create a new directory
touch [file-name]           # Create an empty file
rm [file]                   # Delete a file
rm -r [folder]              # Delete a directory and its contents

<ins>🔧 GCP Configuration</ins>

gcloud init                 # Initialize gcloud with your account
gcloud auth login           # Authenticate your Google account
gcloud config list          # List current configuration
gcloud config set project [PROJECT_ID]    # Set your active project
gcloud projects list        # List all accessible projects

<ins>☁️ GCP Resource Management</ins>
<ins>Compute Engine</ins>

gcloud compute instances list                        # List VM instances
gcloud compute instances create [NAME] --zone=[ZONE] # Create a VM
gcloud compute ssh [INSTANCE_NAME] --zone=[ZONE]     # SSH into a VM
gcloud compute instances delete [NAME] --zone=[ZONE] # Delete a VM

<ins>Cloud Storage</ins>

gsutil ls                                    # List buckets
gsutil mb gs://[BUCKET_NAME]                 # Make a new bucket
gsutil cp [FILE] gs://[BUCKET_NAME]/         # Upload file to bucket
gsutil cp gs://[BUCKET_NAME]/[FILE] ./       # Download file from bucket
gsutil rm gs://[BUCKET_NAME]/[FILE]          # Delete a file in bucket

<ins>🧪 Cloud Functions</ins>

gcloud functions deploy [NAME] --runtime nodejs18 --trigger-http --allow-unauthenticated
gcloud functions list
gcloud functions call [NAME]

<ins>🔄 Kubernetes Engine (GKE)</ins>

gcloud container clusters list                            # List clusters
gcloud container clusters get-credentials [CLUSTER_NAME]  # Connect to a cluster
kubectl get pods                                           # List running pods
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
kubectl expose deployment hello-server --type=LoadBalancer --port 80 --target-port 8080

<ins>🔐 IAM & Roles</ins>

gcloud iam service-accounts list                         # List service accounts
gcloud projects add-iam-policy-binding [PROJECT_ID] \
--member="user:[EMAIL]" \
--role="roles/viewer"                                     # Grant viewer access

<ins>🛠️ Miscellaneous</ins>

gcloud help                                               # General help
gcloud [service] --help                                   # Help on a specific service (e.g., gcloud compute --help)
clear                                                     # Clear the terminal
exit                                                      # Exit Cloud Shell
