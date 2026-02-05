pipeline {
    agent any

    parameters {
        string(name: "BRANCH_NAME", defaultValue: "file/dev", description: "Specify the branch name to deploy")
    }

    stages {
        stage('Checkout') {
            environment {
                GITHUB_USER  = vault(path: 'secret/github', key: 'username')
                GITHUB_TOKEN = vault(path: 'secret/github', key: 'token')
            }
            steps {
                git url: "https://${GITHUB_USER}:${GITHUB_TOKEN}@github.com/myorg/node-app.git", branch: 'main'
            }
        }

        stage('Test') {
            steps {
                sh 'npm ci'
                sh 'npm test'
            }
        }

        stage('Build') {
            when {
                expression {
                    def pkgChanged = sh(script: 'git diff --name-only HEAD~1 HEAD | grep package.json || true', returnStdout: true).trim()
                    def srcChanged = sh(script: 'git diff --name-only HEAD~1 HEAD | grep -E "\\.js$|\\.ts$" || true', returnStdout: true).trim()
                    return (pkgChanged || srcChanged)
                }
            }
            steps {
                echo "Building project since changes detected..."
                sh 'npm run build'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONARQUBE_TOKEN = vault(path: 'secret/sonarqube', key: 'token')
                SONARQUBE_URL   = 'http://3.238.111.36:9000'
            }
            steps {
                withSonarQubeEnv('MySonarQubeServer') {
                    // Use single quotes for shell so Groovy doesn't interpolate
                    sh 'npx son
