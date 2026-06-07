pipeline {
  agent any

  environment {
    DOCKER_HUB_USER = 'ramanijadala'       
    IMAGE_BASE = 'todo-app'
    IMAGE_TAG = "build-${BUILD_NUMBER}"
    LOCAL_IMAGE = "${IMAGE_BASE}:${IMAGE_TAG}"
    REMOTE_IMAGE = "${DOCKER_HUB_USER}/${IMAGE_BASE}:${IMAGE_TAG}"
    CONTAINER_NAME = "todo-app-container"
    PORT = "3000"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/jadalaramani/todo_cicd_end-end_project.git'
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
    stage('Docker Build') {
      steps {
        sh "docker build -t $LOCAL_IMAGE ."
      }
    }

    stage('Tag for Docker Hub') {
      steps {
        sh "docker tag $LOCAL_IMAGE $REMOTE_IMAGE"
      }
    }
    stage('Trivy Vulnerability Scan') {
      steps {
        sh """
          echo 'Running Trivy vulnerability scan...'
               docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image $REMOTE_IMAGE 
        """
      }
    }

    stage('Login to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
         sh ' docker login -u $DOCKER_USER  -p $DOCKER_PASS'
        }
      }
    }

    stage('Push to Docker Hub') {
      steps {
        sh "docker push $REMOTE_IMAGE"
      }
    }

    stage('Docker Run (optional)') {
      steps {
        script {
          sh "docker rm -f $CONTAINER_NAME || true"
          sh "docker run -d --name $CONTAINER_NAME -p $PORT:3000 $LOCAL_IMAGE"
        }
      }
    }

stage('Deploy with Ansible') {
    steps {
        withKubeConfig(
            credentialsId: 'kubeconfig',
            clusterName: 'my-cluster',
            contextName: 'my-cluster',
            namespace: 'todons',
            restrictKubeConfigAccess: false,
            serverUrl: 'https://YOUR-EKS-ENDPOINT'
        ) {
            sh """
            ansible-playbook ansible/deploy.yml \
              -e image_repo=${DOCKER_HUB_USER}/${IMAGE_BASE} \
              -e image_tag=${IMAGE_TAG}
            """
        }
    }
}
}



  post {
    success {
      echo "✅ Image pushed: $REMOTE_IMAGE and Build Success"
    }
    failure {
      echo "❌ Build failed"
    }
      always {
    emailext (
               subject: "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - ${currentBuild.currentResult}",
               body: "Build details: ${env.BUILD_URL}",
                to: 'mindcircuit2025@gmail.com',
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                
                
            )
      }
    
  }
}
