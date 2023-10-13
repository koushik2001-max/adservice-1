pipeline {
    agent any
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        SNYK_HOME = tool name: 'Snyk'
    }
    stages {
        stage('Docker Bench Security') {
            steps {
                sh 'chmod +x docker-bench-security.sh'
                sh './docker-bench-security.sh'
            }
        }
        stage('Snyk Code') {
            steps {
                sh "${SNYK_HOME}/snyk-linux code test"
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool name: 'SonarScanner'
                    def sonarHostUrl = 'http://172.31.7.193:9000'
                    def sonarToken = 'sqp_3ec0d083de10a3b34456bf69cab2f03c25d576c7'
                    sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=checkout-service -Dsonar.sources=. -Dsonar.host.url=${sonarHostUrl} -Dsonar.login=${sonarToken}"
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def dockerImage = docker.build('koushiksai/jenkins-docker-hub:latest', '.')
                }
            }
        }
        stage('Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKERHUB_CREDENTIALS_USR', passwordVariable: 'DOCKERHUB_CREDENTIALS_PSW')]) {
                    sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
                }
            }
        }
        stage('Push') {
            steps {
                sh 'docker push koushiksai/jenkins-docker-hub'
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}
