pipeline {
  agent any
  options {
    skipDefaultCheckout(true)
  }
  stages {
    stage('Checkout') {
      steps {
        sh 'chmod -R u+rwX "$WORKSPACE" "$WORKSPACE@tmp" || true; rm -rf "$WORKSPACE"/* "$WORKSPACE"/.[!.]* "$WORKSPACE"/..?* "$WORKSPACE"/.??* "$WORKSPACE@tmp"/* 2>/dev/null || true'
        checkout scm
      }
    }
    stage('Build and Test') {
      steps {
        sh 'docker run --rm -u "$(id -u):$(id -g)" -v "$PWD":/workspace -w /workspace maven:3.9.9-eclipse-temurin-17 mvn clean package'
      }
    }
    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://host.docker.internal:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'docker run --rm -u "$(id -u):$(id -g)" --add-host=host.docker.internal:host-gateway -e SONAR_AUTH_TOKEN="$SONAR_AUTH_TOKEN" -e SONAR_URL="$SONAR_URL" -v "$PWD":/workspace -w /workspace maven:3.9.9-eclipse-temurin-17 mvn org.sonarsource.scanner.maven:sonar-maven-plugin:4.0.0.4121:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=$SONAR_URL'
        }
      }
    }
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "omarchouchane/ultimate-cicd:${BUILD_NUMBER}"
        // DOCKERFILE_LOCATION = "java-maven-sonar-argocd-helm-k8s/spring-boot-app/Dockerfile"
        REGISTRY_CREDENTIALS = credentials('docker-cred')
      }
      steps {
        script {
          sh 'docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                dockerImage.push()
            }
        }
      }
    }
    stage('Update Deployment File') {
        environment {
          GIT_REPO_NAME = "spring-app-manifests"
            GIT_USER_NAME = "omarchouchane"
        }
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "omar.ch52831@gmail.com"
                    git config user.name "Omar Chouchane"
                    BUILD_NUMBER=${BUILD_NUMBER}
              rm -rf spring-app-manifests
              git clone https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git
              sed -i "s|image: .*|image: omarchouchane/ultimate-cicd:${BUILD_NUMBER}|g" spring-app-manifests/deployment.yaml
              cd spring-app-manifests
              git add deployment.yaml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
              git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git HEAD:main
                '''
            }
        }
    }
  }
}
