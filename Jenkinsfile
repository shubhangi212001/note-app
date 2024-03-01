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
        stage("Code"){
            steps{
                git url: "https://github.com/shubhangi212001/note-app.git", branch: "main"
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=django-note-app \
                    -Dsonar.projectKey=django-note-app
                    '''
                }
            }
        }

        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        } 
        stage("Sonarqube Analysis"){
            steps{
                withSonarQubeEnv('sonar-scanner') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=django-note-app \
                    -Dsonar.projectKey=django-note-app \
                    '''
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-scanner' 
                }
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
    post {
     always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'aman07pathak@gmail.com',
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
}
