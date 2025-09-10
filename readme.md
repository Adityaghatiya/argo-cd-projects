Prequisite Create GKE cluster,

connect to gek cluster via cloud  shell:- gcloud container clusters get-credentials argocd-cluster --zone us-central1-a --project modern-media-471511-c9

adding helm repo:-helm repo add argo https://argoproj.github.io/argo-helm
create namespace argocd:- kubectl create namespace argocd

intalling the argocd via helm:- helm install my-argo-cd argo/argo-cd --version 8.3.5 --namespace argocd
Change ArgoCD server service to LoadBalancer
kubectl edit svc argocd-server -n argocd

checking the pods after deployment:- kubectl get pods -n argocd
adityaghatiya57@cloudshell:~ (modern-media-471511-c9)$ kubectl get pods -n argocd
NAME                                                           READY   STATUS      RESTARTS   AGE
my-argo-cd-argocd-application-controller-0                     1/1     Running     0          49s
my-argo-cd-argocd-applicationset-controller-5c7ff66d8b-9srwh   1/1     Running     0          51s
my-argo-cd-argocd-dex-server-dc4f9459-nj6n8                    1/1     Running     0          51s
my-argo-cd-argocd-notifications-controller-644c7cdc57-ln664    1/1     Running     0          51s
my-argo-cd-argocd-redis-7d44cfdb8-6l2tf                        1/1     Running     0          52s
my-argo-cd-argocd-redis-secret-init-rv52r                      1/1     Running   0          63s
my-argo-cd-argocd-repo-server-7cf54b674d-4tqcm                 1/1     Running     0          50s
my-argo-cd-argocd-server-77f9488b4-mw76l                       1/1     Running     0          50s

to check something is running here or not:- kubectl get nodes

for creating the applicating whose file is located here:-kubectl apply -n argocd -f https://raw.githubusercontent.com/Adityaghatiya/argo-cd-projects/main/staldlone/argocd-files/application1.yaml

after this to access the argo-cd dashborad for the CD pipeline checking go to-
Find this line:

type: ClusterIP


Change it to:

type: LoadBalancer


Save and exit.

🔹 Step 2: Get the external IP

Run:

kubectl get svc -n argocd


You’ll see something like:

NAME            TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                     
argocd-server   LoadBalancer   10.96.128.34   34.118.230.45    80:31111/TCP,443:31802/TCP


Copy the EXTERNAL-IP.

🔹 Step 3: Access the ArgoCD dashboard

Open in browser:

https://<EXTERNAL-IP>

🔹 Step 4: Get the login credentials

Default username:

admin


Get the password:

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo




Use that password in the UI.

