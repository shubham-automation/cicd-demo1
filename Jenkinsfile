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
                  sh "trivy image --format template --template '@html.tpl' --output trivy_report.html --exit-code 0 --severity HIGH,CRITICAL chaudharishubham2911/cicd-demo1:${BRANCH}"
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

         stage('Blue/Green Deployment to EKS') {
           when {
             expression {
               def FRESH_DEPLOYMENT = env.FRESH_DEPLOYMENT
               return FRESH_DEPLOYMENT == 'No'
             }
            }
             steps {
                 script {
                        def greenWeight = input(
                        id: 'greenWeight',
                        message: 'Enter a deployment weightage as integer number:',
                        parameters: [
                            string(name: 'CustomValue', defaultValue: '', description: 'weightage value')
                        ],
                        submitter: 'user'
                        )

                        echo "User entered: ${greenWeight}"
                        sh """
                           #!/bin/bash
                           if [ -z ${greenWeight} ]; then
                             echo 'Please provide the deployment weightage as integer number'
                             exit 1
                           else
                              export "GREEN_WEIGHT=${greenWeight}"
                              export "BLUE_WEIGHT=`expr 100 - $greenWeight`"
                              aws eks update-kubeconfig --name ci-cd-demo1  --region us-east-1
                              envsubst < k8s/app.yaml | kubectl apply -f -
                              envsubst < k8s/istio.yaml | kubectl apply -f -
                              echo "Variable is not empty"
                           fi
                        """
                        
                        sh """
                        #!/bin/bash
                            kubectl get po | grep myapp-v2 | awk '{print \$1}'| xargs kubectl wait --for=condition=Ready pod -n default
                            api_result=`kubectl get po -l version=v2 -o custom-columns=:metadata.name | xargs -I {} kubectl exec -ti {} -- bash -c 'curl -s -o /dev/null -w "%{http_code}" http://localhost:9090/shubham'`
                            if [[ ${api_result} -eq 200 ]]; then
                              echo "V2 Application Version is Running Fine......."
                              export "GREEN_WEIGHT=100"
                              export "BLUE_WEIGHT=0"
                              export "BRANCH=${BRANCH}"
                              aws eks update-kubeconfig --name ci-cd-demo1  --region us-east-1
                              envsubst < k8s/istio.yaml | kubectl apply -f -
                            else
                              echo "Something Wrong with the V2 Application Version......."
                              export "GREEN_WEIGHT=0"
                              export "BLUE_WEIGHT=100"
                              export "BRANCH=v1"
                              aws eks update-kubeconfig --name ci-cd-demo1  --region us-east-1
                              envsubst < k8s/istio.yaml | kubectl apply -f -
                              echo "Application Rolled Back to Version V1......."
                            fi                           
                        """
                     }
                 }
            }

         stage('Deploy New Application to EKS') {
           when {
             expression {
               def FRESH_DEPLOYMENT = env.FRESH_DEPLOYMENT
               return FRESH_DEPLOYMENT == 'Yes'
             }
            }
             steps {
                 script {
                        sh """
                           #!/bin/bash
                              export "GREEN_WEIGHT=0"
                              export "BLUE_WEIGHT=100"
                              export "BRANCH=${BRANCH}"
                              aws eks update-kubeconfig --name ci-cd-demo1  --region us-east-1
                              envsubst < k8s/app.yaml | kubectl apply -f -
                              envsubst < k8s/istio.yaml | kubectl apply -f -
                        """
                     }
                 }
            }
    }
}

