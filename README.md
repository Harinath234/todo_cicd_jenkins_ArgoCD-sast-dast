## Project Structure

```text
todo-project/
│
├── Jenkinsfile
├── Dockerfile
│
├── Kubernetes/
│   ├── deployment.yaml
│   └── service.yaml
│
└── argocd/
    └── application.yaml

 ```  

## Project Flow

```

GitHub
  ↓
Jenkins
  ↓
Build Docker Image
  ↓
Push Docker Image
  ↓
Update deployment.yaml
  ↓
Run Ansible Playbook
  ↓
Kubernetes

```


## In this setup:

Jenkins = Build & update Git
ArgoCD = Deployment tool
Kubernetes = Runtime platform

There is no kubectl apply in Jenkins, because ArgoCD performs the deployment after detecting the Git commit.

## For Jenkins + ArgoCD + Kubernetes (No Helm, No RBAC, No Ansible), Jenkins should:

Build Docker image

Push image to Docker Hub

Update deployment.yaml

Commit & push to GitHub

Then ArgoCD automatically detects the Git change and deploys to Kubernetes.

## Jenkins Credentials Needed
docker-hub-creds (Username/Password)
kubeconfig (Secret File)
