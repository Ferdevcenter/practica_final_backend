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
    image: chikitor/jenkins-nodo-java-bootcamp:2.0
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
        registryCredential='chikitor'
        registryBackend = 'chikitor/spring-boot-app'
    }

    stages{ 

      stage("mostrar versiones"){
        steps{
          sh "java --version"
          sh "mvn --version"
        }  
      }

      stage("build"){
        steps{
          sh "mvn clean package -DskipTests"
        }
      }

      stage("compile"){
        steps{
          sh "mvn compile package"
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
      stage("Test"){
        steps{
          sh "mvn test"
          jacoco()
          junit "target/surefire-reports/*.xml"
        }
      }
      //# HACEMOS PRUEBAS CON NEWMAN
      stage ("Run API Test") {
        steps{
          node("node-nodejs"){
           script {
               if(fileExists("spring-boot-app")){
                   sh 'rm -r spring-boot-app'
               }
               sleep 15 // seconds
               sh 'git clone https://github.com/dberenguerdevcenter/spring-boot-app.git spring-boot-app --branch api-test-implementation'
               sh 'newman run spring-boot-app/src/main/resources/bootcamp.postman_collection.json --reporters cli,junit --reporter-junit-export "newman/report.xml"'
               junit "newman/report.xml"
            }
          }
        }
      }  




      //# HACEMOS LAS PRUEBAS PARA JMETER
      stage ("Setup Jmeter") {
        steps{
          script {

            if(fileExists("jmeter-docker")){
                sh 'rm -r jmeter-docker'
            }

            sh 'git clone https://github.com/Ferdevcenter/jmeter-docker.git'

            dir('jmeter-docker') {

               if(fileExists("apache-jmeter-5.5.tgz")){
                   sh 'rm -r apache-jmeter-5.5.tgz'
               }
               sh 'wget https://dlcdn.apache.org//jmeter/binaries/apache-jmeter-5.5.tgz'
               sh 'tar xvf apache-jmeter-5.5.tgz'
               sh 'cp plugins/*.jar apache-jmeter-5.5/lib/ext'
               sh 'mkdir test'
               sh 'mkdir apache-jmeter-5.5/test'
               sh 'cp ../src/main/resources/*.jmx apache-jmeter-5.5/test/'
               sh 'chmod +775 ./build.sh && chmod +775 ./run.sh && chmod +775 ./entrypoint.sh'
               sh 'rm -r apache-jmeter-5.5.tgz'
               sh 'tar -czvf apache-jmeter-5.5.tgz apache-jmeter-5.5'
               sh './build.sh'
               sh 'rm -r apache-jmeter-5.5 && rm -r apache-jmeter-5.5.tgz'
               sh 'cp ../src/main/resources/perform_test.jmx test'
            }
          }
        }
      }
      stage ("Run Jmeter Performance Test") {
          steps{
            script {
              dir('jmeter-docker') {
               if(fileExists("apache-jmeter-5.5.tgz")){
                   sh 'rm -r apache-jmeter-5.5.tgz'
               }
               sh './run.sh -n -t test/perform_test.jmx -l test/perform_test.jtl'
               sh 'docker cp jmeter:/home/jmeter/apache-jmeter-5.5/test/perform_test.jtl /home/jenkins/workspace/_app_perform-test-implementation/jmeter-docker/test'
               perfReport '/home/jenkins/workspace/_app_perform-test-implementation/jmeter-docker/test/perform_test.jtl'
              }
            }
          }
      }    
      stage ("Generate Taurus Report") {
        steps{
          script {
            dir('jmeter-docker') {
             sh 'pip install bzt'
             sh 'export PATH=$PATH:/home/jenkins/.local/bin'
              BlazeMeterTest: {
                 sh '/home/jenkins/.local/bin/bzt test/perform_test.jtl -report'
             }
            }
          }
        }
      }
      stage("deploy to k8s") {
            steps{
                sh "git clone https://github.com/Ferdevcenter/kubernetes-helm-docker-config.git configuracion --branch test-implementation"
//                sh "kubectl apply -f configuracion/kubernetes-deployment/spring-boot-app/manifest.yml --kubeconfig=configuracion/kubernetes-config/config"
            }
      }
//      stage ("Run API Test") {
//            steps{
//                node("node-nodejs"){
//                    script {
//                        if(fileExists("spring-boot-app")){
//                            sh 'rm -r spring-boot-app'
//                        }
//                        sleep 15 // seconds
//                       sh 'git clone https://github.com/Ferdevcenter/spring-boot-app.git spring-boot-app --branch api-test-implementation'
//                       sh 'newman run spring-boot-app/src/main/resources/bootcamp.postman_collection.json --reporters cli,junit --reporter-junit-export "newman/report.xml"'
//                        junit "newman/report.xml"

//                   }

//                }
//           }
//      }
//#LAS SIGUIENTES LLAVES SON LAS FINALES NO BORRAR
    }
    
}
