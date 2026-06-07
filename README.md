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
└── ansible/
    └── deploy.yml

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


Difference from Jenkins + Kubernetes

Jenkins + Kubernetes

Jenkins → kubectl apply → Kubernetes

Jenkins + Ansible + Kubernetes

Jenkins → Ansible Playbook → kubectl apply → Kubernetes

In this setup, Jenkins never runs kubectl apply directly. Ansible becomes the deployment tool that performs the Kubernetes deployment

## Jenkins Credentials Needed
docker-hub-creds (Username/Password)
kubeconfig (Secret File)

#This is the simplest production-style CI/CD project without Helm, RBAC, Ansible, or ArgoCD. Jenkins handles the entire deployment directly using kubectl apply.
