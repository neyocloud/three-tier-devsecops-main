pipeline {
    agent any

    environment {
        // Docker image name in Docker Hub
        DOCKER_IMAGE = 'neyocicd/three-tier-devsecops-main'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh '''
                  echo ">>> Build stage (placeholder)"
                  echo "Current workspace content:"
                  ls -R
                '''
            }
        }

        stage('SonarQube Scan') {
            steps {
                // "sonar" must match the SonarQube server name in Manage Jenkins → System
                withSonarQubeEnv('sonar') {
                    sh '''
                      echo ">>> Running SonarQube scan with server: $SONAR_HOST_URL"

                      /opt/sonar-scanner/bin/sonar-scanner \
                        -Dsonar.projectKey=three-tier-devsecops-main \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.login=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                // "docker-cred" = DockerHub PAT (username+password) credential
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-cred',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                      echo ">>> Logging in to Docker Hub as $DOCKER_USER"
                      echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                      echo ">>> Building image $DOCKER_IMAGE:latest"
                      docker build -t $DOCKER_IMAGE:latest -f app-code/backend/Dockerfile app-code/backend

                      echo ">>> Pushing image $DOCKER_IMAGE:latest"
                      docker push $DOCKER_IMAGE:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // "k8-cred" = Secret file credential containing your kind kubeconfig
                withCredentials([
                    file(credentialsId: 'k8-cred', variable: 'KUBECONFIG')
                ]) {
                    sh '''
                      echo ">>> Using kubeconfig at: $KUBECONFIG"

                      echo ">>> Cluster nodes:"
                      kubectl --kubeconfig=$KUBECONFIG get nodes

                      echo ">>> Applying Kubernetes manifests from kubernetes-manifests/ (recursive)"
                      kubectl --kubeconfig=$KUBECONFIG apply -R -f kubernetes-manifests/

                      echo ">>> Pods in all namespaces:"
                      kubectl --kubeconfig=$KUBECONFIG get pods -A
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline succeeded – all stages passed.'
        }
        failure {
            echo '❌ Pipeline failed – check the stage logs above.'
        }
    }
}

