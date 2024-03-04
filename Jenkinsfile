pipeline {
    agent any
    tools{
        jdk 'jdk'
    }
    environment {
        SCANNER_HOME=tool 'sonar-server'
    }
    stages {
        stage('Workspace Cleaning'){
            steps{
                cleanWs()
            }
        }  
        stage("Clone Code"){
            steps{
                git url: "https://github.com/LondheShubham153/django-notes-app.git", branch: "main"
            }
        }
        stage('OWASP DP SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'dp-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Build and Test"){
            steps{
                sh "docker build . -t note-app-test-new"
            }
        }
        stage('Push') {
            steps {
                echo 'Pushing docker image on Docker Hub'
                withCredentials([usernamePassword(credentialsId: 'DOCKERHUB', passwordVariable: 'dockerHubpass', usernameVariable: 'dockerHubuser')]) {
                    sh "docker tag my-note-app ${env.dockerHubuser}/my-note-app:latest"
                    sh "docker login -u ${env.dockerHubuser} -p ${env.dockerHubpass}"
                    sh "docker push ${env.dockerHubuser}/my-note-app:latest"
                }
            }
        }
        stage("Deploy"){
            steps{
                sh "docker-compose up -d"
            }
        }
    }
}
