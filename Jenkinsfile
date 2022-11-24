def versionPom = ""
pipeline{
  agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: chikitor/jenkins-nodo-java-bootcamp:1.0
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-socket-volume
    securityContext:
    privileged: true
  volumes:
  - name: docker-socket-volume
    hostPath:
      path: /var/run/docker.sock
      type: Socket
    command:
    - sleep
    args:
    - infinity
  '''
         defaultContainer 'shell'
        }
    }
  //environment {
  //      NEXUS_VERSION = "nexus3"
  //      NEXUS_PROTOCOL = "http"
  //      NEXUS_URL = "192.168.67.3:8081"
  //      NEXUS_REPOSITORY = "bootcamp"
  //      NEXUS_CREDENTIAL_ID = "Ferdevcenter" 
  //      DOCKERHUB_CREDENTIALS=credentials("Ferdevcenterdockerhub")
  //      DOCKER_IMAGE_NAME="chikitor/spring-boot-app" 
  // }

    environment {
        registryCredential='DockerId'
        registryBackend = 'chikitor/spring-boot-app'
    }

    stages{ 

      stage("build"){
        steps{
          sh "mvn clean package -DskipTests"
        }
      }

        stage('Push Image to Docker Hub') {
          steps {
            script {
              dockerImage = docker.build registryBackend + ":$BUILD_NUMBER"
              docker.withRegistry( '', registryCredential) {
                dockerImage.push()
              }
            }
          }
        }

        stage('Push Image latest to Docker Hub') {
          steps {
            script {
              dockerImage = docker.build registryBackend + ":latest"
              docker.withRegistry( '', registryCredential) {
                dockerImage.push()
              }
            }
          }
        }
   
      stage("deploy to k8s") {
            steps{
                sh "git clone https://github.com/Ferdevcenter/kubernetes-helm-docker-config.git configuracion --branch test-implementation"
                sh "kubectl apply -f configuracion/kubernetes-deployment/spring-boot-app/manifest.yml --kubeconfig=configuracion/kubernetes-config/config"
            }
      }
      stage ("Run API Test") {
            steps{
                node("node-nodejs"){
                    script {
                        if(fileExists("spring-boot-app")){
                            sh 'rm -r spring-boot-app'
                        }
                        sleep 15 // seconds
                        sh 'git clone https://github.com/Ferdevcenter/spring-boot-app.git spring-boot-app --branch api-test-implementation'
                        sh 'newman run spring-boot-app/src/main/resources/bootcamp.postman_collection.json --reporters cli,junit --reporter-junit-export "newman/report.xml"'
                        junit "newman/report.xml"

                    }

                }
            }
      }
    }
}
