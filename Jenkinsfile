pipeline {
  agent {
    docker {
      image 'abhishekf5/maven-abhishek-docker-agent:v1'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock --add-host=host.docker.internal:host-gateway' // mount Docker socket to access the host's Docker daemon
    }
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Build and Test') {
      steps {
        sh 'mvn clean package'
      }
    }
    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://34.242.166.110:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "omarchouchane/ultimate-cicd:${BUILD_NUMBER}"
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
            git add spring-app-manifests/deployment.yaml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
            git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git HEAD:main
                '''
            }
        }
    }
  }
}
