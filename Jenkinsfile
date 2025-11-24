#!/usr/bin/env groovy

pipeline {
    agent any

    // keep history of builds
    options {
        buildDiscarder(logRotator(numToKeepStr: '50', artifactNumToKeepStr: '5'))
    }

    environment {
        PUBLISH = 'false'
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

        stage('Test') {
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
                    def imageTag = "${env.BUILD_NUMBER}"
                    sh "docker build -t simple‑web‑app:${imageTag} ."
                }
            }
        }

        stage('Run Container') {
            steps {
                script {
                    def imageTag = "${env.BUILD_NUMBER}"
                    sh "docker run -d --name webapp_${imageTag} -p 3000:3000 simple‑web‑app:${imageTag}"
                }
            }
        }

        stage('Publish (optional)') {
            when {
                expression { env.PUBLISH == 'true' }
            }
            steps {
                script {
                    def imageTag = "${env.BUILD_NUMBER}"
                    sh "docker tag simple‑web‑app:${imageTag} yourdockerhubuser/simple‑web‑app:${imageTag}"
                    sh "docker push yourdockerhubuser/simple‑web‑app:${imageTag}"
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build and deployment succeeded"
        }
        failure {
            echo "❌ Build failed"
        }
        always {
            cleanWs()
        }
    }
}
