def secrets = [
    [
        path: 'secrets/creds/my-secret-text', 
        engineVersion: 2, 
        secretValues: [
            [envVar: 'SONARQUBE_TOKEN', vaultKey: 'secret']
        ]
    ]
]

def configuration = [
    vaultUrl: 'http://65.0.12.82:8200',
    vaultCredentialId: 'vault-geetha-token',
    engineVersion: 2
]
pipeline {
    agent any
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        SONARQUBE_TOKEN = ""
    }
    stages {
       stage('vault') {
         steps {
            withVault([configuration: configuration, vaultSecrets: secrets]) {
                echo "$SONARQUBE_TOKEN"
            }
         }
      }
        stage('Docker Bench Security') {
            steps {
                sh 'chmod +x docker-bench-security.sh'
                sh './docker-bench-security.sh'
            }
        }

       stage('SonarQube Analysis') {
          agent any
       steps {
       

        sh '/var/opt/sonar-scanner-4.7.0.2747-linux/bin/sonar-scanner  -Dsonar.projectKey=adservice-   -Dsonar.sources=. -Dsonar.java.binaries=path/to/compiled/classes  -Dsonar.exclusions=**/*.java  -Dsonar.host.url=http://172.31.7.193:9000 -Dsonar.token=$SONARQUBE_TOKEN'
      
        
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
