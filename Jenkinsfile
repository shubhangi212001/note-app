pipeline {
    agent any
     tools{
        jdk 'jdk'
    }
    environment {
        SCANNER_HOME=tool 'sonar-server'
        GITHUB_CREDENTIALS = credentials('github-credentials')
        GITHUB_USERNAME = 'github-credentials.username'
        GITHUB_PASSWORD = 'github-credentials.password'
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
       stage('OWASP DP SCAN') {
           steps {
               dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'dp-check'
               dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
           }
       }
        stage('Convert XML to HTML') {
            steps {
                script {
                    // Define input and output file paths
                    def inputFile = 'https://github.com/shubhangi212001/note-app/blob/main/dependency-check-report.xml'
                    def outputFile = 'https://github.com/shubhangi212001/note-app/blob/main/dependency-check-report.html'

                    // Fetch the XML file from GitHub using credentials
                    withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'GITHUB_USERNAME', passwordVariable: 'GITHUB_PASSWORD')]) {
                        sh "curl -u $GITHUB_USERNAME:$GITHUB_PASSWORD -LJO https://raw.githubusercontent.com/shubhangi212001/note-app/main/dependency-check-report.xml"
                    }

                    // Check if XML file exists
                    if (!fileExists(inputFile)) {
                        error "Failed to fetch XML file from GitHub"
                    }
                    
                    // Execute the command to convert XML to HTML
                    sh "xsltproc --output ${outputFile} ${inputFile}"

                    // Check if conversion was successful
                    if (fileExists(outputFile)) {
                        echo "XML file converted to HTML successfully"
                    } else {
                        error "Conversion failed. HTML file not generated."
                    }
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
