#!/usr/bin/env groovy

pipeline {
    agent {
        docker { 
            image 'node:20'   // Node.js environment
            args '-u root:root -v /var/run/docker.sock:/var/run/docker.sock' // to allow Docker build/run
        }
    }

    environment {
        PUBLISH = 'false' // change to 'true' to enable docker push
        IMAGE_NAME = 'simple-web-app'
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '50', artifactNumToKeepStr: '5'))
        timeout(time: 60, unit: 'MINUTES')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'test-results/**/*.xml'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:${env.BUILD_NUMBER} ."
                }
            }
        }

        stage('Run Container') {
            steps {
                script {
                    sh "docker run -d --name ${IMAGE_NAME}_${env.BUILD_NUMBER} -p 3000:3000 ${IMAGE_NAME}:${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Optional Publish') {
            when {
                expression { env.PUBLISH == 'true' }
            }
            steps {
                script {
                    sh """
                        docker tag ${IMAGE_NAME}:${env.BUILD_NUMBER} yourdockerhubuser/${IMAGE_NAME}:${env.BUILD_NUMBER}
                        docker push yourdockerhubuser/${IMAGE_NAME}:${env.BUILD_NUMBER}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build, test, and deployment completed successfully!"
        }
        failure {
            echo "❌ Build or tests failed."
        }
        always {
            cleanWs()
        }
    }
}
