pipeline {
    agent any

    environment {
        // ==== SonarQube ====
        // Project key must exist in SonarQube
        SONAR_PROJECT_KEY = 'three-tier-devsecops-main'

        // ==== Docker Hub ====
        DOCKER_IMAGE = 'neyocicd/three-tier-devsecops-main'
        IMAGE_TAG    = 'latest'

        // ==== Kubernetes ====
        K8S_NAMESPACE = 'default'
    }

    options {
        // we do our own git checkout
        skipDefaultCheckout(true)
    }

    stages {

        // ---------- CHECKOUT ----------
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/neyocloud/three-tier-devsecops-main.git',
                    credentialsId: 'git-cred'
            }
        }

        // ---------- BUILD (placeholder) ----------
        stage('Build') {
            steps {
                sh '''
                  echo ">>> Build stage (customize this for your app: npm, pip, etc.)"
                  ls -R
                '''
            }
        }

        // ---------- SONARQUBE SCAN ----------
        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('sonar') {
                    script {
                        // use Jenkins-managed SonarScanner installation
                        def scannerHome = tool 'sonar-scanner'
                        sh """
                          echo ">>> Running SonarQube scan with server: $SONAR_HOST_URL"
                          "${scannerHome}/bin/sonar-scanner" \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY}
                        """
                    }
                }
            }
        }

        // ---------- DOCKER BUILD & PUSH ----------
        stage('Docker Build & Push') {
            steps {
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

                      echo ">>> Building image $DOCKER_IMAGE:$IMAGE_TAG from backend Dockerfile"
                      docker build -t $DOCKER_IMAGE:$IMAGE_TAG \
                        -f app-code/backend/Dockerfile \
                        app-code/backend

                      echo ">>> Pushing image $DOCKER_IMAGE:$IMAGE_TAG"
                      docker push $DOCKER_IMAGE:$IMAGE_TAG
                    '''
                }
            }
        }

        // ---------- DEPLOY TO KUBERNETES ----------
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(credentialsId: 'k8-cred', variable: 'KUBECONFIG')
                ]) {
                    sh '''
                      echo ">>> Using kubeconfig at: $KUBECONFIG"

                      echo ">>> Cluster nodes:"
                      kubectl --kubeconfig=$KUBECONFIG get nodes

                      echo ">>> Applying Kubernetes manifests from kubernetes-manifests/"
                      kubectl --kubeconfig=$KUBECONFIG apply -f kubernetes-manifests/

                      echo ">>> Pods in all namespaces:"
                      kubectl --kubeconfig=$KUBECONFIG get pods -A
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed – check stage logs."
        }
    }
}

