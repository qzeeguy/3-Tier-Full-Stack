pipeline {
    agent any
    environment {
        VAULT_ADDR = 'http://44.222.232.182:8200'
    }
    stages {
        stage('Fetch Secrets') {
            steps {
                sh 'vault kv get secret/dockerhub'
            }
        }
    }
}
