pipeline {
    agent any

    stages {        
        stage('Run Automation Test Cases') {
            steps {
                script {
                    sh "ls -lrt"
                    sh "pip3 install -r requirements.txt"
                    sh "python3 test.py"
                }
            }
          // post {
          //   always {
          //     junit 'test-reports/*.xml'
          //   }
          // } 
         post {
           always {
             testNG(reportFilenamePattern: '/var/lib/jenkins/jobs/test/builds/8/testng/testng-results.xml')
           }
         }
        }
    }
}
