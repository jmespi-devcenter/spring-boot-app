def versionPom = ""
pipeline{
agent {
        kubernetes {
            // Rather than inline YAML, in a multibranch Pipeline you could use: yamlFile 'jenkins-pod.yaml'
            // Or, to avoid YAML:
            // containerTemplate {
            //     name 'shell'
            //     image 'ubuntu'
            //     command 'sleep'
            //     args 'infinity'
            // }
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: maven
    command:
    - sleep
    args:
    - infinity
  - name: jdk11
    image: alledodev/jenkins-nodo-java-bootcamp:latest
    command:
    - sleep
    args:
    - infinity
  - name: nodejs
    image: alledodev/jenkins-nodo-nodejs-bootcamp:latest
    command:
    - sleep
    args:
    - infinity
  - name: imgkaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - /busybox/cat
    tty: true
  - name: springboot
    image: jmespidevcenter/app-pf-backend:latest
    command:
    - sleep
    args:
    - infinity
'''
            // Can also wrap individual steps:
            // container('shell') {
            //     sh 'hostname'
            // }
            defaultContainer 'shell'
        }
    }
	stages {
        stage('Unit Tests') {
            steps {
            echo '''04# Stage - Unit Tests
(develop y main): Lanzamiento de test unitarios.
'''
                sh "mvn test"
                sh "pwd"
                junit "target/surefire-reports/*.xml"
            }
        }
        stage('SonarQube analysis') {
          steps {
            withSonarQubeEnv(credentialsId: "sonarid", installationName: "SonarQube"){
                sh "mvn clean verify sonar:sonar -DskipTests"
            }
          }
        }

      stage ('Quality Gate') {
          steps {
            timeout(time: 10, unit: "MINUTES") {
              script {
                def qg = waitForQualityGate(webhookSecretId: 'sonarid')
                if (qg.status != 'OK') {
                   error "Pipeline aborted due to quality gate failure: ${qg.status}"
                }
              }
            }
          }
        }
  stage('Package') {
            steps {
            echo '''07# Stage - Package
(develop y main): Generación del artefacto .jar (SNAPSHOT)
'''
                sh 'mvn package -DskipTests'
            }
        }
	
  /*stage('Build & Push') {
            steps {
            echo '''08# Stage - Build & Push
(develop y main): Construcción de la imagen con Kaniko y subida de la misma a repositorio personal en Docker Hub.
Para el etiquetado de la imagen se utilizará la versión del pom.xml
'''
                container('imgkaniko') {
                   
                    script {
                        def APP_IMAGE_NAME = "app-pf-backend"
                        def APP_IMAGE_TAG = "0.0.1" //Aqui hay que obtenerlo de POM.txt
                        withCredentials([usernamePassword(credentialsId: 'idCredencialesDockerHub', passwordVariable: 'idCredencialesDockerHub_PASS', usernameVariable: 'idCredencialesDockerHub_USER')]) {
                            AUTH = sh(script: """echo -n "${idCredencialesDockerHub_USER}:${idCredencialesDockerHub_PASS}" | base64""", returnStdout: true).trim()
                            command = """echo '{"auths": {"https://index.docker.io/v1/": {"auth": "${AUTH}"}}}' >> /kaniko/.docker/config.json"""
                            sh("""
                                set +x
                                ${command}
                                set -x
                                """)
                            sh "/kaniko/executor --dockerfile Dockerfile --context ./ --destination ${idCredencialesDockerHub_USER}/${APP_IMAGE_NAME}:${APP_IMAGE_TAG}"
                            sh "/kaniko/executor --dockerfile Dockerfile --context ./ --destination ${idCredencialesDockerHub_USER}/${APP_IMAGE_NAME}:latest --cleanup"
                        }
                    }
                } 
            }
        }
 stage('Probar mi imagen'){
    steps{
    container('springboot'){
      script{
      echo "soy un contenedor hecho con mi imagen"
      sh 'sleep infinity'
          }        
        }
      }
    } */
          //11
        stage('Nexus') {
            environment {
		        NEXUS_VERSION = "nexus3"
                NEXUS_PROTOCOL = "http"
                NEXUS_URL = "http://192.168.88.251:8081/repository/upskilling/"
                NEXUS_REPOSITORY = "upskilling/"
                NEXUS_CREDENTIAL_ID = "idnexus"
            }
            steps {
            echo '''11# Stage - Nexus
(develop y main): Si se ha llegado a esta etapa sin problemas:
Se deberá depositar el artefacto generado (.jar) en Nexus.(develop y main)
Generación del artefacto .jar (SNAPSHOT)
'''
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
  
