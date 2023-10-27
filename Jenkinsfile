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

        stage('Docker Build') {
            steps {
                script {
                  sh "docker build -t ${BRANCH} ."
                  sh "docker tag ${BRANCH}:latest chaudharishubham2911/cicd-demo1:${BRANCH}"
                }
            }
        }

        stage('Docker Image Scanning') {
            steps {
                script {
                  sh "curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl > html.tpl"
                  sh "curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/asff.tpl > asff.tpl"
                  sh "sed -i '1d;\$d' asff.tpl"
                  sh "aws securityhub list-enabled-products-for-import --region us-east-1 | grep -q aquasecurity && echo "Aqua Security integration has already been enabled in SecurityHub" || { aws securityhub enable-import-findings-for-product --region us-east-1 --product-arn "arn:aws:securityhub:us-east-1::product/aquasecurity/aquasecurity" && echo "Enabled Aqua Security integration in SecurityHub"; }"
                  sh "trivy image --format template --template '@html.tpl' --output trivy_report.html --exit-code 1 --severity HIGH,CRITICAL chaudharishubham2911/cicd-demo1:${BRANCH}"
                  sh "trivy image --format template --template '@asff.tpl' --output trivy_report.asff --exit-code 1 --severity HIGH,CRITICAL chaudharishubham2911/cicd-demo1:${BRANCH}"
                  sh "aws securityhub batch-import-findings --findings file://trivy_report.asff --region us-east-1"
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: "trivy_report.html", fingerprint: true      
                    publishHTML (target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: '.',
                        reportFiles: 'trivy_report.html',
                        reportName: 'Trivy Scan',
                        ])
                    }
             }            
        }

        stage('Push Image Scanning Result to SecurityHub') {
            steps {
                script {
                  sh "curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl > html.tpl"
                  sh "trivy image --format template --template '@html.tpl' --output trivy_report.html --exit-code 1 --severity HIGH,CRITICAL chaudharishubham2911/cicd-demo1:${BRANCH}"
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                        def registryCredentials = [
                        credentialsId: 'docker-creds'
                        ]
                        withCredentials([usernamePassword(credentialsId: 'docker-creds', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                        sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                        sh "docker push chaudharishubham2911/cicd-demo1:${BRANCH}"
                    }
                }
            }
        }
    }
}
