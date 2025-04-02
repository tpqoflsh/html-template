pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: alpine:latest
    command:
    - /bin/sh
    args:
    - -c
    - sleep 99999
    tty: true
    volumeMounts:
    - name: kaniko-secret
      mountPath: /kaniko/.docker

  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    command: ["/kaniko/executor"]
    args:
    - --dockerfile=/workspace/\$(JOB_NAME)/Dockerfile
    - --context=dir:///workspace/\$(JOB_NAME)
    - --destination=jdptest.azurecr.io/jdp-web-app:\$(BUILD_NUMBER)
    - --verbosity=debug
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
