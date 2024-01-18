# cicd-demo1
CI/CD Demo 1 Presentation

1. aws eks update-kubeconfig --name ci-cd-demo1  --region us-east-1


2. kubectl edit cm aws-auth -n kube-system 

    - groups:
      - system:masters
      rolearn: <Jenkins_Instance_Role_ARN>
      username: <Jenkins_Instance_Role_Name>

3. ./istioctl install --set profile=demo -y

4. kubectl label namespace default istio-injection=enabled

5. java -jar jenkins-cli.jar -s http://admin:admin@3.91.61.55:8080 create-credentials-by-xml  system::system::jenkins _ < docker-creds.xml

6. java -jar jenkins-cli.jar -s http://admin:admin@3.91.61.55:8080 create-credentials-by-xml  system::system::jenkins _ < git-creds.xml

7. java -jar jenkins-cli.jar -s http://admin:admin@3.91.61.55:8080 create-job demo1 < latest-pipeline.xml

8. kubectl get svc -A | grep istio-system 

9. CURRENT_APP_VERSION = FRESH/FIRST deployment
