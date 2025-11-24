#!/usr/bin/env groovy

// Jenkins job properties
def listOfProperties = []
listOfProperties << buildDiscarder(logRotator(numToKeepStr: '50', artifactNumToKeepStr: '5'))

if ((env.BRANCH_NAME ?: "").trim() == 'master') {
    listOfProperties << pipelineTriggers([cron('''H H/6 * * 0-2,4-6
H 6,21 * * 3''')])
}

properties(listOfProperties)

// Default environment variable
def envVars = ['PUBLISH=true']

// Dummy infra object for local builds
def infra = [
    isTrusted: { -> false },
    withDockerCredentials: { closure -> closure.call() }
]

// Linux image builds
def images = [
    'alpine_jdk17',
    'alpine_jdk21',
    'debian_jdk17',
    'debian_jdk21',
    'debian_slim_jdk17',
    'debian_slim_jdk21',
    'rhel_ubi9_jdk17',
    'rhel_ubi9_jdk21',
]

stage('Build') {
    withEnv(envVars) {

        echo '= bake target: linux'

        for (i in images) {
            def imageToBuild = i

            stage("Checkout ${imageToBuild}") {
                node {
                    deleteDir()
                    checkout scm
                }
            }

            stage("Static analysis ${imageToBuild}") {
                node {
                    sh 'make hadolint shellcheck'
                }
            }

            stage("Build ${imageToBuild}") {
                node {
                    infra.withDockerCredentials {
                        sh "make build-${imageToBuild}"
                    }
                }
            }

            stage("Test ${imageToBuild}") {
                node {
                    sh 'make prepare-test'
                    try {
                        infra.withDockerCredentials {
                            sh "make test-${imageToBuild}"
                        }
                    } catch (err) {
                        error("${err.toString()}")
                    } finally {
                        junit(allowEmptyResults: true, keepLongStdio: true, testResults: 'target/*.xml')
                    }
                }
            }
        }

        stage('Multiarch build') {
            node {
                deleteDir()
                checkout scm
                infra.withDockerCredentials {
                    sh '''
                        make docker-init
                        docker buildx bake --file docker-bake.hcl linux
                    '''
                }
            }
        }

        stage('Optional Publish') {
            node {
                if (env.PUBLISH == 'true') {
                    infra.withDockerCredentials {
                        sh 'make docker-init'
                        sh 'make publish || echo "Skipping publish locally"'
                    }
                }
            }
        }

    }
}
