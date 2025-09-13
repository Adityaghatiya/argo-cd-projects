GKE Multi-Cluster GitOps with Argo CD
This repository provides a step-by-step guide for setting up a multi-cluster Kubernetes environment on Google Cloud using a GitOps workflow. We will use a centralized control cluster running Argo CD to deploy applications to a target application cluster in a separate project.
![hub_spoke11](https://github.com/user-attachments/assets/bdedd7ac-1788-461e-bb51-64c86715bf25)

🚀 Architecture Overview
Hub Project (P1): This project hosts the control cluster (control-cluster), which runs the Argo CD server.

Spoke Project (P2): This project hosts the application cluster (spoke-cluster), which serves as our deployment target.

VPC Network Peering: We use VPC Peering to create a private and secure network connection between the two clusters. This allows the Argo CD pods to communicate with the private API endpoint of the spoke cluster.

📋 Prerequisites
Two Google Cloud projects (P1: neat-veld-471511-j4 and P2: sandeep-project-471710).

The gcloud CLI, kubectl, and argocd CLI installed and authenticated.

A bastion host VM located in the same VPC as your P1 cluster.
<img width="1366" height="380" alt="image" src="https://github.com/user-attachments/assets/5e712025-a308-4b5c-8d51-5da394cfecba" />
🛠️ Step-by-Step Setup
Step 1: Network & Cluster Configuration
Create VPCs and Peering: Set up the VPCs in both projects and establish a VPC Network Peering connection between them.
<img width="979" height="387" alt="image" src="https://github.com/user-attachments/assets/adb8f297-6ed7-40dd-9c2e-19c5c58a7262" />
<img width="979" height="392" alt="image" src="https://github.com/user-attachments/assets/b1f831cf-9bd0-4cfa-bfc1-fb400e49dfb5" />

Create Private GKE Clusters: Create a private cluster in each project (control-cluster in P1 and spoke-cluster in P2).

Step 2: Cross-Cluster Network Connectivity
This is the most critical step to avoid network timeouts.

Get P1 Cluster Network Details: Run the following command from your bastion host to get the pod and node IP ranges for your P1 cluster.

```

gcloud container clusters describe control-cluster --project=neat-veld-471511-j4 --zone=us-central1-a --format="value(networkConfig.podIpv4CidrBlock, networkConfig.privateClusterConfig.masterIpv4CidrBlock, network)"
Create Firewall Rule in P2: In the P2 project, create an ingress firewall rule to allow traffic from the P1 cluster's IP ranges (10.184.0.0/14 and 10.1.0.0/20) to the P2 cluster's nodes on tcp:443.
```
Configure Master Authorized Networks: In P2, go to Kubernetes Engine > Clusters > spoke-cluster > Networking and add the P1 cluster's IP ranges to the Control Plane authorized networks.

Step 3: Fleet Management
Register both clusters to a single fleet for centralized management and to enable multi-cluster features.

In P1, navigate to Kubernetes Engine > Clusters and register the control-cluster.

From P2, register the spoke-cluster to the fleet in the neat-veld-471511-j4 project.

Step 4: Argo CD Setup
Install Argo CD on P1: Set your kubectl context to the control-cluster and install Argo CD.

```

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
Add P2 Cluster to Argo CD: Switch your kubectl context to the control-cluster and run the argocd CLI command with the full context name for your spoke cluster.

```

```


argocd cluster add gke_sandeep-project-471710_us-central1-a_spoke-cluster
```
