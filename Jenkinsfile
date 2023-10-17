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

     stage('vaultt'){
           steps{
         withVault(configuration: [timeout: 60, vaultCredentialId: 'vault-geetha-token' ,vaultUrl: 'http://13.232.255.50:8200'], vaultSecrets: [[path: 'secret/data/creds/my-secret-text', secretValues: [[envVar: 'secrets', vaultKey: 'secret']]]]) { 
//         withVault([configuration: configuration, vaultSecrets: secrets]) {
        sh "echo ${env.secrets}"

        }

      }
      
    }
      
       stage('SonarQube Analysis') {
          agent any
       steps {
       

        sh '/var/opt/sonar-scanner-4.7.0.2747-linux/bin/sonar-scanner  -Dsonar.projectKey=adservice-   -Dsonar.sources=. -Dsonar.java.binaries=path/to/compiled/classes  -Dsonar.exclusions=**/*.java  -Dsonar.host.url=http://172.31.7.193:9000 -Dsonar.token=sqp_391fc52230ca8a7ea4fd28686ac079f514c89dd2'
      
        
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
