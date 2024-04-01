pipeline {
    agent any
    environment {
        //sonarqube env variables
        SONAR_SCANNER='/opt/sonar-scanner' 
        SONAR_URL= 'http://localhost:9000'
        SONAR_PROJECTKEY='react'
        SONAR_LOGIN='sqp_5ecda522e2bfd890796bbe764381d30dae231b99'
        // harbor env variables
        HARBOR = credentials('harbour123')
        HARBOR_REGISTRY = 'new-harbor.duckdns.org'
        HARBOR_PROJECT = 'new_project'
        IMAGE_NAME = 'react'
        IMAGE_TAG = 'latest'
        IMAGE_PORT= '8000:8000'
    }
    stages {
        // sonar code quality check
        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv(credentialsId: 'sonarqube', installationName: 'sonarqube') {
                    sh """
                    \${SONAR_SCANNER}/bin/sonar-scanner \
                    -Dsonar.projectKey=$SONAR_PROJECTKEY \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=$SONAR_URL \
                    -Dsonar.login=$SONAR_LOGIN
                    """
                }
            }
        }
        // docker image build using dockerfile
        stage('Docker Build') {
            steps {
                sh 'docker image build -t $IMAGE_NAME .'
            }
        }
        // push docker image to Harbor
         stage('Push to Harbor Registry') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'harbor123', usernameVariable: 'HARBOR_USER', passwordVariable: 'HARBOR_PASSWORD')]) {
                        sh "docker login $HARBOR_REGISTRY -u $HARBOR_USER -p $HARBOR_PASSWORD"
                        sh "docker tag IMAGE_NAME $HARBOR_REGISTRY/$HARBOR_PROJECT/$IMAGE_NAME:$IMAGE_TAG"
                        sh "docker push $HARBOR_REGISTRY/$HARBOR_PROJECT/$IMAGE_NAME:$IMAGE_TAG"
                    }
                }
            }
        }
         //Run the container
        stage('Run') {
            steps {
                sh 'docker run -d -p $IMAGE_PORT --name python $HARBOR_REGISTRY/$HARBOR_PROJECT/$IMAGE_NAME'
            }
        }
        //trivy image scanner
        stage('Trivy image scan') {
            steps {
                sh 'trivy image $IMAGE_NAME'
            }
        }
    }
    // email notification
          post {
        success {
            // Send Slack notification on successful build
            slackSend(
                color: '#00FF00',
                message: "Build successful: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
            )
        }
        failure {
            // Send Slack notification on build failure
            slackSend(
                color: '#FF0000',
                message: "Build failed: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
            )
        }
    }
}
