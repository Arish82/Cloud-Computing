# Cloud-Computing

### Set a default compute zone
  <ul>
  gcloud config set compute/zone us-central1-a
  </ul>

### To create your VM from gcloud console
<ul>
<li>gcloud compute instances create < instance name > --machine-type <machine type> --zone < ZONE ></li>

  "gcloud compute" allows you to manage your Compute Engine resources<br>
  "instances create" creates a new instance.
  </ul>
  
### Explore gcloud commands
  <ul>
    Help
    <li>gcloud -h</li>
    You can access more verbose help by appending the --help flag onto a command or running the gcloud help command.
    <li>gcloud config --help</li>
    <li> gcloud help config</li>
    
    View list of configuration in your env.
    <li>gcloud config list</li>
    <li>gcloud config list --all</li>
   
  </ul>
  
### Install a new component
  <ul>
  Install the beta components:
    <li>sudo apt-get install google-cloud-sdk</li><br>
  
  Enable the gcloud interactive mode
    <li>gcloud beta interactive</li>
  </ul>

## Connect VM with SSH
  <ul>
  gcloud compute ssh [vm Instances name] --zone $ZONE
  </ul>
  
  
# Create a GKE cluster
  <ul>
    To create a cluster,
    <li>gcloud container clusters create [CLUSTER-NAME]</li>
    
    To authenticate the cluster
    <li>gcloud container clusters get-credentials [CLUSTER-NAME]</li>
  </ul>
  
## Deploy an application to the cluster
  <ul>
    To create a new Deployment hello-server from the hello-app container image
    <li>kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0</li>
    
    To create a Kubernetes Service, which is a Kubernetes resource that lets you expose your application to external traffic
    <li>kubectl expose deployment hello-server --type=LoadBalancer --port 8080</li>
    <ul>
      --port specifies the port that the container exposes.
      type="LoadBalancer" creates a Compute Engine load balancer for your container.
    </ul>
    To inspect the hello-server Service
    <li>kubectl get service</li>
  </ul>
  
  ### To delete the cluster
  <ul>
    To delete the cluster
    <li>gcloud container clusters delete [CLUSTER-NAME]</li>
  </ul>
  
  
# Set Up Network and HTTP Load Balancers
  ## Create web server instances
  <ul>
  <li>
    gcloud compute instances create www1 \<br>
  --image-family debian-9 \<br>
  --image-project debian-cloud \<br>
  --zone us-central1-a \<br>
  --tags network-lb-tag \<br>
  --metadata startup-script="#! /bin/bash<br>
    sudo apt-get update<br>
    sudo apt-get install apache2 -y<br>
    sudo service apache2 restart<br>
    echo '<!doctype html><html><body>HTML Here</body></html>' | tee /var/www/html/index.html"
    </li>
    #### Create a firewall rule to allow external traffic to the VM instances
    <li>
      gcloud compute firewall-rules create www-firewall-network-lb \<br>
    --target-tags network-lb-tag --allow tcp:80
    </li>
    #### List your instances.  You can see their IP addresses in the EXTERNAL_IP column
    <li>gcloud compute instances list</li>
  </ul>
  
## Configure the load balancing service
  <ul>
    #### Create a static external IP address for your load balancer:
    <li>gcloud compute addresses create network-lb-ip-1 \<br>
 --region us-central1</li>
    #### Add a legacy HTTP health check resource
    <li>gcloud compute http-health-checks create basic-check</li>
  </ul>
  <ul>
    ##### Add a target pool in the same region as your instances. Run the following to create the target pool and use the health check, which is required for the service to function
    <li>gcloud compute target-pools create www-pool \<br>
    --region us-central1 --http-health-check basic-check</li>
    #### Add the instances to the pool:
    <li>gcloud compute target-pools add-instances www-pool \<br>
    --instances www1,www2,www3</li>
  </ul>
  <ul>
    #### Add a forwarding rule:
    <li>gcloud compute forwarding-rules create www-rule \<br>
    --region us-central1 \<br>
    --ports 80 \<br>
    --address network-lb-ip-1 \<br>
    --target-pool www-pool</li>
  </ul>
  
### Sending traffic to your instances
  <ul>
    ##### Enter the following command to view the external IP address of the www-rule forwarding rule used by the load balancer
    <li>gcloud compute forwarding-rules describe www-rule --region us-central1</li>
    ##### Use curl command to access the external IP address, replacing IP_ADDRESS with an external IP address from the previous command
    <li>while true; do curl -m1 IP_ADDRESS; done</li>
  </ul>
  
## Create an HTTP load balancer
  <ul>
    #### First, create the load balancer template:
    <ul>
      gcloud compute instance-templates create lb-backend-template \<br>
   --region=us-central1 \<br>
   --network=default \<br>
   --subnet=default \<br>
   --tags=allow-health-check \<br>
   --image-family=debian-9 \<br>
   --image-project=debian-cloud \<br>
   --metadata=startup-script='#! /bin/bash<br>
     apt-get update<br>
     apt-get install apache2 -y<br>
     a2ensite default-ssl<br>
     a2enmod ssl<br><br>
     vm_hostname="$(curl -H "Metadata-Flavor:Google" \<br>
     http://169.254.169.254/computeMetadata/v1/instance/name)"
     echo "Page served from: $vm_hostname" | \
     tee /var/www/html/index.html
     systemctl restart apache2'
    </ul>
    #### Create a managed instance group based on the template:
    <ul>
      gcloud compute instance-groups managed create lb-backend-group \<br>
      --template=lb-backend-template --size=2 --zone=us-central1-a
    </ul>
    
    ##### Create the fw-allow-health-check firewall rule. This is an ingress rule that allows traffic from the Google Cloud health checking systems (130.211.0.0/22 and 35.191.0.0/16). This lab uses the target tag allow-health-check to identify the VMs.
    <ul>
      gcloud compute firewall-rules create fw-allow-health-check \<br>
    --network=default \<br>
    --action=allow \<br>
    --direction=ingress \<br>
    --source-ranges=130.211.0.0/22,35.191.0.0/16 \<br>
    --target-tags=allow-health-check \<br>
    --rules=tcp:80
    </ul>
    #### Now that the instances are up and running, set up a global static external IP address that your customers use to reach your load balancer.
    <ul>
      gcloud compute addresses create lb-ipv4-1 \<br>
    --ip-version=IPV4 \<br>
    --global
    </ul>
    #### Note the IPv4 address that was reserved:
    <ul>
      gcloud compute addresses describe lb-ipv4-1 \<br>
    --format="get(address)" \<br>
    --global
    </ul>
    #### Create a healthcheck for the load balancer:
    <ul>
          gcloud compute health-checks create http http-basic-check \<br>
        --port 80
    </ul>
    #### Create a backend service:
    <ul>
          gcloud compute backend-services create web-backend-service \<br>
        --protocol=HTTP \<br>
        --port-name=http \<br>
        --health-checks=http-basic-check \<br>
        --global
    </ul>
    #### Add your instance group as the backend to the backend service:
    <ul>
          gcloud compute backend-services add-backend web-backend-service \<br>
        --instance-group=lb-backend-group \<br>
        --instance-group-zone=us-central1-a \<br>
        --global
    </ul>
    #### Create a URL map to route the incoming requests to the default backend service:
    <ul>
          gcloud compute url-maps create web-map-http \<br>
        --default-service web-backend-service
    </ul>
    #### Create a target HTTP proxy to route requests to your URL map:
    <ul>
          gcloud compute target-http-proxies create http-lb-proxy \<br>
        --url-map web-map-http
    </ul>
    #### Create a global forwarding rule to route incoming requests to the proxy:
    <ul>
          gcloud compute forwarding-rules create http-content-rule \<br>
        --address=lb-ipv4-1\<br>
        --global \<br>
        --target-http-proxy=http-lb-proxy \<br>
        --ports=80
    </ul>
  </ul>
  
  
  
