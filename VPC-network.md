# Multiple VPC Netwoks
  <ul>
  <b>Command to create the privatenet network</b>
  <p>gcloud compute networks create privatenet --subnet-mode=custom</p>
  <b>Command to create the privatesubnet-us subnet</b>
  <p>gcloud compute networks subnets create privatesubnet-us --network=privatenet --region=us-central1 --range=172.16.0.0/24</p>
  <p> Above Cmd. creates a private subnet in region= us-central1 with IP as 172.16.0.0/24</p>
  <b>Command to list the available VPC networks</b>
  <p>gcloud compute networks list</p>
  <b>Command to list the available VPC subnets sorted by VPC network</b>
  <p>gcloud compute networks subnets list --sort-by=NETWORK</p>
  </ul>
# Create a firewall rules for privatenet
  <ul>
  <b>command to create the privatenet-allow-icmp-ssh-rdp firewall rule</b>
  <p>gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0</p>
  <b>Command to list all the firewall rules</b>
  <p>gcloud compute firewall-rules list --sort-by=NETWORK</p>
  </ul>
  
# Create the privatenet-us-vm instance
  <ul>
  <b>Create the privatenet-us-vm instance</b>
  <p>gcloud compute instances create private-us-vm --zone=us-central1-f --machine-type=n1-standard-1 --subnet=privatesubnet-us</p>
  <b>List of all the VM instances sorted by ZONE</b>
  <p>gcloud compute instances lsit --sort-by=ZONE</p>
  </ul>
# Explore the connectivity between the VM instances
  <ul>
  <b>PING the external IP adresses. Run cmd in SSh terminal</b>
  <p>ping -c 3 < external IP ></p>
  <b>PING the internal IP addresses</b>
  <p>ping -c 3 < internal IP addresses ></p>
  </ul>
  
# Create a VM instance with multiple network interfaces
  <ul>
  <b>Create the vm-appliance instance with network interfaces in privatesubnet-us, managementsubnet-us and mynetwork.</b>
  <i>The CIDR ranges of these subnets do not overlap, which is a requirement for creating a VM with multiple network interface controllers (NICs).</i>
  <b>List the network interfaces within the VM instance.</b>
  <p>sudo ifconfig</p>
  
  <p></p>
  
  </ul>
