pipeline {
    agent any

    environment {
        // SonarQube
        SONAR_PROJECT_KEY = 'three-tier-devsecops-main'

        // Docker image to build & push
        DOCKER_IMAGE = 'neyocicd/three-tier-devsecops-main'
    }

    stages {

        stage('Checkout') {
            steps {
                // Uses the Git settings you configured in "Pipeline script from SCM"
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh '''
                  echo ">>> Build stage (you can customize this later)"
                  ls -R
                '''
            }
        }

        stage('SonarQube Scan') {
            steps {
                // "sonar" must match the name of the SonarQube server in Manage Jenkins → Configure System
                withSonarQubeEnv('sonar') {
                    // sonar-token is a Secret text credential with your Sonar token
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        script {
                            // "sonar-scanner" must match the Tool name you configured in Jenkins (SonarScanner installation)
                            def scannerHome = tool 'sonar-scanner'
                            sh """
                              echo ">>> Running SonarQube scan with server: \$SONAR_HOST_URL"
                              "\${scannerHome}/bin/sonar-scanner" \
                                -Dsonar.projectKey=${env.SONAR_PROJECT_KEY} \
                                -Dsonar.sources=. \
                                -Dsonar.host.url=\$SONAR_HOST_URL \
                                -Dsonar.login=\$SONAR_TOKEN
                            """
                        }
                    }
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                // docker-cred is "Username with password" (Docker Hub user + PAT)
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
                      # Backend Dockerfile lives in app-code/backend
                      docker build -t $DOCKER_IMAGE:latest -f app-code/backend/Dockerfile app-code/backend

                      echo ">>> Pushing image $DOCKER_IMAGE:latest"
                      docker push $DOCKER_IMAGE:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // k8-cred is a "Secret file" credential with your kind kubeconfig
                withCredentials([
                    file(credentialsId: 'k8-cred', variable: 'KUBECONFIG')
                ]) {
                    sh '''
                      echo ">>> Using kubeconfig at: $KUBECONFIG"

                      echo ">>> Cluster nodes:"
                      kubectl --kubeconfig=$KUBECONFIG get nodes

                      echo ">>> Applying Kubernetes manifests from kubernetes-manifests/ (recursively)"
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
            echo '✅ Pipeline completed successfully.'
        }
        failure {
            echo '❌ Pipeline failed – check stage logs.'
        }
    }
}

