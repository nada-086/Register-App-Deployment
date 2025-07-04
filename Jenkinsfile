pipeline {
    agent any

    tools {
        jdk "Java17"
        maven "Maven3"
    }

    environment {
        APP_NAME = "nadaessa/register-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${APP_NAME}:${IMAGE_TAG}"
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
                    docker build -t ${IMAGE_NAME} .
                '''
            }
        }

        stage ('Scanning the Image using Trivy') {
            steps {
                sh '''
                    trivy image --format table --output register-app-scan.html ${IMAGE_NAME}
                '''
            }
        }

        stage ('Push the Docker Image to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-token', passwordVariable: 'dockerhubpass', usernameVariable: 'dockerhubuser')]) {
                    sh "docker login -u ${dockerhubuser} -p ${dockerhubpass}"
                }
                sh '''
                    docker push ${IMAGE_NAME}
                '''
            }
        }

        stage ('ArgoCD Trigger') {
            steps {
                sh """
                    sed -i 's/${APP_NAME}.*/$IMAGE_NAME/g' ./k8s/deployment.yaml
                    cat ./k8s/deployment.yaml
                """
            }
        }

        stage("Push the changed deployment file to Git") {
            steps {
                sh '''
                    git config --global user.name "Nada Essa"
                    git config --global user.email "nadaessa086@gmail.com"
                    git add ./k8s/deployment.yaml
                    git commit -m "Updated Deployment Manifest"
                '''
                withCredentials([usernamePassword(credentialsId: 'github-token', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    sh '''
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/nada-086/Register-App-Deployment.git master
                    '''
                }
            }
        }
    }
}