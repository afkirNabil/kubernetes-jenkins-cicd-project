pipeline {

    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod

spec:
  containers:

    - name: python
      image: python:3.13-slim
      command:
        - sleep
      args:
        - 99d

    - name: kanikoi
      image: ghcr.io/osscontainertools/kaniko:debug
      command:
        - /busybox/cat
      tty: true

    - name: kubectl
      image: alpine/k8s:1.34.1
      command:
        - cat
      tty: true
'''
        }
    }

    environment {
        IMAGE_NAME = 'nabilas/jenkis-cicd-pv-app'
    }

    stages {

        stage('Test') {
            steps {
                container('python') {
                    sh '''
                        pip install -r requirements.txt
                        python -m py_compile app.py
                    '''
                }
            }
        }

        stage('Build and Push') {
            steps {
                container('kaniko') {

                    withCredentials([
                        usernamePassword(
                            credentialsId: 'dockerhub-creds',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_TOKEN'
                        )
                    ]) {

                        sh '''
                            mkdir -p /kaniko/.docker

                            AUTH=$(printf "%s:%s" "$DOCKER_USER" "$DOCKER_TOKEN" | base64 | tr -d '\\n')

                            cat > /kaniko/.docker/config.json <<EOF
{
  "auths": {
    "https://index.docker.io/v1/": {
      "auth": "$AUTH"
    }
  }
}
EOF

                            /kaniko/executor \
                                --context "$WORKSPACE" \
                                --dockerfile "$WORKSPACE/Dockerfile" \
                                --destination "$IMAGE_NAME:$BUILD_NUMBER"
                        '''
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                container('kubectl') {
                    sh '''
                        kubectl set image \
                          deployment/jenkins-cicd-pv-app \
                          jenkis-cicd-pv-app=$IMAGE_NAME:$BUILD_NUMBER \
                          -n default

                        kubectl rollout status \
                          deployment/jenkins-cicd-pv-app \
                          -n default
                    '''
                }
            }
        }
    }
}
