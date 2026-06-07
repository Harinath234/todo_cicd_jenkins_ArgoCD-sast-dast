pipeline {
agent any

```
environment {
    DOCKER_USER = 'ramanijadala'
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
            sh """
            sed -i 's|image:.*|image: ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG}|g' Kubernetes/deployment.yaml

            cat Kubernetes/deployment.yaml
            """
        }
    }

    stage('Deploy using Ansible') {
        steps {
            withCredentials([
                file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')
            ]) {
                sh '''
                ansible-playbook ansible/deploy.yml
                '''
            }
        }
    }
}
```

}
