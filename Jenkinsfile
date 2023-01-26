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

        stage('Quality Gate') {
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

	}

}