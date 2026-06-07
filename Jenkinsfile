pipeline {
    agent any

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

        stage('Login and Push Image to Dockerhub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                    docker login -u $DOCKER_USER  -p $DOCKER_PASS
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
            echo "Updating deployment.yaml..."

            sed -i 's|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g' k8s/deployment.yaml

            cat k8s/deployment.yaml

            git config user.name "jenkins"
            git config user.email "jenkins@example.com"

            git add k8s/deployment.yaml

            git commit -m "Update image tag to ${IMAGE_TAG}" || echo "No changes to commit"

            git push https://\${GITHUB_TOKEN}@github.com/jadalaramani/todo_cicd_end-end_project.git HEAD:main
            """
        }
    }
}

        stage('Deploy RBAC') {
            steps {
                withCredentials([
                    file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')
                ]) {
                    sh '''
                    kubectl apply -f Kubernetes/namespace.yaml
                    kubectl apply -f rbac/
                    '''
                }
            }
        }

        stage('Deploy Application') {
            steps {
                withCredentials([
                    file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')
                ]) {
                    sh '''
                    kubectl apply -f Kubernetes/deployment.yaml
                    kubectl apply -f Kubernetes/service.yaml
                    '''
                }
            }
        }
    }
}
