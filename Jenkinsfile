pipeline {

  environment {
    dockerimagename = "nebbog/react-app"
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
               registryCredential = 'Dockerhub-credentials'
           }
      steps{
        container('docker') {
          script {
            docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {
            dockerImage.push("latest")
          }
        }
      }
    }
    }


   stage('Deploying React.js container to Kubernetes') {
     steps{
        withKubeConfig([caCertificate: '/var/run/secrets/kubernetes.io/serviceaccount/ca.crt', credentialsId: 'minikube-jenkins-secret', serverUrl: 'https://kubernetes.default.svc']) {
          sh 'cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt'
          sh 'ls -lrth /var/run/secrets/kubernetes.io/serviceaccount'
          sh 'cat /var/run/secrets/kubernetes.io/serviceaccount/token'
          sh 'curl -v https://kubernetes.default.svc'
          sh 'curl -LO "https://storage.googleapis.com/kubernetes-release/release/v1.30.2/bin/linux/amd64/kubectl"'  
          sh 'chmod u+x ./kubectl'  
          sh './kubectl get pods'
        }
       }
     }

  }

}
