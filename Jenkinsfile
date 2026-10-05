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

'''
        }
    }

    stages {

        stage('Test') {
            steps {
                container('python') {
                    sh '''
                        python --version
                        pip install -r requirements.txt
                        python -m py_compile app.py
                    '''
                }
            }
        }

    }
}
