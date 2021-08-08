pipeline {
  agent any
  options {
    timestamps()
  }
  environment {
    dockerImage = "dreamspace04/"
    username = "niloybiswas"
    devport = 7300
    masterport = 7200
    k8sMasterPort = 30157
    k8sDevelopPort = 30158
    CREDENTIALS_ID = 'gcp-cred'
    LOCATION = 'us-central1-c'
    CLUSTER_NAME = 'niloy-cluster'
    PROJECT_ID = 'unique-iterator-321302'
  }
  tools {
    nodejs 'nodejs'
    jdk 'Java'
  }
  stages {
    stage('Git code checkout') {
      steps {
        checkout scm
      }
    }
    stage('Build') {
      steps {
        echo "My branch is: ${env.BRANCH_NAME}"
        bat 'npm install'
      }
    }

    stage('Unit Testing') {
      when {
        branch 'master'
      }
      steps {
        bat 'npm test'
      }
    }

    stage('Sonar Analysis') {
      when {
        branch 'develop'
      }
      steps {
        bat '..\\..\\tools\\hudson.plugins.sonar.SonarRunnerInstallation\\SonarQubeScanner\\bin\\sonar-scanner.bat -Dsonar.host.url=http://localhost:9000 -Dsonar.login=53a0f89afc930a3c200bb204b29c96c8a7cfe641'
      }
    }

    stage('Docker Image') {
      steps {
        echo "Building Docker Image"
        bat "docker build -t i-${username}-${env.BRANCH_NAME} ."
      }

    }

    stage("Containers") {
      steps {
        parallel(
          "Push to Docker Hub": {
            echo "Pushing Docker Image ${env.BRANCH_NAME}"
            bat "docker tag i-${username}-${env.BRANCH_NAME} ${dockerImage}${env.BRANCH_NAME}:${BUILD_NUMBER}"

            withDockerRegistry([credentialsId: 'DockerHub', url: ""]) {
              bat "docker push ${dockerImage}${env.BRANCH_NAME}:${BUILD_NUMBER}"
            }
          },
          "Pre-container check": {
            script {
              try {
                bat "docker rm -f c-${username}-${env.BRANCH_NAME}"
              } catch (Exception e) {
                echo 'No Existing container'
              }
            }
          }
        )
      }
    }

    stage('Docker deployment') {
      steps {
        script {
          if (env.BRANCH_NAME == "master") {
            echo "Running Docker Image Master"
            bat "docker run --name c-${username}-${env.BRANCH_NAME} -d -p=${masterport}:7100 ${dockerImage}${env.BRANCH_NAME}:${BUILD_NUMBER}"
          } else {
            echo "Running Docker Image Dev"
            bat "docker run --name c-${username}-${env.BRANCH_NAME} -d -p=${devport}:7100 ${dockerImage}${env.BRANCH_NAME}:${BUILD_NUMBER}"
          }
        }
      }
    }

    stage('Kubernetes Deployment') {
      steps {
        echo 'Deploying to Google Kubernetes'
        step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'k8s/deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: false])
      }
    }
  }
}