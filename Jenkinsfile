pipeline {
    agent any
    environment {
        VAULT_ADDR = 'https://vault.mycompany.com:8200'
    }
    stages {
        stage('Fetch Secrets') {
            steps {
                sh 'vault kv get secret/dockerhub'
            }
        }
    }
}
