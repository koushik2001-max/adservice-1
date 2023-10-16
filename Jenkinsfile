def secrets = [
    [
        path: 'secrets/metadata/creds/my-secret-text', 
        engineVersion: 2, 
        secretValues: [
            [envVar: 'secrets', vaultKey: 'secret']
        ]
    ]
]

def configuration = [vaultUrl: 'http://13.233.214.235:8200',  vaultCredentialId: 'vault-geetha-token', engineVersion: 2]


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
        stage("Test"){
    node(defaultNodeLabel()){
        println("TEST 1")
        withCredentials([[
            $class: 'VaultTokenCredentialBinding',
            credentialsId: 'vault-geetha-token'',
            vaultAddr: 'http://13.233.214.235:8200']]) {
            sh 'vault kv get -field=user v1/secrets/metadata/creds/my-secret-text'
        }

        def secrets = [
            [path: 'v1/secrets/metadata/creds/my-secret-text', secretValues: [
                [vaultKey: 'secret']]]
        ]
        withVault([vaultSecrets: secrets]) {
            println("TEST 2")
            sh 'echo ${env.secret}'
        }
    }
}

     stage('vaultt'){
           steps{
//          withVault(configuration: [timeout: 60, vaultCredentialId: 'vault-geetha-token' ,vaultUrl: 'http://13.233.214.235:8200'], vaultSecrets: [[path: 'secrets/metadata/creds/my-secret-text', secretValues: [[envVar: 'secrets', vaultKey: 'secret']]]]) { 
         withVault([configuration: configuration, vaultSecrets: secrets]) {
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
