pipeline {
  agent any

  environment {
    DOCKERHUB_REPO = "shashidharreddy217/springboot-hello-world"
    DOCKERHUB_CRED = "dockerhub-cred"   // credential id in Jenkins
    KUBECONFIG_CRED = "kubeconfig"      // secret file (your kubeconfig) in Jenkins
    IMAGE_TAG = "${env.BUILD_NUMBER}"
  }

  tools {
    maven 'Maven'   // configure in Jenkins → Global Tool Configuration
    jdk 'Java17'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build (Maven)') {
      steps {
        sh 'mvn -B clean package -DskipTests=false'
      }
    }

    stage('Build & Push Docker Image') {
      steps {
        script {
          docker.withRegistry('', "${DOCKERHUB_CRED}") {
            def img = docker.build("${DOCKERHUB_REPO}:${IMAGE_TAG}")
            img.push()
            sh "docker tag ${DOCKERHUB_REPO}:${IMAGE_TAG} ${DOCKERHUB_REPO}:latest"
            sh "docker push ${DOCKERHUB_REPO}:latest"
          }
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        withCredentials([file(credentialsId: "${KUBECONFIG_CRED}", variable: 'KUBECONFIG_FILE')]) {
          sh '''
            export KUBECONFIG=${KUBECONFIG_FILE}
            # apply manifests (they reference <DOCKERHUB_USER>/springboot-hello-world:latest)
            kubectl apply -f k8s/deployment.yaml
            kubectl apply -f k8s/service-nodeport.yaml
            kubectl rollout status deployment/springboot-hello-world --timeout=120s
            kubectl get pods -o wide
          '''
        }
      }
    }

    stage('Smoke Test') {
      steps {
        withCredentials([file(credentialsId: "${KUBECONFIG_CRED}", variable: 'KUBECONFIG_FILE')]) {
          sh '''
            export KUBECONFIG=${KUBECONFIG_FILE}
            NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[0].address}')
            echo "Node IP: $NODE_IP"
            curl -sS http://$NODE_IP:30081/ || true
          '''
        }
      }
    }
  }
pipeline {
  agent any

  environment {
    DOCKERHUB_REPO = "REPLACE_WITH/<DOCKERHUB_USER>/springboot-hello-world"
    DOCKERHUB_CRED = "dockerhub-cred"   // credential id in Jenkins
    KUBECONFIG_CRED = "kubeconfig"      // secret file (your kubeconfig) in Jenkins
    IMAGE_TAG = "${env.BUILD_NUMBER}"
  }

  tools {
    maven 'Maven'   // configure in Jenkins → Global Tool Configuration
    jdk 'Java17'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build (Maven)') {
      steps {
        sh 'mvn -B clean package -DskipTests=false'
      }
    }

    stage('Build & Push Docker Image') {
      steps {
        script {
          docker.withRegistry('', "${DOCKERHUB_CRED}") {
            def img = docker.build("${DOCKERHUB_REPO}:${IMAGE_TAG}")
            img.push()
            sh "docker tag ${DOCKERHUB_REPO}:${IMAGE_TAG} ${DOCKERHUB_REPO}:latest"
            sh "docker push ${DOCKERHUB_REPO}:latest"
          }
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        withCredentials([file(credentialsId: "${KUBECONFIG_CRED}", variable: 'KUBECONFIG_FILE')]) {
          sh '''
            export KUBECONFIG=${KUBECONFIG_FILE}
            # apply manifests (they reference <DOCKERHUB_USER>/springboot-hello-world:latest)
            kubectl apply -f k8s/deployment.yaml
            kubectl apply -f k8s/service-nodeport.yaml
            kubectl rollout status deployment/springboot-hello-world --timeout=120s
            kubectl get pods -o wide
          '''
      }
    }

    stage('Smoke Test') {
      steps {
        withCredentials([file(credentialsId: "${KUBECONFIG_CRED}", variable: 'KUBECONFIG_FILE')]) {
          sh '''
            export KUBECONFIG=${KUBECONFIG_FILE}
            NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[0].address}')
            echo "Node IP: $NODE_IP"
            curl -sS http://$NODE_IP:30081/ || true
          '''
        }
      }
    }
  }

  post {
    success { echo "Pipeline completed: ${DOCKERHUB_REPO}:${IMAGE_TAG}" }
    failure { echo "Pipeline failed" }
  }
}

