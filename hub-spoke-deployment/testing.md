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
