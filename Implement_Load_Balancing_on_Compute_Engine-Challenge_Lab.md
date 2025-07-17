# Implement Load Balancing on Compute Engine: Challenge Lab

### Set Variables
```
export INSTANCE=[your-given-instance]
export FIREWALL=[your-given-firewall]
export ZONE=[your-given-zone]
export REGION=[your-given-region]
```
# Desc: These export commands set environment variables to store commonly used values like instance name, firewall rule name, zone, and region. This makes future commands easier to write and less error-prone.


### Task 1. Create a project jumphost instance ###
```
gcloud compute instances create $INSTANCE \
    --zone=$ZONE \
    --machine-type=e2-micro
```
# Desc: Creates a small VM instance (jumphost) in the specified zone using the e2-micro machine type. This is often used for testing, debugging, or SSH-ing into other resources in the project.

### Task 2. Set up an HTTP load balancer ###
```
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
```
# Desc: This creates a startup.sh script that installs and starts the NGINX web server. It also modifies the default web page to include the hostname of the instance, which is helpful for verifying load balancing across different instances.

```
gcloud compute instance-templates create web-server-template \
        --metadata-from-file startup-script=startup.sh \
        --machine-type e2-medium \
        --region $REGION
```
# Desc: Creates a reusable instance template that includes the startup.sh script. This template defines how each VM in the managed instance group should be configured.

```
gcloud compute instance-groups managed create web-server-group \
        --base-instance-name web-server \
        --size 2 \
        --template web-server-template \
        --region $REGION
```
# Desc: Creates a regional managed instance group with 2 instances using the template created above. The group is responsible for automatically scaling and maintaining identical instances.

```
gcloud compute firewall-rules create $FIREWALL \
        --allow tcp:80 \
        --network default
```
# Desc: Creates a firewall rule to allow incoming traffic on TCP port 80 (HTTP) to reach your VM instances. Without this, the load balancer can't reach the web servers.

```
gcloud compute http-health-checks create http-basic-check
```
# Desc: Creates a health check that continuously checks if your instances are responding to HTTP requests. The load balancer uses this to route traffic only to healthy instances.

```
gcloud compute instance-groups managed \
        set-named-ports web-server-group \
        --named-ports http:80 \
        --region $REGION
```
# Desc: Assigns a named port (http) to port 80 for the instance group. This lets the backend service know which port to use for HTTP traffic.

```
gcloud compute backend-services create web-server-backend \
        --protocol HTTP \
        --http-health-checks http-basic-check \
        --global
```
# Desc: Creates a backend service that will route traffic to your instance group using HTTP and health check. This is the core part of the load balancer that manages backend instances.

```
gcloud compute backend-services add-backend web-server-backend \
        --instance-group web-server-group \
        --instance-group-region $REGION \
        --global
```
# Desc: Associates your managed instance group with the backend service so it can receive traffic from the load balancer.

```
gcloud compute url-maps create web-server-map \
        --default-service web-server-backend
```
# Desc: Creates a URL map that routes incoming requests to the backend service. For this basic setup, all requests are sent to the default backend.

```
gcloud compute target-http-proxies create http-lb-proxy \
        --url-map web-server-map
```
# Desc: Creates a target HTTP proxy that uses the URL map to decide where to route incoming HTTP requests.

```
gcloud compute forwarding-rules create http-content-rule \
      --global \
      --target-http-proxy http-lb-proxy \
      --ports 80
```
# Desc: Creates a global forwarding rule that listens on port 80 and directs incoming traffic to the HTTP proxy. This is the external entry point of your load balancer.

(This one is Optional)
```
gcloud compute forwarding-rules list
```
# Desc: Lists all forwarding rules so you can confirm your load balancer is set up and find its external IP address for testing.
