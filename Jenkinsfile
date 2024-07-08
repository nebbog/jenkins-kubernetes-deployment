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
        withKubeConfig([caCertificate: 'MIIDBjCCAe6gAwIBAgIBATANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwptaW5pa3ViZUNBMB4XDTI0MDYzMDA4MjU0M1oXDTM0MDYyOTA4MjU0M1owFTETMBEGA1UEAxMKbWluaWt1YmVDQTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMUDic/4KVi6arliEG1X+13YmcD1cISL3Zz/qVJpo5Ymr38xP+yDfr27cO+kfuLyJ0l0wZOc9wWhaY8QAQq0zo44a89k8bdb3VyPDk4qIgJf6CCFBi+T4kUpvBn61NtGiGfcSpnPYk/QYHQNG5P6P3m+FXG6lqjqcjTd7KYHgeOUZBPZRNLB9f99hjicoKUl8RQfk+t7tPAuHtvjOp5IVnJAE9S22N3nv2kDnJrE0P8aQkZEBsnH9tVg9k3htEyHPY939hrfCJ2VfUPKQp5fJh5n74oH+Y+P7z9ZXkIxC62TVDwvQIl66i4wLyhrUlkiJ1VX63mFVpXcusHjcK83KJcCAwEAAaNhMF8wDgYDVR0PAQH/BAQDAgKkMB0GA1UdJQQWMBQGCCsGAQUFBwMCBggrBgEFBQcDATAPBgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBSv1CBk7XV+AHfbKVtgtULpScMvZjANBgkqhkiG9w0BAQsFAAOCAQEAimjVGI3Wn4wMVx0wt/Z/Z8YNYc/MIu1pe/+lKySkxBtvyw+UHql/YnFLLaEGr/UZRhKiN29L98LdKOoVayzScnkp92wTZyuhq3+zcxSzWLocKcmo2kUx1pa86+FVdxkG7xDHZQoRCUmnWnINaJiNpfZaiyEFUiQWV2fDqfqQT/CWFH/lbZex6LDtwVFFHVo34bWAG/lLNi+CK1Qefkoo2j2tcLdl/9uyv69TAIY7La71rTHF5Xnu76QnC50BKHaKLmJcWyYFLXKCDbDk0LdEDkVn8CTSli3sQUUE5hblt6sJziC3pU5Gg6lKbsLLZiV1NncZ7CLCd1FkLPBn12offA==', credentialsId: 'jenkins-deployer-secret', serverUrl: 'https://kubernetes.default.svc']) {
          sh 'curl -LO "https://storage.googleapis.com/kubernetes-release/release/v1.30.2/bin/linux/amd64/kubectl"'  
          sh 'chmod u+x ./kubectl'  
          sh 'sleep 1000000'
          sh './kubectl get pods'
        }
       }
     }

  }

}
