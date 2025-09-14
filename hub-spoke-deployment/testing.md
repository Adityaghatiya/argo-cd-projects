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
