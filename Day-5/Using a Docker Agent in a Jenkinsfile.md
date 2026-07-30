Module 3 – Using a Docker Agent in a Jenkinsfile
==================================================
Example:

pipeline {
    agent {
        docker {
            image 'my-jenkins-agent:v1'
        }
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}

What happens?
-----------------
Jenkins pulls my-jenkins-agent:v1.
Starts a container.
Mounts the workspace into the container.
Executes the pipeline inside that container.
Removes the container when the job finishes.
