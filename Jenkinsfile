pipeline {
    agent any
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-hub')
        DOCKER = "${dockerTool}/bin"
    }
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    def dockerImage = docker.build('geethapeddinni/jenkins-docker-hub:latest', '.')
                }
            }
        }
        stage('Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Push') {
            steps {
                sh 'docker push geethapeddinni/jenkins-docker-hub'
            }
        }
        stage('Syft Scan') {
            steps {
                script {
                    try {
                        def syftOutput = sh(returnStatus: true, script: 'syft --json geethapeddinni/jenkins-docker-hub')
                        echo "Syft Output: ${syftOutput}"
                    } catch (Exception e) {
                        echo "Syft failed with an error: ${e}"
                        currentBuild.result = 'FAILURE'
                    }
                }
            }
        }
    } // This was missing in your script
    post {
        always {
            sh 'docker logout'
        }
    }
}
