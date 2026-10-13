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
                        /home/jenkinsadm/sonar-scanner-8.0.1.6346-linux-x64/bin/sonar-scanner \
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
                    nexusUrl: '54.160.156.132:8081',
                    groupId: 'classroom',
                    version: '1.0-SNAPSHOT',
                    repository: 'kalai',
                    credentialsId: '70fcb998-9865-4f49-8796-433463e2917b',

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


stage('Docker Push') {
    steps {
        withCredentials([
            usernamePassword(
                credentialsId: 'dockerhub-creds',
                usernameVariable: 'DOCKER_USER',
                passwordVariable: 'DOCKER_PASS'
            )
        ]) {
            sh '''
                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                docker tag demo-app:v1 $DOCKER_USER/demo-app:v1

                docker push $DOCKER_USER/demo-app:v1
            '''
        }
    }
}

stage('Deploy to Kubernetes') {
    steps {
        sh '''
        kubectl apply -f kubernetes/
        kubectl rollout status deployment/demo-app
        kubectl get pods
        '''
    }
}
    }
}
