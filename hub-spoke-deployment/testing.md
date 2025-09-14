GKE Multi-Cluster GitOps with Argo CD
![hub_spoke11](https://github.com/user-attachments/assets/bdedd7ac-1788-461e-bb51-64c86715bf25)
A comprehensive guide for setting up a secure, multi-cluster Kubernetes environment on Google Cloud Platform using GitOps principles. This setup features a centralized control cluster running Argo CD that manages deployments across multiple target clusters in separate GCP projects.

## Project Architecture

| Component      | Environment   | Cluster Name  | VPC CIDR    | Purpose                                      |
|----------------|--------------|---------------|-------------|----------------------------------------------|
| **Hub Project**   | Control Plane | `hub-cluster`  | `10.10.0.0/16` | Hosts Argo CD for centralized GitOps management |
| **Spoke1 Project**| Development   | `dev-cluster`  | `10.20.0.0/16` | Development environment workloads             |
| **Spoke2 Project**| Production    | `prod-cluster` | `10.30.0.0/16` | Production environment workloads              |

## Network Topology

| Connection       | Source        | Destination     | Purpose                             |
|------------------|---------------|-----------------|-------------------------------------|
| **Hub → Spoke1** | `hub-project` | `spoke1-project` | Deploy to development environment   |
| **Hub → Spoke2** | `hub-project` | `spoke2-project` | Deploy to production environment    |
| **VPC Peering**  | Cross-project | Bidirectional   | Secure private connectivity         |
| **Fleet Management** | Centralized   | All clusters    | Unified monitoring and management   |

🚀 Implementation Guide
Step 1: Network Foundation
Create VPC Networks with Non-Overlapping CIDR Ranges
Hub Project - GitOps Control Plane

```
# Create VPC for hub project
gcloud compute networks create hub-vpc \
    --project=your-hub-project \
    --subnet-mode=custom \
    --description="Hub VPC for Argo CD control plane"

# Create subnet for hub cluster
gcloud compute networks subnets create hub-subnet \
    --project=your-hub-project \
    --network=hub-vpc \
    --range=10.10.0.0/24 \
    --region=us-central1 \
    --secondary-range=hub-pods=10.11.0.0/16,hub-services=10.12.0.0/20 \
    --enable-private-ip-google-access
```

Spoke1 Project - Development Environment
```
# Create VPC for development project
gcloud compute networks create spoke1-vpc \
    --project=your-dev-project \
    --subnet-mode=custom \
    --description="Spoke1 VPC for development workloads"

# Create subnet for dev cluster
gcloud compute networks subnets create spoke1-subnet \
    --project=your-dev-project \
    --network=spoke1-vpc \
    --range=10.20.0.0/24 \
    --region=us-central1 \
    --secondary-range=spoke1-pods=10.21.0.0/16,spoke1-services=10.22.0.0/20 \
    --enable-private-ip-google-access
```

Spoke2 Project - Production Environment

```
# Create VPC for production project
gcloud compute networks create spoke2-vpc \
    --project=your-prod-project \
    --subnet-mode=custom \
    --description="Spoke2 VPC for production workloads"

# Create subnet for prod cluster
gcloud compute networks subnets create spoke2-subnet \
    --project=your-prod-project \
    --network=spoke2-vpc \
    --range=10.30.0.0/24 \
    --region=us-central1 \
    --secondary-range=spoke2-pods=10.31.0.0/16,spoke2-services=10.32.0.0/20 \
    --enable-private-ip-google-access
```
## Network CIDR Allocation Summary

| Project Type   | VPC Name   | Subnet CIDR   | Pod CIDR     | Service CIDR  | Purpose       |
|----------------|------------|---------------|--------------|---------------|---------------|
| **Hub**        | `hub-vpc`  | `10.10.0.0/24` | `10.11.0.0/16` | `10.12.0.0/20` | Control Plane |
| **Development**| `spoke1-vpc` | `10.20.0.0/24` | `10.21.0.0/16` | `10.22.0.0/20` | Development   |
| **Production** | `spoke2-vpc` | `10.30.0.0/24` | `10.31.0.0/16` | `10.32.0.0/20` | Production    |

Establish VPC Peering (Hub-to-Spoke Architecture)
Hub to Spoke1 Peering

```
# Create peering from hub to spoke1
gcloud compute networks peerings create hub-to-spoke1 \
    --project=your-hub-project \
    --network=hub-vpc \
    --peer-project=your-dev-project \
    --peer-network=spoke1-vpc \
    --auto-create-routes

# Create peering from spoke1 to hub
gcloud compute networks peerings create spoke1-to-hub \
    --project=your-dev-project \
    --network=spoke1-vpc \
    --peer-project=your-hub-project \
    --peer-network=hub-vpc \
    --auto-create-routes
```

Hub to Spoke2 Peering
```
# Create peering from hub to spoke2
gcloud compute networks peerings create hub-to-spoke2 \
    --project=your-hub-project \
    --network=hub-vpc \
    --peer-project=your-prod-project \
    --peer-network=spoke2-vpc \
    --auto-create-routes

# Create peering from spoke2 to hub
gcloud compute networks peerings create spoke2-to-hub \
    --project=your-prod-project \
    --network=spoke2-vpc \
    --peer-project=your-hub-project \
    --peer-network=hub-vpc \
    --auto-create-routes
```
Verify VPC Peering Status

```
# Check hub peering status
gcloud compute networks peerings list --project=your-hub-project

# Check spoke1 peering status  
gcloud compute networks peerings list --project=your-dev-project

# Check spoke2 peering status
gcloud compute networks peerings list --project=your-prod-project
```

Step 2: Private GKE Clusters
Create Hub Cluster (Argo CD Control Plane)
```
gcloud container clusters create hub-cluster \
    --project=your-hub-project \
    --zone=us-central1-a \
    --enable-private-nodes \
    --enable-private-endpoint \
    --master-ipv4-cidr-block=10.13.0.0/28 \
    --enable-ip-alias \
    --network=hub-vpc \
    --subnetwork=hub-subnet \
    --cluster-secondary-range-name=hub-pods \
    --services-secondary-range-name=hub-services \
    --num-nodes=3 \
    --machine-type=e2-medium \
    --disk-size=50GB \
    --enable-autorepair \
    --enable-autoupgrade \
    --max-nodes=5 \
    --min-nodes=1 \
    --enable-autoscaling
```

Create Development Cluster (Spoke1)
```
gcloud container clusters create dev-cluster \
    --project=your-dev-project \
    --zone=us-central1-a \
    --enable-private-nodes \
    --enable-private-endpoint \
    --master-ipv4-cidr-block=10.23.0.0/28 \
    --enable-ip-alias \
    --network=spoke1-vpc \
    --subnetwork=spoke1-subnet \
    --cluster-secondary-range-name=spoke1-pods \
    --services-secondary-range-name=spoke1-services \
    --num-nodes=2 \
    --machine-type=e2-medium \
    --disk-size=50GB \
    --enable-autorepair \
    --enable-autoupgrade \
    --max-nodes=10 \
    --min-nodes=1 \
    --enable-autoscaling
```
Create Production Cluster (Spoke2)
```
gcloud container clusters create prod-cluster \
    --project=your-prod-project \
    --zone=us-central1-a \
    --enable-private-nodes \
    --enable-private-endpoint \
    --master-ipv4-cidr-block=10.33.0.0/28 \
    --enable-ip-alias \
    --network=spoke2-vpc \
    --subnetwork=spoke2-subnet \
    --cluster-secondary-range-name=spoke2-pods \
    --services-secondary-range-name=spoke2-services \
    --num-nodes=3 \
    --machine-type=e2-standard-2 \
    --disk-size=100GB \
    --enable-autorepair \
    --enable-autoupgrade \
    --max-nodes=20 \
    --min-nodes=3 \
    --enable-autoscaling
```

## Cluster Configuration Summary

| Cluster        | Project Type   | Master CIDR    | Node Count   | Machine Type   | Environment   |
|----------------|----------------|----------------|--------------|----------------|---------------|
| **hub-cluster** | Hub            | `10.13.0.0/28` | 3 (1-5)      | `e2-medium`    | Control Plane |
| **dev-cluster** | Development    | `10.23.0.0/28` | 2 (1-10)     | `e2-medium`    | Development   |
| **prod-cluster**| Production     | `10.33.0.0/28` | 3 (3-20)     | `e2-standard-2`| Production    |

Step 3: Cross-Cluster Network Configuration

Get Hub Cluster Network Details
```
# Get hub cluster network configuration
gcloud container clusters describe hub-cluster \
    --project=your-hub-project \
    --zone=us-central1-a \
    --format="value(networkConfig.podIpv4CidrBlock, networkConfig.privateClusterConfig.masterIpv4CidrBlock, network)"

# Expected output: 10.11.0.0/16 10.13.0.0/28 hub-vpc
```

Create Firewall Rules for Cross-Cluster Communication
Allow Hub to Access Spoke1 (Dev) Cluster
```
# Allow hub cluster to communicate with spoke1 cluster nodes
gcloud compute firewall-rules create allow-hub-to-spoke1-nodes \
    --project=your-dev-project \
    --allow=tcp:443,tcp:10250 \
    --source-ranges=10.11.0.0/16,10.13.0.0/28,10.10.0.0/24 \
    --target-tags=gke-dev-cluster-node \
    --network=spoke1-vpc \
    --description="Allow hub cluster to access spoke1 dev cluster API and kubelet"

# Allow spoke1 nodes to communicate back to hub
gcloud compute firewall-rules create allow-spoke1-to-hub-nodes \
    --project=your-hub-project \
    --allow=tcp:443,tcp:10250 \
    --source-ranges=10.21.0.0/16,10.23.0.0/28,10.20.0.0/24 \
    --target-tags=gke-hub-cluster-node \
    --network=hub-vpc \
    --description="Allow spoke1 dev cluster to communicate with hub cluster"
```

Allow Hub to Access Spoke2 (Prod) Cluster
```
# Allow hub cluster to communicate with spoke2 cluster nodes
gcloud compute firewall-rules create allow-hub-to-spoke2-nodes \
    --project=your-prod-project \
    --allow=tcp:443,tcp:10250 \
    --source-ranges=10.11.0.0/16,10.13.0.0/28,10.10.0.0/24 \
    --target-tags=gke-prod-cluster-node \
    --network=spoke2-vpc \
    --description="Allow hub cluster to access spoke2 prod cluster API and kubelet"

# Allow spoke2 nodes to communicate back to hub
gcloud compute firewall-rules create allow-spoke2-to-hub-nodes \
    --project=your-hub-project \
    --allow=tcp:443,tcp:10250 \
    --source-ranges=10.31.0.0/16,10.33.0.0/28,10.30.0.0/24 \
    --target-tags=gke-hub-cluster-node \
    --network=hub-vpc \
    --description="Allow spoke2 prod cluster to communicate with hub cluster"
```

Configure Master Authorized Networks
For Development Cluster (Spoke1)

Navigate to Kubernetes Engine → Clusters → dev-cluster → Networking
Edit Control Plane authorized networks
Add hub cluster IP ranges:

10.11.0.0/16 (Hub Pod CIDR)
10.13.0.0/28 (Hub Master CIDR)
10.10.0.0/24 (Hub Subnet CIDR)



For Production Cluster (Spoke2)

Navigate to Kubernetes Engine → Clusters → prod-cluster → Networking
Edit Control Plane authorized networks
Add hub cluster IP ranges:
```
10.11.0.0/16 (Hub Pod CIDR)
10.13.0.0/28 (Hub Master CIDR)
10.10.0.0/24 (Hub Subnet CIDR)

```

Verify Firewall Rules

```
# Check firewall rules in all projects
gcloud compute firewall-rules list --project=your-hub-project --filter="name~'hub|spoke'"
gcloud compute firewall-rules list --project=your-dev-project --filter="name~'hub|spoke'" 
gcloud compute firewall-rules list --project=your-prod-project --filter="name~'hub|spoke'"
```
Step 4: Fleet Management Setup
Register All Clusters to Centralized Fleet
```
# Register hub cluster to its own project fleet
gcloud container fleet memberships register hub-cluster-membership \
    --project=your-hub-project \
    --gke-cluster=us-central1-a/hub-cluster \
    --location=us-central1-a

# Register spoke1 dev cluster to hub project fleet for centralized management
gcloud container fleet memberships register spoke1-dev-cluster-membership \
    --project=your-hub-project \
    --gke-cluster=projects/your-dev-project/locations/us-central1-a/clusters/dev-cluster \
    --location=us-central1-a

# Register spoke2 prod cluster to hub project fleet for centralized management
gcloud container fleet memberships register spoke2-prod-cluster-membership \
    --project=your-hub-project \
    --gke-cluster=projects/your-prod-project/locations/us-central1-a/clusters/prod-cluster \
    --location=us-central1-a
```
# List all registered fleet members
```
gcloud container fleet memberships list --project=your-hub-project
```
# Expected output should show all three clusters 
```
# Enable Config Management for fleet-wide policies
gcloud container fleet config-management enable --project=your-hub-project

# Enable Multi Cluster Ingress if needed
gcloud container fleet ingress enable --project=your-hub-project
```
Step 5: Argo CD Installation & Configuration
#Install Argo CD on Hub Cluster
```
# Get hub cluster credentials
gcloud container clusters get-credentials hub-cluster \
    --zone=us-central1-a \
    --project=your-hub-project

# Create namespace
kubectl create namespace argocd

# Add Argo Helm Repo
helm repo add argo https://argoproj.github.io/argo-helm

# Install Argo CD via Helm
helm install my-argo-cd argo/argo-cd \
  --version 8.3.5 \
  --namespace argocd

```
