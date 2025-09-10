🚀 Argo CD on GKE with Helm

This project demonstrates how to set up Argo CD on a Google Kubernetes Engine (GKE) cluster using Helm.

📋 Prerequisites

A GCP Project (with billing enabled)

GKE Cluster created and running

Google Cloud SDK
 (Cloud Shell is recommended)

Helm
 installed

⚙️ Setup Instructions
1. Connect to GKE Cluster
gcloud container clusters get-credentials argocd-cluster \
  --zone us-central1-a \
  --project modern-media-471511-c9

2. Add Argo Helm Repo
helm repo add argo https://argoproj.github.io/argo-helm

3. Create Argo CD Namespace
kubectl create namespace argocd

4. Install Argo CD via Helm
helm install my-argo-cd argo/argo-cd \
  --version 8.3.5 \
  --namespace argocd

5. Expose Argo CD Server

Edit the Argo CD service to use a LoadBalancer:

kubectl edit svc argocd-server -n argocd


Find:

type: ClusterIP


Change it to:

type: LoadBalancer

6. Verify Deployment

Check if Argo CD pods are running:

kubectl get pods -n argocd


Check cluster nodes:

kubectl get nodes

7. Deploy an Application

Apply your application YAML file:

kubectl apply -n argocd -f https://raw.githubusercontent.com/Adityaghatiya/argo-cd-projects/main/staldlone/argocd-files/application1.yaml

8. Access the Argo CD Dashboard

🔹 Get the external IP:

kubectl get svc argocd-server -n argocd


Example output:

NAME            TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)
argocd-server   LoadBalancer   10.96.128.34   34.118.230.45   80:31111/TCP,443:31802/TCP


🔹 Open in browser:

https://<EXTERNAL-IP>
<img width="1366" height="695" alt="image" src="https://github.com/user-attachments/assets/ea86bcf0-0a0d-4ad5-b394-676ea78adb25" />


9. Login to Argo CD

Username: admin

Password:

kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d; echo


Use these credentials to log in to the dashboard.

