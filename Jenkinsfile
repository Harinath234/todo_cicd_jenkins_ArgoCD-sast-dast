pipeline {
agent any

```
environment {
    DOCKER_USERNAME = 'harinath93811'
    IMAGE_NAME = 'todo-app'
    IMAGE_TAG = "${BUILD_NUMBER}"
    GIT_REPO_NAME = "todo_cicd_jenkins_ArgoCD-sast-dast" 
    GIT_USER_NAME = "Harinath234" 
}
stages {

    stage('Checkout') {
        steps {
            git branch: 'main',
            url: 'https://github.com/Harinath234/todo_cicd_jenkins_ArgoCD-sast-dast.git'
        }
    }

stage('SONARQUBE ANALYSIS') {
  environment {
    SCANNER_HOME = tool 'SonarQubeScanner' 
  }
  steps {
    withSonarQubeEnv('SonarQubeServer') { 
      sh """
        ${SCANNER_HOME}/bin/sonar-scanner
      """
    }
  }
}
stage('QUALITY GATE') {
    steps {
        timeout(time: 2, unit: 'MINUTES') {   
            waitForQualityGate abortPipeline: true
        }
    }
}
    stage('Build Image') {
        steps {
            sh '''
            docker build -t $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG .
            '''
        }
    }

    stage('Scan Docker Image using Trivy') { 
            steps { 
                echo 'scanning Image'    
                sh 'trivy image harinath93811/dockerized-app:${BUILD_NUMBER}' 
            } 
        } 
    
    stage('Login and Push Image to DockerHub') {
        steps {
            withCredentials([
                usernamePassword(
                    credentialsId: 'docker-hub-creds',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASS'
                )
            ]) {
                sh '''
                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                docker push $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG
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

                git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                """
            }
        }
    }

	stage('DAST Scan using OWASP ZAP') {
                 steps {
                         sh '''
                                 mkdir -p zap-reports

                                          docker run --rm \
                                          -v $(pwd)/zap-reports:/zap/wrk/:rw \
                                           ghcr.io/zaproxy/zaproxy:stable \
                                           zap-baseline.py \
                                           -t http://<JENKINS_SERVER_IP>:8080 \
                                           -r zap-report.html
                                            '''
    }
    }
	}
	}
