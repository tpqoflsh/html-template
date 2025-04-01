pipeline {
  agent none
   
  environment {
    ACR_NAME = "jdptest.azurecr.io"
    IMAGE_NAME = "jdp-web-app"
    IMAGE_TAG = "v${env.BUILD_NUMBER}"
    DEPLOY_PATH = "deploy/deployment.yaml"
  }
  stages {
    stage('Checkout') {
      agent { label 'built-in' }
      steps {
        git url: 'https://github.com/tpqoflsh/html-template.git', branch: 'master'
      }
    }
    stage('Build & Push Docker Image') {
      agent { label 'built-in' }
      steps {
        withCredentials([usernamePassword(credentialsId: 'jdp-acr', usernameVariable: 'ACR_USER', passwordVariable: 'ACR_PASS')]) {
            sh """
              which az || echo 'az not found'
              az acr login --name $ACR_NAME
              az acr build --registry $ACR_NAME --image $IMAGE_NAME:$IMAGE_TAG .
            """
            
            /*"""
              echo $ACR_PASS | docker login $ACR_NAME -u $ACR_USER --password-stdin
              docker build -t $ACR_NAME/$IMAGE_NAME:$IMAGE_TAG .
              docker push $ACR_NAME/$IMAGE_NAME:$IMAGE_TAG
            """*/
        }
      }
    }
    stage('Update YAML & Push') {
      agent { label 'built-in' }
      steps {
        withCredentials([
            usernamePassword(credentialsId: 'jdp-acr', usernameVariable: 'ACR_USER', passwordVariable: 'ACR_PASS'),
            usernamePassword(credentialsId: 'jdp-github', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')
        ]) {
            sh """
              sed -i "s|image: .*|image: $ACR_NAME/$IMAGE_NAME:$IMAGE_TAG|g" $DEPLOY_PATH
              git config user.name "$GIT_USER"
              git config user.email "tpqoflsh@gmail.com"
              git add $DEPLOY_PATH
              git commit -m "update image to $IMAGE_TAG"
              git push https://$GIT_USER:$GIT_PASS@github.com/tpqoflsh/html-template.gi main
            """
        }
      }
    }
  }
}
