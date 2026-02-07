pipeline {
    agent {
        docker {
            image 'alqoseemi/runner-node-docker:latest'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Branch to build')
    }

    stages {

        // -----------------------------
        stage('Checkout from GitHub') {
            environment {
                GITHUB_USER  = vault(path: 'secret/github', key: 'username')
                GITHUB_TOKEN = vault(path: 'secret/github', key: 'token')
            }
            steps {
                echo "Checking out branch ${params.BRANCH_NAME} from GitHub..."
                git url: "https://${GITHUB_USER}:${GITHUB_TOKEN}@github.com/myorg/node-app.git",
                    branch: "${params.BRANCH_NAME}"
            }
        }

        // ------------------------------
        stage('Build & Test') {
            steps {
                sh '''
                    echo "Installing dependencies..."
                    npm ci

                    echo "Running tests..."
                    npm test
                '''
            }
        }

        // ------------------------------
        stage('SonarQube Analysis') {
            environment {
                SONARQUBE_TOKEN = vault(path: 'secret/sonarqube', key: 'token')
                SONARQUBE_URL   = 'http://100.53.185.191:9000'
            }
            steps {
                withSonarQubeEnv('MySonarQubeServer') {
                    sh """
                        npx sonar-scanner \
                          -Dsonar.projectKey=node-app \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=${SONARQUBE_URL} \
                          -Dsonar.login=${SONARQUBE_TOKEN}
                    """
                }
            }
        }

        // ------------------------------
        stage('Docker Build & Push') {
            steps {
                withCredentials([
                    string(credentialsId: 'vault-dockerhub-roleid', variable: 'ROLE_ID'),
                    string(credentialsId: 'vault-dockerhub-secretid', variable: 'SECRET_ID')
                ]) {
                    sh '''
                        VAULT_ADDR="https://vault.example.com"

                        # Authenticate with Vault using AppRole
                        VAULT_TOKEN=$(curl -s --request POST \
                          --data "{\"role_id\":\"$ROLE_ID\", \"secret_id\":\"$SECRET_ID\"}" \
                          $VAULT_ADDR/v1/auth/approle/login | jq -r '.auth.client_token')

                        # Fetch DockerHub credentials from Vault
                        DOCKER_USER=$(curl -s --header "X-Vault-Token: $VAULT_TOKEN" \
                          $VAULT_ADDR/v1/secret/data/dockerhub | jq -r '.data.data.username')

                        DOCKER_PASS=$(curl -s --header "X-Vault-Token: $VAULT_TOKEN" \
                          $VAULT_ADDR/v1/secret/data/dockerhub | jq -r '.data.data.password')

                        # Login to DockerHub
                        docker login -u $DOCKER_USER -p $DOCKER_PASS

                        # Generate timestamp tag (YYYYMMDD-HHMMSS)
                        TIMESTAMP=$(date +%Y%m%d-%H%M%S)

                        # Include Jenkins build number in tag
                        IMAGE_NAME="myrepo/myimage"
                        IMAGE_TAG="${TIMESTAMP}-build${BUILD_NUMBER}"

                        # Build and push with timestamp + build number tag
                        docker build -t $IMAGE_NAME:$IMAGE_TAG .
                        docker push $IMAGE_NAME:$IMAGE_TAG

                        # Optionally still push 'latest'
                        docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest
                        docker push $IMAGE_NAME:latest
                    '''
                }
            }
        }
    }

    post {
        always {
            cleanWs()  // clean up workspace after each run
        }
    }
}
