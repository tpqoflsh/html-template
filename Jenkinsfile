pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
spec:
  imagePullSecrets:
  - name: acr-secret
  containers:
  - name: kaniko
    image: jdptest.azurecr.io/jdp-kaniko:v1
    command:
    - bash
    args:
    - -c
    - sleep infinity
    volumeMounts:
    - name: kaniko-secret
      mountPath: /kaniko/.docker
  volumes:
  - name: kaniko-secret
    projected:
      sources:
      - secret:
          name: acr-secret
  restartPolicy: Never
"""
    }
  }

  environment {
    ACR_NAME = "jdptest.azurecr.io"
    IMAGE_NAME = "jdp-web-app"
    IMAGE_TAG = "v${env.BUILD_NUMBER}"
    DEPLOY_PATH = "deploy/deployment.yaml"
  }

  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/tpqoflsh/html-template.git', branch: 'master'
      }
    }

    stage('Build & Push using Kaniko') {
      steps {
        container('kaniko') {
          sh """
            echo "jdp"
            /kaniko/executor \
              --dockerfile=Dockerfile \
              --context=dir:///workspace/${JOB_NAME} \
              --destination=$ACR_NAME/$IMAGE_NAME:$IMAGE_TAG
              --verbosity=info
          """
        }
      }
    }

    stage('Update YAML & Push') {
      steps {
        withCredentials([
          usernamePassword(credentialsId: 'jdp-github', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')
        ]) {
          sh """
            sed -i "s|image: .*|image: $ACR_NAME/$IMAGE_NAME:$IMAGE_TAG|g" $DEPLOY_PATH
            git config user.name "$GIT_USER"
            git config user.email "tpqoflsh@gmail.com"
            git add $DEPLOY_PATH
            git commit -m "update image to $IMAGE_TAG"
            git push https://$GIT_USER:$GIT_PASS@github.com/tpqoflsh/html-template.git main
          """
        }
      }
    }
  }
}
