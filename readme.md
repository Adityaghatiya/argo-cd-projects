🚀 Argo CD on GKE with Helm

This project demonstrates how to set up Argo CD on a Google Kubernetes Engine (GKE) cluster using Helm.

🔹 What is GitOps?

GitOps is a modern way of implementing Continuous Deployment (CD) for cloud-native applications.
It uses Git as the single source of truth to deliver both applications and infrastructure.

In GitOps:

All configuration and deployment manifests are stored in Git.

Argo CD continuously monitors Git repositories.

Any change in Git is automatically synced and applied to the Kubernetes cluster.

This ensures deployments are declarative, version-controlled, and auditable.

🔹 What is Standalone Argo CD?
In a standalone setup, Argo CD operates as an independent deployment management system where:

Argo CD runs in its own dedicated namespace
Applications are managed through individual Application manifests
Each application can have its own Git repository source
Provides maximum flexibility for application deployment strategies

📋 Prerequisites

A GCP Project (with billing enabled)

GKE Cluster created and running

Google Cloud SDK (Cloud Shell is recommended)

Helm installed

⚙️ Setup Instructions
1. Connect to GKE Cluster
```
gcloud container clusters get-credentials argocd-cluster \
  --zone us-central1-a \
  --project modern-media-471511-c9
```
<img width="1308" height="50" alt="image" src="https://github.com/user-attachments/assets/108be496-d56b-4e2a-8197-68d0780cb941" />
2. Add Argo Helm Repo
```
helm repo add argo https://argoproj.github.io/argo-helm
```

3. Create Argo CD Namespace
```
kubectl create namespace argocd
```
4. Install Argo CD via Helm
```
helm install my-argo-cd argo/argo-cd \
  --version 8.3.5 \
  --namespace argocd
```

5. Expose Argo CD Server

Edit the Argo CD service to use a LoadBalancer:

```
kubectl edit svc argocd-server -n argocd
```

Find:

type: ClusterIP


Change it to:

type: LoadBalancer

6. Verify Deployment

Check if Argo CD pods are running:
```
kubectl get pods -n argocd
```

Check cluster nodes:
```
kubectl get nodes
```
7. Deploy an Application

Apply your application YAML file:

```
kubectl apply -n argocd -f https://raw.githubusercontent.com/Adityaghatiya/argo-cd-projects/main/staldlone/argocd-files/application1.yaml
```

8. Access the Argo CD Dashboard

🔹 Get the external IP:
```
kubectl get svc argocd-server -n argocd
```
```
Example output:

NAME            TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)
argocd-server   LoadBalancer   10.96.128.34   34.118.230.45   80:31111/TCP,443:31802/TCP
```

🔹 Open in browser:
```
https://<EXTERNAL-IP>
```

<img width="1366" height="695" alt="image" src="https://github.com/user-attachments/assets/ea86bcf0-0a0d-4ad5-b394-676ea78adb25" />
9. Login to Argo CD

Username: admin

Password:
```
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d; echo
```

<img width="1219" height="36" alt="image" src="https://github.com/user-attachments/assets/bc2573cc-9b62-45b8-b3c1-2cf751a92770" />

Use these credentials to log in to the dashboard.

10. Application in Argo CD Dashboard

The application is created inside the Argo CD dashboard from the GitHub YAML file:
<img width="1280" height="637" alt="image" src="https://github.com/user-attachments/assets/e68857ab-64fa-4842-a1f4-1e4a4ec28c65" />

When opening the application, the required CD deployment is visible:
<img width="1366" height="637" alt="image" src="https://github.com/user-attachments/assets/256e21d7-4f83-44e9-a948-318f4d58042c" />

