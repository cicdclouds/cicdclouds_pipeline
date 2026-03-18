pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "cicdclouds-java-app"
        // Use a static tag or build number for local tracking
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                // Ensure the branch name matches (main or master)
                git branch: 'main', url: 'https://github.com/cicdclouds/cicdclouds_javawebappi.git'
            }
        }

        stage('Maven Build') {
            steps {
                // Generates the .war file in the 'target' directory
                sh 'mvn clean package'
            }
        }

        stage('Local Docker Build') {
            steps {
                // This builds the image locally on your server
                sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
                sh "docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest"
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo "Copying WAR to local Tomcat..."
                // Adjust the path to where your Tomcat 'webapps' folder is located
                sh "cp target/*.war /var/lib/tomcat9/webapps/app.war"
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    echo "Deploying to Minikube..."
                    // This tells Minikube to use the image you just built locally
                    sh "kubectl set image deployment/java-web-deployment java-container=${DOCKER_IMAGE}:${IMAGE_TAG}"
                }
            }
        }
    }

    post {
        always {
            echo "Build Finished."
        }
    }
}
