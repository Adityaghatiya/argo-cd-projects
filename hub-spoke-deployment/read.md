GKE Multi-Cluster GitOps with Argo CD

This guide demonstrates how to set up a multi-cluster Kubernetes environment on Google Cloud using a GitOps workflow. A centralized control cluster running Argo CD will deploy applications to a target application cluster in a separate project.

🚀 Architecture Overview

Hub Project (P1):

Hosts the control cluster (control-cluster) running Argo CD.

Spoke Project (P2):

Hosts the application cluster (spoke-cluster) as the deployment target.

VPC Network Peering:

Provides private and secure connectivity between clusters.

Allows Argo CD pods in P1 to communicate with the private API endpoint of the spoke cluster.

📋 Prerequisites

Two GCP projects:

P1: neat-veld-471511-j4

P2: sandeep-project-471710

gcloud, kubectl, and argocd CLI installed and authenticated.

Bastion host VM in the same VPC as the P1 cluster.

<img width="1366" height="380" src="https://github.com/user-attachments/assets/5e712025-a308-4b5c-8d51-5da394cfecba" />
🛠️ Step-by-Step Setup
Step 1: Network & Cluster Configuration

Create VPCs and Peering

Set up VPCs in both projects.

Establish VPC Network Peering between them.

<img width="979" height="387" src="https://github.com/user-attachments/assets/adb8f297-6ed7-40dd-9c2e-19c5c58a7262" /> <img width="979" height="392" src="https://github.com/user-attachments/assets/b1f831cf-9bd0-4cfa-bfc1-fb400e49dfb5" />

Create Private GKE Clusters

control-cluster in P1

spoke-cluster in P2

Step 2: Cross-Cluster Network Connectivity

Critical for avoiding network timeouts.

Get P1 Cluster Network Details

gcloud container clusters describe control-cluster \
  --project=neat-veld-471511-j4 \
  --zone=us-central1-a \
  --format="value(networkConfig.podIpv4CidrBlock, networkConfig.privateClusterConfig.masterIpv4CidrBlock, network)"


Create Firewall Rule in P2

Allow traffic from P1 cluster IP ranges (10.184.0.0/14, 10.1.0.0/20) to P2 cluster nodes on TCP port 443.

Configure Master Authorized Networks

In P2: Kubernetes Engine → Clusters → spoke-cluster → Networking

Add P1 cluster IP ranges to Control Plane authorized networks.

Step 3: Fleet Management

Register both clusters to a single fleet for centralized management.

P1: Register control-cluster

P2: Register spoke-cluster under the P1 project (neat-veld-471511-j4)

Step 4: Argo CD Setup

Install Argo CD on P1

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


Add P2 Cluster to Argo CD

argocd cluster add gke_sandeep-project-471710_us-central1-a_spoke-clusterGKE Multi-Cluster GitOps with Argo CD

This guide demonstrates how to set up a multi-cluster Kubernetes environment on Google Cloud using a GitOps workflow. A centralized control cluster running Argo CD will deploy applications to a target application cluster in a separate project.

🚀 Architecture Overview

Hub Project (P1):

Hosts the control cluster (control-cluster) running Argo CD.

Spoke Project (P2):

Hosts the application cluster (spoke-cluster) as the deployment target.

VPC Network Peering:

Provides private and secure connectivity between clusters.

Allows Argo CD pods in P1 to communicate with the private API endpoint of the spoke cluster.

📋 Prerequisites

Two GCP projects:

P1: neat-veld-471511-j4

P2: sandeep-project-471710

gcloud, kubectl, and argocd CLI installed and authenticated.

Bastion host VM in the same VPC as the P1 cluster.

<img width="1366" height="380" src="https://github.com/user-attachments/assets/5e712025-a308-4b5c-8d51-5da394cfecba" />
🛠️ Step-by-Step Setup
Step 1: Network & Cluster Configuration

Create VPCs and Peering

Set up VPCs in both projects.

Establish VPC Network Peering between them.

<img width="979" height="387" src="https://github.com/user-attachments/assets/adb8f297-6ed7-40dd-9c2e-19c5c58a7262" /> <img width="979" height="392" src="https://github.com/user-attachments/assets/b1f831cf-9bd0-4cfa-bfc1-fb400e49dfb5" />

Create Private GKE Clusters

control-cluster in P1

spoke-cluster in P2

Step 2: Cross-Cluster Network Connectivity

Critical for avoiding network timeouts.

Get P1 Cluster Network Details

gcloud container clusters describe control-cluster \
  --project=neat-veld-471511-j4 \
  --zone=us-central1-a \
  --format="value(networkConfig.podIpv4CidrBlock, networkConfig.privateClusterConfig.masterIpv4CidrBlock, network)"


Create Firewall Rule in P2

Allow traffic from P1 cluster IP ranges (10.184.0.0/14, 10.1.0.0/20) to P2 cluster nodes on TCP port 443.

Configure Master Authorized Networks

In P2: Kubernetes Engine → Clusters → spoke-cluster → Networking

Add P1 cluster IP ranges to Control Plane authorized networks.

Step 3: Fleet Management

Register both clusters to a single fleet for centralized management.

P1: Register control-cluster

P2: Register spoke-cluster under the P1 project (neat-veld-471511-j4)

Step 4: Argo CD Setup

Install Argo CD on P1
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Add P2 Cluster to Argo CD

argocd cluster add gke_sandeep-project-471710_us-central1-a_spoke-cluster
