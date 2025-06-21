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
    }
}