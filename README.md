## Project Structure

```text
todo-project/
│
├── Jenkinsfile
├── Dockerfile
├── package.json
├── package-lock.json
├── sonar-project.properties
│
├── backend/
│
├── Kubernetes/
│   ├── namespace.yaml
│   ├── deployment.yaml
│   └── service.yaml
│
├── rbac/
│   ├── serviceaccount.yaml
│   ├── role.yaml
│   ├── rolebinding.yaml
│   └── secret.yaml
│
└── .github/
    └── workflows/
```
    

```
Developer
   |
   v
GitHub
   |
   v
Jenkins
   |
   +--> SonarQube
   +--> Trivy
   +--> Docker Build
   +--> DockerHub
   |
   +--> kubectl apply
            |
            v
        Kubernetes
```
