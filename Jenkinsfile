pipeline {
    agent any
    environment {
        DOCKER_CREDS = credentials('docker123')
        DOCKER_IMAGE =sivapujitha
    }
    stages {
                 
        // docker image build using dockerfile 
        stage('Docker Build') {
            steps {
                sh 'docker image build -t react .'
            }
        }
        // push docker image to dockerhub
         stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker123', usernameVariable: 'DOCKER_CREDS_USR', passwordVariable: 'DOCKER_CREDS_PSW')]) {
                        sh 'docker login -u $DOCKER_CREDS_USR -p $DOCKER_CREDS_PSW'
                    }
                    sh 'docker tag react $DOCKER_IMAGE/react:latest'
                    sh 'docker push $DOKCER_IMAGE/react:latest'
                }
            }
        }
        // run the container using docker image
        stage('Run') {
            steps {
                sh 'docker run -d -p 3000:3000 --name react react'
            }
        }
        //trivy image scanner
        stage('Trivy image scan') {
            steps {
                sh 'trivy image react'
            }
        }
    }
}
