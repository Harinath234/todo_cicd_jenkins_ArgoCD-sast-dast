pipeline {
agent any

```
environment {
    DOCKER_USER = 'harinath93811'
    IMAGE_NAME = 'todo-app'
    IMAGE_TAG = "${BUILD_NUMBER}"
}

stages {

    stage('Checkout') {
        steps {
            git branch: 'main',
            url: 'https://github.com/jadalaramani/todo_cicd_end-end_project.git'
        }
    }

    stage('Build Image') {
        steps {
            sh '''
            docker build -t $DOCKER_USER/$IMAGE_NAME:$IMAGE_TAG .
            '''
        }
    }

    stage('Login and Push Image to DockerHub') {
        steps {
            withCredentials([
                usernamePassword(
                    credentialsId: 'docker-hub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )
            ]) {
                sh '''
                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                docker push $DOCKER_USER/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }

    stage('Update Deployment YAML') {
        steps {
            withCredentials([
                string(credentialsId: 'githubtoken', variable: 'GITHUB_TOKEN')
            ]) {
                sh """
                sed -i 's|image:.*|image: ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG}|g' Kubernetes/deployment.yaml

                cat Kubernetes/deployment.yaml

                git config user.name "jenkins"
                git config user.email "jenkins@example.com"

                git add Kubernetes/deployment.yaml

                git commit -m "Update image tag to ${IMAGE_TAG}" || echo "No changes to commit"

                git push https://\${GITHUB_TOKEN}@github.com/jadalaramani/todo_cicd_end-end_project.git HEAD:main
                """
            }
        }
    }

    stage('Trigger ArgoCD') {
        steps {
            echo 'Git repository updated successfully. ArgoCD Auto Sync will deploy the application.'
        }
    }
}
```

}
