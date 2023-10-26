pipeline {
    agent any

    stages {        
        stage('Run Automation Test Cases') {
            steps {
                script {
                    sh "pip3 install -r requirements.txt"
                    sh "python3 test.py"
                }
            }
                post {
                  always {
                    junit 'test-reports/*.xml'
                  }
                }    
        }

        stage('Docker build') {
            steps {
                script {
                        def registryCredentials = [
                        credentialsId: 'docker-creds'
                        ]
                        withDockerRegistry(credentials: registryCredentials, registry: 'chaudharishubham2911') {
                        sh "docker build -t v1 ."
                        sh "docker tag v1:latest chaudharishubham2911/cicd-demo1:v1"
                        sh "docker push chaudharishubham2911/cicd-demo1:v1"
                    }
                }
            }
        }
    }
}
