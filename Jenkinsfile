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
  environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "192.168.67.3:8081"
        NEXUS_REPOSITORY = "bootcamp"
        NEXUS_CREDENTIAL_ID = "Ferdevcenter" 
        DOCKERHUB_CREDENTIALS=credentials("chikitor")
        DOCKER_IMAGE_NAME="chikitor/spring-boot-app" 
  // }

  //  environment {
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
      stage("Test"){
        steps{
          sh "mvn test"
          jacoco()
          junit "target/surefire-reports/*.xml"
        }
      }
      stage("build"){
        steps{
          sh "mvn clean compile package -DskipTests"
        }
      }

//      stage("compile"){
//        steps{
//          sh "mvn compile package"
//        }
//      }
//      stage('Push Image to Docker Hub') {
//        steps {
//          script {
//             dockerImage = docker.build registryBackend + ":$BUILD_NUMBER"
//             docker.withRegistry( '', registryCredential) {
//             dockerImage.push()
//             }
//          }
//        }
//      }
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
      
//      stage('NPM build') {
//        steps {
//          script {
//            sh 'npm install'
//            sh 'npm run build'
//          }
//        }
//      }
      stage('SonarQube analysis') {
        steps {
          withSonarQubeEnv(credentialsId: "sonarqube", installationName: "Sonarqube-server"){
            sh 'mvn clean package sonar:sonar'
          }
        }
      }
      stage('Quality Gate') {
        steps {
          timeout(time: 10, unit: "MINUTES") {
            script {
              def qg = waitForQualityGate()
              if (qg.status != 'OK') {
                 error "Pipeline aborted due to quality gate failure: ${qg.status}"
              }
            }
          }
        }
      }

      //# HACEMOS PRUEBAS CON NEWMAN
//      stage ("Run API Test") {
//        steps{
//           script {
//               if(fileExists("spring-boot-app")){
//                   sh 'rm -r spring-boot-app'
//               }
//               sleep 15 // seconds
//               sh 'git clone https://github.com/Ferdevcenter/practica_final_backend.git spring-boot-app --branch develop'
//               sh 'newman run spring-boot-app/src/main/resources/bootcamp.postman_collection.json --reporters cli,junit --reporter-junit-export newman/report.xml'
//               junit "newman/report.xml"
//            }
//        }
//      }  
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
//                       sh 'git clone https://github.com/Ferdevcenter/spring-boot-app.git spring-boot-app --branch develop'
//                       sh 'newman run spring-boot-app/src/main/resources/bootcamp.postman_collection.json --reporters cli,junit --reporter-junit-export "newman/report.xml"'
//                        junit "newman/report.xml"

//                   }

//                }
//           }
//      }
//#LAS SIGUIENTES LLAVES SON LAS FINALES NO BORRAR
        stage("Publish to Nexus") {
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml"
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath                    
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"
                        versionPom = "${pom.version}"                        
                            nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: "",
                                file: artifactPath,
                                type: pom.packaging],                                
                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                                [artifactId: pom.artifactId,
                                classifier: "",
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        )                    
                        } else {
                        error "*** File: ${artifactPath}, could not be found"
                    }
                }
            }
        }
    }
    
}
