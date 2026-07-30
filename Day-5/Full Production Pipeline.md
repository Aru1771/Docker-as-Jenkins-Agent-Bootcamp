Full Production Pipeline:
==========================

pipeline {
    agent {
        docker {
            image 'my-jenkins-agent:v1'
        }
    }

    stages {

        stage('Checkout') {
            steps {
                git url: 'https://github.com/company/app.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t app:1.0 .'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy image app:1.0'
            }
        }

        stage('Deploy') {
            steps {
                sh 'kubectl apply -f deployment.yaml'
            }
        }
    }
}


Because the agent image already contains all required tools, the pipeline is clean and consistent.
