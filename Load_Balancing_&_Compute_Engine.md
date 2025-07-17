# 🌐 Basic gcloud setup commands

gcloud config set project [PROJECT_ID]
## Desc: Sets the default Google Cloud project for all subsequent gcloud commands.

gcloud config set compute/region REGION
## Desc: Sets the default region for compute resources (e.g., us-central1).

gcloud config set compute/zone ZONE
## Desc: Sets the default zone within the specified region (e.g., us-central1-a).

export PROJECT_ID=$(gcloud config get-value project)
## Desc: Exports the currently set project ID as an environment variable.

export ZONE=$(gcloud config get-value compute/zone)
## Desc: Exports the current compute zone into the shell session.

# 🛠️ Virtual Machine Creation

gcloud compute instances create gcelab2 --machine-type e2-medium --zone $ZONE
## Desc: Creates a VM named gcelab2 with the e2-medium machine type in the specified zone.

gcloud compute ssh gcelab2 --zone $ZONE
## Desc: SSHs into the gcelab2 VM in the specified zone.

gcloud compute instances list
## Desc: Lists all current VM instances in your project.

gcloud compute instances list --filter="name=('gcelab2')"
## Desc: Filters and lists VM instances with the name "gcelab2".

# 🌐 Installing NGINX Web Server

sudo apt update && sudo apt install -y nginx
## Desc: Updates the package list and installs the NGINX web server on the VM.

curl http://$(gcloud compute instances list --filter=name:gcelab2 --format='value(EXTERNAL_IP)')
## Desc: Sends an HTTP request to the external IP of the gcelab2 VM to verify web server response.

# 🔐 Firewall and Access

gcloud compute instances add-tags gcelab2 --tags http-server,https-server
## Desc: Adds network tags to allow firewall rules to apply (like HTTP/HTTPS access).

gcloud compute firewall-rules create default-allow-http \
  --direction=INGRESS --priority=1000 --network=default \
  --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 \
  --target-tags=http-server
## Desc: Creates a firewall rule to allow incoming HTTP traffic (TCP port 80) to instances tagged `http-server`.

# 🔁 Load Balancing (Network & Application)

gcloud compute addresses create network-lb-ip-1 --region REGION
## Desc: Reserves a regional static external IP for a Network Load Balancer.

gcloud compute http-health-checks create basic-check
## Desc: Creates a legacy HTTP health check for checking backend instance health.

gcloud compute target-pools create www-pool \
  --region REGION --http-health-check basic-check
## Desc: Creates a target pool and associates it with the HTTP health check.

gcloud compute target-pools add-instances www-pool \
  --instances www1,www2,www3
## Desc: Adds three web server instances to the target pool.

gcloud compute forwarding-rules create www-rule \
  --region REGION --ports 80 \
  --address network-lb-ip-1 --target-pool www-pool
## Desc: Sets up a forwarding rule that connects the external IP to the target pool on port 80.

gcloud compute addresses create lb-ipv4-1 --ip-version=IPV4 --global
## Desc: Reserves a **global** static external IP for use with the HTTP(S) Load Balancer.

gcloud compute health-checks create http http-basic-check --port 80
## Desc: Creates an HTTP health check for the backend service.

gcloud compute backend-services create web-backend-service \
  --protocol=HTTP --port-name=http \
  --health-checks=http-basic-check --global
## Desc: Creates a backend service using the HTTP protocol and attaches the HTTP health check.

gcloud compute backend-services add-backend web-backend-service \
  --instance-group=lb-backend-group \
  --instance-group-zone=ZONE --global
## Desc: Adds a managed instance group to the backend service for traffic distribution.

gcloud compute url-maps create web-map-http \
  --default-service web-backend-service
## Desc: Creates a URL map that routes all traffic to the backend service.

gcloud compute target-http-proxies create http-lb-proxy \
  --url-map web-map-http
## Desc: Creates an HTTP proxy that uses the URL map to route traffic.

gcloud compute forwarding-rules create http-content-rule \
  --address=lb-ipv4-1 --global \
  --target-http-proxy=http-lb-proxy --ports=80
## Desc: Creates a global forwarding rule to direct traffic to the HTTP proxy on port 80.

# 🔍 Diagnostics & Logging

gcloud logging read "resource.type=gce_instance AND resource.labels.instance_id=$(gcloud compute instances describe gcelab2 --zone=$ZONE --format='get(id)')" \
  --limit 10 --format json
## Desc: Reads the latest 10 log entries for the gcelab2 VM instance from Cloud Logging.
