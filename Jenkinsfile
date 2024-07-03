pipeline {

  environment {
    dockerimagename = "react-app"
    dockerImage = ""
  }

  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: docker
            image: docker:latest
          env:
          - name: DOCKER_TLS_CERTDIR
            value: ""
            command:
            - cat
            tty: true
            volumeMounts:
             - mountPath: /var/run/docker.sock
               name: docker-sock
          volumes:
          - name: docker-sock
            hostPath:
              path: /var/run/docker.sock    
        '''
    }
  }
  stages {
    
    stage('Checkout Source') {
      steps {
        script {
          git branch: 'main',
          url: 'https://github.com/nebbog/jenkins-kubernetes-deployment.git'
        }
      }
    }

   stage('Build image') {
      steps{
        container('docker') {
          script {
            dockerImage = docker.build dockerimagename
          }
        }
      }
    }
   
   stage('Pushing Image') {
     environment {
       DOCKER_INSECURE_REGISTRY = 'labtest.local:5000'
    }
      steps{
        container('docker') {
          script {
            docker.withRegistry( 'https://labtest.local:5000', '' ) {
              dockerImage.push("latest")
            }
          }
        }
      }
    }

    stage('Deploying React.js container to Kubernetes') {
      steps {
        script {
          kubernetesDeploy(configs: "deployment.yaml", "service.yaml")
        }
      }
    }

  }

}
