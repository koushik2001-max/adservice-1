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

        stage('SonarQube Analysis') {
          agent any
      steps {
       

        sh '/var/opt/sonar-scanner-4.7.0.2747-linux/bin/sonar-scanner  -Dsonar.projectKey=checkout-service   -Dsonar.sources=.   -Dsonar.host.url=http://172.31.7.193:9000   -Dsonar.token=sqp_eecb5e93052d8e7c279f21774114833d87c98709'
      
        
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
