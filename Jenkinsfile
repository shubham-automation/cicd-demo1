pipeline {
    agent any

    stages {
        // stage('Git Clone') {
        //     steps {
        //         // Checkout a specific branch of your Git repository
        //         git branch: 'your-branch-name', url: 'https://github.com/your/repo.git'
        //     }
        // }
        
        stage('Docker Build') {
            steps {
                script {
                    sh "ls -lrt"
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
