pipeline {
  agent any

  environment {
    DOCKER_HUB_USER = 'mnlove'
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
        git branch: 'main', url: 'https://github.com/Mchirram41/todo_cicd_end-end_project.git'
      }
    }

    stage('SONARQUBE ANALYSIS') {
      environment {
        SCANNER_HOME = tool 'SonarQube'
      }
      steps {
        withSonarQubeEnv('SonarQube') {
          sh "${SCANNER_HOME}/bin/sonar-scanner"
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 2, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }

    stage('Docker Image Build') {
      steps {
        sh "docker build -t ${LOCAL_IMAGE} ."
      }
    }

    stage('Tag for Docker Hub') {
      steps {
        sh "docker tag ${LOCAL_IMAGE} ${REMOTE_IMAGE}"
      }
    }

    stage('Trivy Vulnerability Scan') {
      steps {
        sh """
          echo 'Running Trivy vulnerability scan...'
          docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image ${REMOTE_IMAGE}
        """
      }
    }

    stage('Login to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'DockerHub',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS')]) {

          sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
        }
      }
    }

    stage('Push to Docker Hub') {
      steps {
        sh "docker push ${REMOTE_IMAGE}"
      }
    }
    stage('Update ArgoCD Repo') {
    steps {
        cleanWs()
        withCredentials([string(credentialsId: 'github-token', variable: 'GIT_TOKEN')]) {
            sh '''
                git config --global user.email "jenkins@example.com"
                git config --global user.name "jenkins"

                git clone https://github.com/Mchirram41/argocd.git
                cd argocd/k8s

                sed -i 's#image: .*#image: Mchirram41/todo-app:22#g' deployment.yml

                git add .
                git commit -m "Update image tag 22"
                git push https://Mchirram41:${GIT_TOKEN}@github.com/Mchirram41/argocd.git main
            '''
        }
    }
}
  }
}
