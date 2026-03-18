pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "cicdclouds/java-web-app"
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        TOMCAT_IP = "http://localhost:9090" // Change to your VM IP
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/cicdclouds/cicdclouds_javawebappi.git'
            }
        }

        stage('Build with Maven') {
            steps {
                // Assuming this is a Maven project; if Gradle, use ./gradlew build
                sh 'mvn clean package'
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    docker.withRegistry('', 'docker-hub-credentials-id') {
                        def appImage = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                        appImage.push()
                        appImage.push('latest')
                    }
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo "Deploying .war file to Tomcat..."
                // Uses scp to move the built war file to Tomcat's webapps folder
                sshagent(['tomcat-ssh-key']) {
                    sh "scp -o StrictHostKeyChecking=no target/*.war vboxuser@${TOMCAT_IP}:/var/lib/tomcat9/webapps/app.war"
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                echo "Updating Kubernetes Deployment..."
                // Updates the image in your K8s cluster
                sh "kubectl set image deployment/java-app-deployment java-app-container=${DOCKER_IMAGE}:${DOCKER_TAG}"
            }
        }
    }

    post {
        success {
            echo "Successfully deployed to both Tomcat and Minikube!"
        }
        failure {
            echo "Build failed. Check the Jenkins console output."
        }
    }
}
