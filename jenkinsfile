pipeline {
    agent any
    environment {
        APP_NAME = "testapp"   // context path
        WAR_NAME = "cicdclouds-api.war" // artifact name
    }
    stages {
        stage('Download') {
            steps {
                git branch: 'main', url: 'https://github.com/cicdclouds/cicdclouds_javawebappi.git'
            }
        }
        stage('Build') {
            steps {
                bat 'mvn clean package'
            }
        }
        stage('Deployment') {
            steps {
                bat """curl -u admin:admin123 -T "%WORKSPACE%\\target\\${WAR_NAME}" "http://localhost:9090/manager/text/deploy?path=/${APP_NAME}&update=true"""
            }
        }
        stage('Testing') {
            steps {
                git branch: 'main', poll: false, url: 'https://github.com/cicdclouds/cicdclouds_testwebappi.git'
                bat "java -Dapp.host=localhost -Dapp.port=9090 -Dapp.context=${APP_NAME} -jar target/functionalapi-tests.jar"
            }
        }
    }
}
