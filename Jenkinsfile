pipeline {

    agent {
        label 'slave'
    }

    stages {

        stage('Maven Build') {
            steps {
                sh '''
                cd samplejar
                mvn clean package
                '''
            }
        }


        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    withCredentials([
                        string(credentialsId: 'sonarnew', variable: 'SONAR_TOKEN')
                    ]) {

                        sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=wetech \
                        -Dsonar.projectName=wetech \
                        -Dsonar.sources=samplejar/src \
                        -Dsonar.java.binaries=samplejar/target/classes
                        '''
                    }
                }
            }
        }


        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }


        stage('Upload Artifact to Nexus') {
            steps {

                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: 'NEXUS_SERVER:8081',
                    groupId: 'classroom',
                    version: '1.0-SNAPSHOT',
                    repository: 'kalai',
                    credentialsId: 'NEXUS_CREDENTIAL_ID',

                    artifacts: [
                        [
                            artifactId: 'demo',
                            classifier: '',
                            file: 'samplejar/target/demo-1.0-SNAPSHOT.jar',
                            type: 'jar'
                        ]
                    ]
                )
            }
        }


        stage('Docker Build') {
            steps {
                sh '''
                docker build -t demo-app:v1 .
                '''
            }
        }

    }
}
