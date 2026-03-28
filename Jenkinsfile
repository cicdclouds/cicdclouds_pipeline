pipeline {
    agent any

    environment {
        // FIX 1: Added your username so Docker Hub knows who owns the image
        DOCKER_HUB_USER = "your_dockerhub_username_here" // <-- CHANGE THIS!
        DOCKER_IMAGE = "${DOCKER_HUB_USER}/cicdclouds-java-app"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/cicdclouds/cicdclouds_javawebappi.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
                sh "docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest"
            }
        }

        stage('Push to Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    
                    // FIX 2: Network diagnostic test (The "|| true" stops it from failing the build if curl fails)
                    sh 'curl -v -I https://auth.docker.io/token || true'
                    
                    // FIX 3: Used SINGLE quotes here for security
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    
                    sh "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    sh "kubectl set image deployment/java-web-deployment java-container=${DOCKER_IMAGE}:${IMAGE_TAG}"
                }
            }
        }
    }

    post {
        always {
            echo "Build Finished."
            sh 'docker logout || true' // Added a quick cleanup step
        }
    }
}
