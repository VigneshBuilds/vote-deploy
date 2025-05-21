# GitOps with Argo CD
We will be using the vote-deploy repository for our Gitops + Argo CD setup

I'm using WSL for this and KIND for the kuberenetes cluster using Docker

![image](https://github.com/user-attachments/assets/202053a1-cda1-4f53-b6a3-5d07f49b7f74)

Argos CD has been installed in the cluster using the following commands
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
Password to the admin account has been reset to 'password' using the following command:
The value set in admin.password is the bcrypt value of the string 'password'
```
kubectl -n argocd patch secret argocd-secret \
  -p '{"stringData": {
    "admin.password": "$2a$10$rRyBsGSHK6.uc8fntPwVIuLVHgsAhAX7TcdrqW/RADU0uh7CaChLa",
    "admin.passwordMtime": "'$(date +%FT%T%Z)'"
  }}'
```

By default, the Argo CD API server is not exposed
To expose it we need to execute the following : 

```
kubectl patch svc argocd-server -n argocd --patch '{"spec": { "type": "NodePort", "ports": [ { "nodePort": 30200, "port": 443, "protocol": "TCP", "targetPort": 8080 } ] } }'
```
As I'm running KIND I can access the Argo CD UI on http://localhost:30200/

Now we will be creating 2 namespaces: staging and prod
kubectl create ns staging
kubectl create ns prod

To list the namespaces
kubectl get ns
![image](https://github.com/user-attachments/assets/c8c8c0c2-a6a7-44c8-a3c7-c9d1a41fd43a)

After login to the UI we will need to create 2 applications 
```
Application Name : vote-staging
Project : default 
Sync Policy : Automatic 
Prune Resources Checked 
Repository URL : https://github.com/VigneshBuilds/vote-deploy.git 
Revision : main 
Path : staging 
Cluster URL : https://kubernetes.default.svc 
Namespace : staging

Application Name : vote-prod
Project : default
Sync Policy : Automatic
Prune Resources Checked
Repository URL : https://github.com/VigneshBuilds/vote-deploy.git
Revision : release
Path : prod
Cluster URL : https://kubernetes.default.svc
Namespace : prod
```
![image](https://github.com/user-attachments/assets/d75f374f-bda5-45e0-9935-441e9aa374ab)

Once these applications are created we can get their info using the following command:
For prod ns 
![image](https://github.com/user-attachments/assets/38f04ac1-aebd-4e74-a7ee-d7d1df8fca58)
![image](https://github.com/user-attachments/assets/51d2dfbb-af69-4e29-aac2-4c51990616f2)

In the above logs you can see we have 2 replicasets, this is because I've pushed changes to main branch which triggered changes to the application for staging and similarly to the release branch which triggered changes to prod
These replicasets will be used for rollback in case required

![image](https://github.com/user-attachments/assets/456486fa-710f-49cf-b989-718a05954b30)

Staging and prod application with different images 
![image](https://github.com/user-attachments/assets/ae3a27c3-fda0-4e21-b9d5-b6de5e45d879)
![image](https://github.com/user-attachments/assets/238c06e3-5e13-472e-929f-f724b433d2af)

