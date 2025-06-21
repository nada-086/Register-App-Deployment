pipeline {
    agent any

    tools {
        jdk "Java17"
        maven "Maven3"
    }

    stages {
        stage ('Clean Up') {
            steps {
                cleanWs()
            }
        }

        stage ('Checkout from SCM') {
            steps {
                git branch: 'master', url: 'https://github.com/nada-086/Register-App-Deployment.git'
            }
        }

        stage ('Trivy File Scan') {
            steps {
                sh "trivy fs --format table -o fs.html ."         
            }
        }

        stage ('Application Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage ('Application Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage ('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonarqube-token') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }

        stage ('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-token'
                }
            }
        }

        stage ('Build Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-token', passwordVariable: 'dockerhubpass', usernameVariable: 'dockerhubuser')]) {
                    sh "docker login -u ${dockerhubuser} -p ${dockerhubpass}"
                }
                sh '''
                    docker build -t nadaessa/register-app:v${BUILD_NUMBER} .
                '''
            }
        }

        stage ('Scanning the Image using Trivy') {
            steps {
                sh '''
                trivy image --format table --output register-app-scan.html nadaessa/register-app:v${BUILD_NUMBER}
                '''
            }
        }

        stage ('Push the Docker Image to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-token', passwordVariable: 'dockerhubpass', usernameVariable: 'dockerhubuser')]) {
                    sh "docker login -u ${dockerhubuser} -p ${dockerhubpass}"
                }
                sh '''
                    docker build -t nadaessa/register-app:v${BUILD_NUMBER} .
                '''
            }
        }
    }
}