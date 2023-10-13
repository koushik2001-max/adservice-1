pipeline {
    agent any
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
    }
    stages {
        stage('Docker Bench Security') {
            steps {
                sh 'chmod +x docker-bench-security.sh'
                sh './docker-bench-security.sh'
            }
        }
stage('snyk checking') {

      steps {

        echo 'snyk testing...'

        snykSecurity(

          snykInstallation: "snyk@latest",

          snykTokenId: "organisation-snyk-api-token",

          // place other parameters here

        )

      }

    }
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool name: 'SonarScanner'
                    def sonarHostUrl = 'http://172.31.7.193:9000'
                    def sonarToken = 'sqp_eecb5e93052d8e7c279f21774114833d87c98709'
                    sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=checkout-service -Dsonar.sources=. -Dsonar.host.url=${sonarHostUrl} -Dsonar.login=${sonarToken}"
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def dockerImage = docker.build('koushiksai/adservice:latest', '.')
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
                sh 'docker push koushiksai/adservice'
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}
