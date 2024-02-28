pipeline {
    agent any
    
    stages {
        stage('code') {
            steps {
                echo 'Cloning the code'
                git "https://github.com/shubhangi212001/note-app.git", branch: "main"
            }
        }
        stage('Build') {
            steps {
                echo 'Building the image'
                sh "docker build -t my-note-app ."
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
        stage('Deploy') {
            steps {
                echo 'Deploying the container'
                sh "docker-compose down && docker-compose up -d"
            }
        }
    }
}
