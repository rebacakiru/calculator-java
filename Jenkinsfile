pipeline {
    agent any

    tools {
        maven 'Maven-3'   // must match name in Global Tool Configuration
    }

    environment {
        APP_NAME = 'calculator-java'
    }

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {

        stage('Initialize') {
            steps {
                echo "Application: ${APP_NAME}"
                echo "Build Number: ${BUILD_NUMBER}"
                sh 'java -version'
                sh 'mvn -version'
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
                sh 'ls -la'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile -B'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test -B'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests -B'
                sh 'ls -la target'
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Build Docker Image') {
            when {
                expression { fileExists('Dockerfile') }
            }
            steps {
                sh '''
                  docker build -t calculator-app:${BUILD_NUMBER} .
                  docker tag calculator-app:${BUILD_NUMBER} calculator-app:latest
                '''
            }
        }
    }

    post {
        success {
            echo ' Pipeline completed successfully'
        }
        failure {
            echo ' Pipeline failed'
        }
        always {
            cleanWs()
        }
    }
}
