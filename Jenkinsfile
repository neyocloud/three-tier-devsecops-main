pipeline {
    agent any

    environment {
        // SonarQube
        SONAR_HOST_URL     = 'http://sonar:9000'

        // Docker
        DOCKER_REGISTRY    = 'docker.io'
        DOCKER_IMAGE_NAME  = 'neyocicd/three-tier-devsecops-main'  // change if your image name is different

        // Kubernetes
        K8S_NAMESPACE      = 'default'
    }

    stages {
        stage('Checkout') {
            steps {
                // Uses the job's SCM config (GitHub with git-cred)
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh '''
                  echo ">>> Build stage (customize for your app: tests, packaging, etc.)"
                  ls -R
                '''
            }
        }

        stage('SonarQube Scan') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    script {
                        // Get path to the SonarQube Scanner tool configured in Jenkins
                        def SCANNER_HOME = tool 'sonar-scanner'

                        sh """
                            ${SCANNER_HOME}/bin/sonar-scanner \
                              -Dsonar.projectKey=three-tier-devsecops-main \
                              -Dsonar.host.url=${SONAR_HOST_URL} \
                              -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Docker Build & Push') {
            when {
                expression { currentBuild.currentResult == null || currentBuild.currentResult == 'SUCCESS' }
            }
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-cred',        // Docker Hub PAT
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    // Use single-quoted Groovy string so secrets aren't interpolated by Groovy
                    sh '''
                        echo ">>> Logging in to Docker Hub as $DOCKER_USER"
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin $DOCKER_REGISTRY

                        echo ">>> Building image $DOCKER_IMAGE_NAME:latest"
                        docker build -t $DOCKER_IMAGE_NAME:latest .

                        echo ">>> Tagging image with build number: $BUILD_NUMBER"
                        docker tag $DOCKER_IMAGE_NAME:latest $DOCKER_IMAGE_NAME:$BUILD_NUMBER

                        echo ">>> Pushing images"
                        docker push $DOCKER_IMAGE_NAME:latest
                        docker push $DOCKER_IMAGE_NAME:$BUILD_NUMBER

                        echo ">>> Docker logout"
                        docker logout $DOCKER_REGISTRY || true
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                expression { currentBuild.currentResult == null || currentBuild.currentResult == 'SUCCESS' }
            }
            steps {
                withCredentials([
                    file(credentialsId: 'k8-cred', variable: 'KUBECONFIG')   // kubeconfig file
                ]) {
                    sh """
                        echo ">>> Using kubeconfig at: ${KUBECONFIG}"
                        kubectl --kubeconfig=${KUBECONFIG} config set-context --current --namespace=${K8S_NAMESPACE}
                        echo ">>> Applying Kubernetes manifests from kubernetes-manifests/"
                        kubectl --kubeconfig=${KUBECONFIG} apply -f kubernetes-manifests/
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline succeeded – all stages completed."
        }
        failure {
            echo "❌ Pipeline failed – check stage logs."
        }
        always {
            echo "🏁 Pipeline finished with status: ${currentBuild.currentResult}"
        }
    }
}

