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
    }
}
