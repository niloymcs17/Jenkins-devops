pipeline {
  agent any
  options {
    timestamps()
  }
  environment {
    dockerImage = "dreamspace04/nagp-devops-nodejs"
    username = "niloybiswas"
    devport = 7300
    masterport = 7200
    k8sMasterPort = 30157
    k8sDevelopPort = 30158
  }
  tools {
    nodejs 'nodejs'
    jdk 'Java'
    dockerTool 'Test_Docker'
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

    // stage('testing ') {
    //   steps {
    //     script {
    //       if (env.BRANCH_NAME == 'master') {
    //         echo 'I only execute on the master branch'
    //       } else {
    //         echo 'I execute elsewhere'
    //       }
    //     }
    //   }
    // }

    stage('Unit Testing') {
      steps {
        bat 'npm test'
      }
    }

    stage('Sonar Analysis') {
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
            bat "docker tag i-${username}-${env.BRANCH_NAME} ${dockerImage}:${BUILD_NUMBER}"

            withDockerRegistry([credentialsId: 'DockerHub', url: ""]) {
              bat "docker push ${dockerImage}:${BUILD_NUMBER}"
            }
          },
          "Pre-container check": {
            bat "docker rm c-${username}-master"
          }
        )
      }
    }

    // stage("remove existing container") {
    //   steps {
    //     bat "docker rm c-${username}-master"
    //   }
    // }

    stage('Docker deployment') {
      steps {
         script {
            if(env.BRANCH_NAME == "master") {
              echo "Running Docker Image Master"
              bat "docker run --name c-${username}-${env.BRANCH_NAME} -d -p=${masterport}:7100 ${dockerImage}:${BUILD_NUMBER}"
            } else {
              echo "Running Docker Image Dev"
              bat "docker run --name c-${username}-${env.BRANCH_NAME} -d -p=${devport}:7100 ${dockerImage}:${BUILD_NUMBER}"
            }
         }
      } 
    }

    stage('k8s Run') {
      steps {
        echo "k8s"
        bat "gcloud container clusters get-credentials autopilot-cluster-1 --region us-central1 --project unique-iterator-321302"
        bat "kubectl apply -f k8s/deployment.yaml"
      }
    }
  }
}