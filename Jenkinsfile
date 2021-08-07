pipeline {
    agent any
    options {
            timestamps()
    }
    environment {
        location = "dreamspace04/nagp-devops-nodejs"
        username = "dreamspace04"
        port=7100
    }
    tools {
        nodejs 'nodejs'
        jdk 'Java'
        dockerTool 'Test_Docker'
    }
    stages {
            stage('Git code checkout') {
                steps {
                    // One or more steps need to be included within the steps block.
		    checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-cred', url: 'https://github.com/niloymcs17/nagp-devops']]])
                }
            }
            stage('npm install') {
                steps {
                    bat 'npm install'
                }
            }
            // stage('npm test') {
            //     steps {
            //         bat 'npm test'
            //     }
            // }
            stage('SonarQube') {
                steps {
                    bat '..\\..\\tools\\hudson.plugins.sonar.SonarRunnerInstallation\\SonarQubeScanner\\bin\\sonar-scanner.bat -Dsonar.host.url=http://localhost:9000 -Dsonar.login=53a0f89afc930a3c200bb204b29c96c8a7cfe641'
                }
            }
          
            stage('Build Docker Image') {
                steps {
                    echo "Building Docker Image"
                    bat "docker build -t i-${username}-master ."
                }
            }
            stage('Docker Push') {
                steps {
                    echo "Tagging name with build number"
                    bat "docker tag i-${username}-master ${location}:${BUILD_NUMBER}"
  
                    withDockerRegistry([credentialsId: 'DockerHub',url:""]){
                        bat "docker push ${location}:${BUILD_NUMBER}"
                    }
                }
            }
            stage('Docker Run') {
                steps {
                    echo "Running Docker Image"
                    bat "docker run --name c-${username}-master -d -p=${port}:7100 ${location}:${BUILD_NUMBER}"
                }
            }
    }
}
