

# Three-Tier DevSecOps Full-Stack Web Application Deployment  
 
 ### Local Deployment using Docker • Jenkins • SonarQube • Prometheus • Grafana



## 📘 Table of Contents
 
 [Overview](#overview)
 
 [Architecture](#architecture)
 
 [Tools Used](#tools-used)
 
 [Prerequisites](#prerequisites--tested-versions)
 
 [Setup Instructions](#setup-instructions)
 
 [Jenkins Credentials](#-jenkins-credentials)
 
 [CI/CD Pipeline](#pipeline-definition-jenkinsfile)
 
 [Running Locally (Docker)](#3-running-the-application-and-monitoring-containers)
 
 [Prometheus & Grafana Setup](#4-configuring-prometheus-scrape-targets)
 
 [Troubleshooting](#common-errors-and-troubleshooting)
 
 [Security Notes](#security-minimal)
 
 [Verify Services](#verify)

















### Overview


This project provides a complete DevSecOps workflow for deploying a **three-tier web application** (frontend, backend, database) locally using Docker, fully automated through a **Jenkins CI/CD pipeline**.


It includes:

•✔ CI/CD 

• ✔ Containerization 

• ✔ Code Scanning 

• ✔ Monitoring 

• ✔ Automated Deployment


## Architecture



<img width="2238" height="2048" alt="image" src="https://github.com/user-attachments/assets/206dad65-b667-4a93-8a08-fad6d37c57ac" />



Three-Tier DevSecOps Full-Stack Web Application Deployment (Local with Docker, Prometheus, Grafana, and Jenkins): 

This application is designed using a **clean three-tier architecture**, each tier running in its own container:

The tiers are

Frontend: user interface (e.g. React SPA), served through a web server (e.g. Nginx). 
Backend: API server (e.g. Node.js/Express app) which is the business logic layer and communicates with the frontend (through HTTP API calls) and the database. 
Database: the database server (e.g. MongoDB), which holds the application's data. Containerized Deployment: Each tier is packaged as a docker image. In a local deployment, each one of these layers is executed (e.g. via Docker Compose, or docker run,) and the layers are networked together. 

The frontend layer is making HTTP calls to the backend layer, while the backend is able to access the db via the network alias of the database container. CI/CD Pipeline is controlled by a Jenkins server.


Monitoring and Logging: The deployment includes a monitoring stack:

Prometheus: Collects metrics from various targets (e.g. the host machine via Node Exporter, or app-specific metrics if exposed).

Node Exporter: Exposes host machine metrics (CPU, memory, etc.) to Prometheus.

Grafana: Connects to Prometheus to visualize metrics on dashboards. This provides insight into the health and performance of the application and infrastructure in real time.

Overall, this architecture ensures the application is delivered in a repeatable, automated way (thanks to Jenkins and Docker), and that it’s observable via metrics (thanks to Prometheus and Grafana).


## 📁 Project Structure (File Map)


```
three-tier-devsecops-main/
│── app-code/
│   ├── frontend/
│   └── backend/
│── kubernetes-manifests/
│── monitoring/prometheus.yml
│── Jenkinsfile
│── docker-compose.yml   (added for simplified local deploy)
```



## Tools Used and Why




<img width="1400" height="900" alt="image" src="https://github.com/user-attachments/assets/d1c0d7fc-7d3a-4a9a-a774-86c6afbb3e23" />









**Jenkins**: Used as the Continuous Integration/Continuous Deployment tool. Jenkins Pipeline (defined in a Jenkinsfile) automates building the code, running tests, performing security scans, and deploying the application. Jenkins orchestrates the DevSecOps workflow end-to-end, from pulling code to pushing containers and deploying updates.

**Docker**: Used for containerizing all parts of the application and infrastructure tools. Docker ensures environment consistency across deployments. We containerize the frontend, backend, and database, as well as Jenkins itself (running Jenkins in a Docker container), Prometheus, Grafana, etc. Docker Compose or Docker networks manage the multi-container setup and networking.

**Prometheus**: Used for monitoring and metrics collection. Prometheus (running as a container) scrapes metrics from targets like Node Exporter (for host metrics) and possibly the application (if the app exposes a /metrics endpoint). Prometheus stores these time-series metrics and allows querying them via PromQL. This provides data on system performance (CPU usage, memory usage, etc.) and application behavior.

**Grafana**: Used for metrics visualization and dashboards. Grafana (containerized) connects to Prometheus as a data source and displays the collected metrics on configurable dashboards. We use Grafana to visualize key metrics (e.g. server CPU load, memory usage, container status) to quickly identify issues. Grafana also supports setting up alerts on certain conditions (though in a local setup, this is mainly for demonstration).

**SonarQube (Scanner)**: Used for static code analysis and security scanning in the CI pipeline. In our Jenkins pipeline, there is a SonarQube scan stage that analyzes the code for bugs and vulnerabilities. We utilize the SonarQube Scanner CLI to run the analysis. (In a full setup, a SonarQube server would be running as a separate service or container – the pipeline sends analysis data to that server. Here we assume a SonarQube server is accessible at http://sonar:9000 as configured in the Jenkinsfile.)



<img width="3024" height="1964" alt="image" src="https://github.com/user-attachments/assets/55e79752-d376-48dd-9ce0-2f48a0daf068" />





Additionally, other DevSecOps tools like Trivy (container vulnerability scanning) or Nexus (artifact repository) could be integrated, but the core focus here is on Jenkins, Docker, Prometheus, and Grafana. The SonarQube stage already covers code quality/security scanning in this project.)





**End-to-End DevSecOps Workflow:** Incorporate security scans, monitoring, and scalable infrastructure for a production-ready deployment.





To make learning even easier, This is a detailed guide, walking you through each step visually. Whether you're a DevOps enthusiast or a seasoned cloud professional, this tutorial will help you bridge the gap between theory and practical implementation.




## Setup Instructions


1. Cloning the Repository

Begin by obtaining the project source code from the repository. Clone the Git repository to your local machine using Git:


```bash

git clone https://github.com/neycloud/three-tier-devsecops-main.git

```

```
cd three-tier-devsecops-main
 ```

This repository contains all the necessary code and configuration:



Application source code for the frontend and backend (e.g. in an app-code/ directory).



Dockerfiles for each component (to build container images for the frontend, backend, etc.).



Jenkins pipeline definition (Jenkinsfile) and any Jenkins-specific scripts (possibly in a jenkins-pipeline/ directory).



Infrastructure as code or deployment manifests (e.g. Kubernetes manifests in kubernetes-manifests/ for deploying to a cluster).



Configuration for monitoring (e.g. a Prometheus config file, Docker Compose files if provided for Prometheus/Grafana/Node Exporter).



Make sure you have Docker installed on your system before proceeding (and Docker Compose if you plan to use it). It’s also helpful to have Git and (if running Jenkins outside of Docker) a recent JDK installed, but in our case we will run Jenkins in a Docker container.






## Prerequisites & tested versions

Docker Engine >= 20.10
Docker Compose v2.x (docker compose)
Git >= 2.25
Node.js >= 16 (for building frontend/backend)
npm >= 8
Java 11+ (for Sonar Scanner if running in Jenkins)
Jenkins LTS (run via image jenkins/jenkins:lts)
kubectl (if deploying to Kubernetes) ARM / Apple Silicon (M1/M2) notes
Prefer arm64 images when available. For images that do not provide arm64, use --platform linux/amd64 (may fail on pure arm hosts).
Example: docker run --platform linux/amd64 ...
For Jenkins Sonar Scanner on arm64, install the aarch64 scanner or run scanner in an agent that matches architecture.





## 2. Setting Up Jenkins for CI/CD

Next, set up Jenkins, which will run our CI/CD pipeline.

Launch Jenkins: The easiest way is via Docker.

```
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```

This starts Jenkins in a container, accessible on port 8080 (web UI) and 50000 for agent connections. We use a named volume jenkins_home to persist Jenkins data (so configuration isn’t lost if the container restarts).

Initial Setup: Once Jenkins is running, access the web UI at http://localhost:8080. Jenkins will ask for an initial admin password. You can retrieve this from the container log or from the file on the container. For example, run:


```
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

This will print the one-time password (a long alphanumeric string). Copy that into the Jenkins UI to unlock it, then proceed with the setup. Install the recommended plugins when prompted.

Install Required Plugins and SonarQube Scanner: Ensure the following Jenkins plugins are installed for our pipeline:

Pipeline and Pipeline: Stage View for Jenkins Pipeline jobs.

Git (for pulling from GitHub) – and optionally GitHub integration plugins if using webhooks.

Docker Pipeline (if the pipeline will run Docker commands or use Docker agents).

Blue Ocean (optional, for a nicer pipeline visualization UI).

SonarQube Scanner for Jenkins just to integrate SonarQube scans in the pipeline. After installing this, go to Manage Jenkins > Configure System and add a SonarQube server configuration (URL and authentication token). Name this server e.g. “SonarQubeServer” for use in the pipeline. You’ll also need to add a Jenkins credential for the Sonar token the pipeline uses credentials('sonar-token') which should match the ID of a secret text credential containing your SonarQube token.

In addition to plugins, the pipeline will use the SonarQube Scanner CLI to perform code analysis. On Jenkins x86_64, the plugin can automatically download the scanner. However, if your Jenkins is running on an ARM64 system (e.g. Apple M1/M2 Mac), you may need to manually install an ARM-compatible scanner. 

Below are the steps to install the SonarQube Scanner CLI inside the Jenkins container (for ARM64):

Open a shell in the running Jenkins container as root: 

```
docker exec -u 0 -it jenkins bash
```

(The -u 0 flag runs the shell as root user, which is needed to install software in the container.)

Inside the container, install necessary tools and download the ARM64 scanner:

```
apt-get update && apt-get install -y curl unzip
cd /opt
curl -LO https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-7.3.0.5189-linux-aarch64.zip
unzip sonar-scanner-cli-7.3.0.5189-linux-aarch64.zip
```


The zip will extract a directory like sonar-scanner-7.3.0.5189-linux-aarch64. Rename it for convenience and set proper permissions:

```
mv sonar-scanner-7.3.0.5189-linux-aarch64 sonar-scanner
chmod -R a+rx /opt/sonar-scanner
```



The chmod -R a+rx command makes all files in the scanner directory readable and executable by all users. This is important because Jenkins runs as a non-root user inside the container; without adjusting permissions, Jenkins might not be able to execute the scanner. By giving all users read/execute rights, we ensure the Jenkins user can run the sonar-scanner binaries.

(Optional) Add a symlink to make the scanner easily accessible from the PATH:

```
ln -sf /opt/sonar-scanner/bin/sonar-scanner /usr/local/bin/sonar-scanner
```
This lets you simply call sonar-scanner from anywhere in the container.






Exit the container shell (exit) and, for good measure, test that Jenkins can use the scanner. For example, you can trigger a build with a Sonar stage (as we’ll set up in the Jenkinsfile) or exec into the container as the Jenkins user to run /opt/sonar-scanner/bin/sonar-scanner -v. You should see the scanner’s version output, confirming it’s installed correctly.






## Credentials Setup: In Jenkins, add any necessary credentials that the pipeline will use:



<img width="3024" height="1964" alt="image" src="https://github.com/user-attachments/assets/f630cf94-294b-4076-816f-4cf64da2ab10" />


**Git Credentials**: If your Git repository is private, add credentials (e.g. SSH key or username/password) so Jenkins can pull the code.

**Docker Registry Credentials**: If the pipeline pushes images to Docker Hub (or another registry), configure credentials (username/password or token) in Jenkins and use with docker.withRegistry or via environment variables.

**SonarQube Token**: As mentioned, add a Secret Text credential for SonarQube authentication (the token from your SonarQube server). Ensure the credential ID matches what the Jenkinsfile expects (sonar-token in our case).

**Kubernetes Credentials**: If deploying to a Kubernetes cluster that requires authentication (for local clusters like Kind or Docker Desktop’s Kubernetes, Jenkins might be able to use the default kubeconfig), you may need to provide a kubeconfig file or token. In a simple local setup, Jenkins can just call kubectl assuming the kubeconfig is mounted or Jenkins container is configured to use the host’s kubeconfig.





### Jenkins credentials


Add the following credentials in Jenkins (Manage Jenkins → Credentials → System → Global credentials) so the pipeline can run:

### 🔐 Jenkins Credentials

| ID             | Kind                    | Description                                    | Used in Jenkinsfile                           |
|----------------|-------------------------|------------------------------------------------|-----------------------------------------------|
| 🟦 sonar-token | Secret text             | SonarQube authentication token                 | SonarQube Scan (`withSonarQubeEnv`)           |
| 🟩 git-cred    | Secret text             | GitHub Personal Access Token (PAT)             | Checkout / Git pull from private repos        |
| 🐳 docker-cred | Username with password  | Docker Hub credentials (username + PAT/token)  | Docker Build & Push                           |
| ☸️ k8-cred     | Secret file             | `kubeconfig` for local Kubernetes / kind       | Deploy to Kubernetes                          |


## Notes:

- Do NOT commit tokens, secrets, or kubeconfig files to the repo. Use Jenkins credentials only.
- Prefer least-privilege tokens and restrict credential scope (folder-level) where possible.
- If any token or kubeconfig was exposed publicly, rotate it immediately and treat it as compromised.
- Example: The Jenkinsfile expects these IDs (sonar-token, git-cred, docker-cred, k8-cred); ensure they match.

**Double-check that each ID in the table matches the IDs used in the Jenkinsfile**



Create the Pipeline Job: In Jenkins, create a new Pipeline job for this project (e.g. name it “three-tier-devsecops-pipeline”). Configure it to pull the Jenkinsfile from the repo:

<img width="1512" height="982" alt="image" src="https://github.com/user-attachments/assets/dfca31cd-73ee-4d7d-9061-7bd8cca8a475" />


In the job config, under Pipeline, select “Pipeline script from SCM”.

Choose Git as the SCM, and enter the repository URL (the one you cloned).



<img width="1512" height="982" alt="image" src="https://github.com/user-attachments/assets/1a358192-aebe-4859-9960-9567ac79988a" />




Set the branch to the appropriate branch (e.g. main).




<img width="1512" height="982" alt="image" src="https://github.com/user-attachments/assets/00b364c3-8aed-46ec-8410-3de456e572b5" />




Set the Script Path to the location of the Jenkinsfile (Jenkinsfile if it’s at the repo root).

Save the job.

Jenkins Pipeline Definition (Jenkinsfile): The Jenkinsfile in this project defines the stages of our CI/CD pipeline. It contains environment variables and sequential stages for build, scan, and deployment. Below is a simplified version of the Jenkinsfile for reference:



```
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


```



**The Jenkinsfile above outlines the key stages. The actual commands might differ based on your project structure (for example, you might build frontend and backend separately, or run tests, etc.), but this gives an idea of the pipeline flow. The SonarQube stage uses the scanner we installed to perform code analysis, and the Docker stage builds the app’s Docker image and pushes it to Docker Hub. Finally, the deploy stage applies Kubernetes manifests to run the new version of the app.*



**Run the Pipeline**: Once the Jenkins job is configured, trigger a build (click “Build Now” in Jenkins). Jenkins will pull the code and execute the pipeline stages in order. Monitor the console output in Jenkins to follow along each stage. If everything is set up correctly, all stages should succeed. You’ll have a built application, a SonarQube scan report (viewable in your SonarQube server if configured), a Docker image pushed to your registry, and the application deployed to your local Kubernetes (or local Docker environment).


<img width="1450" height="646" alt="image" src="https://github.com/user-attachments/assets/e4432a4c-127d-4eb3-9b23-d3133f4d3d8f" />




If the build fails, Jenkins will indicate which stage had an error. Common issues could be misconfigured credentials, failing tests, or Docker build errors. Check the logs for details in any failing stage. Once issues are fixed, you can run the pipeline again. After a successful run, you should see all stages marked with a green check in Jenkins. For example, a successful pipeline might show stages: Checkout → Build → SonarQube Scan → Docker Build & Push → Deploy, all completed. .......


<img width="3024" height="1964" alt="image" src="https://github.com/user-attachments/assets/bd498274-5aad-4c10-856e-db68a4fd3d55" />



In a real project, you might set this pipeline to trigger on each code push via webhooks. For now, manual triggering is fine for testing.







## 3. Running the Application and Monitoring Containers






With the pipeline run completed (or as an alternative to using Jenkins deployment), you can now ensure the application and monitoring stack are running locally using Docker. We will start the frontend, backend, database, Prometheus, Node Exporter, and Grafana containers. If you used the Jenkins pipeline with Kubernetes deploy, your app might already be running in a local cluster; but here we’ll describe how to run everything with plain Docker for simplicity.

Create a Docker Network: To allow all containers to communicate easily by name, create a user-defined network (so Docker will handle DNS for containers on it):




```
docker network create monitoring
```

We’ll use the name “monitoring” for the network (you can choose another name).





## Launch the Database (MongoDB):


```
docker run -d --name mongodb --network monitoring -p 27017:27017 mongo:latest
```

This runs a MongoDB container in detached mode. It attaches to our monitoring network and exposes port 27017 to the host so you could connect to the DB at localhost:27017 if needed. The backend service will use the hostname mongodb (the container name) to connect on the internal Docker network.




## Launch the Backend:

```
docker run -d --name backend --network monitoring -p 5000:5000 \
  -e DB_HOST=mongodb -e DB_PORT=27017 \
  neyocicd/three-tier-devsecops-main:latest
```



Replace the image name (neyocicd/three-tier-devsecops-main:latest) with the actual backend image if it’s different. We pass environment variables to tell the backend how to reach the database (here, DB_HOST=mongodb so it resolves the MongoDB container by name, and the default MongoDB port). This publishes the backend’s API on port 5000 of the host (so you can reach the API at http://localhost:5000).

## Launch the Frontend:

```
docker run -d --name frontend --network monitoring -p 80:80 \
  neyocicd/frontend-image:latest
```


Replace neyocicd/frontend-image:latest with your actual frontend image name/tag if it differs. We map the container’s port 80 to host port 80, so the web UI will be accessible at http://localhost. (If your frontend container listens on a different port internally, adjust the ports accordingly. For example, if it’s a development server on 3000, use -p 80:3000 to map it to host port 80).

**At this point, the three-tier application containers (frontend, backend, database) should be up and connected via the monitoring network:*

The backend can resolve mongodb and connect to the database.

The frontend can make API requests to the backend. (If the frontend needs to know how to reach the backend, ensure it’s configured to use the correct base URL. Since we exposed backend on localhost:5000, the frontend (running in a container) might need to call the backend at http://backend:5000 if it’s networked, or you might configure it to use http://localhost:5000 for simplicity if you’re accessing the frontend via your browser on the host. This depends on how the frontend is implemented; for a containerized frontend, using the container name backend is a common approach.)





## Launch Prometheus and Node Exporter (Monitoring):

First, prepare a Prometheus configuration file on your host (call it prometheus.yml). This config will tell Prometheus what targets to scrape. For example:




```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']    # Prometheus scrapes itself

  - job_name: nodeexporter
    static_configs:
      - targets: ['nodeexporter:9100'] # Scrape Node Exporter container

```


This config has Prometheus scrape itself (so we get metrics on Prometheus’ performance) and scrape the Node Exporter on port 9100. Make sure the job_name and target hostnames match what we will run (we will name the Node Exporter container nodeexporter).



Now run the Prometheus container:

```
docker run -d --name prometheus --network monitoring -p 9090:9090 \
  -v $PWD/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus:latest
```




<img width="1512" height="982" alt="image" src="https://github.com/user-attachments/assets/b58cb076-4fe2-4dc0-aae6-7453f74027a5" />




We attach Prometheus to the same monitoring network and mount our config file into the container. Prometheus UI will be available on http://localhost:9090.

Run the Node Exporter container:


```
docker run -d --name nodeexporter --network monitoring -p 9100:9100 \
  prom/node-exporter:latest
```


Node Exporter exposes host metrics on port 9100. We attached it to monitoring network so that Prometheus can resolve the hostname nodeexporter as configured. (Note: Node Exporter by default looks at the filesystem to read system metrics. In this simple setup, it will be reporting on the container’s view. For a more accurate host monitoring on Linux, you could run Node Exporter with --net=host and appropriate mounts, but that’s beyond scope here.)

**Run the Grafana container**:

```
docker run -d --name grafana --network monitoring -p 3000:3000 grafana/grafana:latest
```


Grafana will be available at http://localhost:3000 (default login is admin/admin, which you’ll change on first login). By connecting Grafana to Prometheus (next step), we’ll be able to visualize the metrics.



<img width="1512" height="982" alt="image" src="https://github.com/user-attachments/assets/b06359e5-892c-4116-9141-d426c5a110fc" />




After executing the above, all components of the stack should be running as Docker containers. You can verify by running docker ps (you should see containers for frontend, backend, mongodb, prometheus, nodeexporter, grafana, and jenkins). All services are now up and networked together. For instance, Prometheus and Grafana are on the monitoring network along with Node Exporter and our app, which means Prometheus can reach Node Exporter by name, and Grafana can reach Prometheus by name.

(If you orchestrated the deployment via Kubernetes, the pods would be running in your cluster instead. In that case, Prometheus and Grafana could also be set up in the cluster. This guide uses Docker for simplicity, but the monitoring principles apply the same way in Kubernetes.)

## 4. Configuring Prometheus Scrape Targets

Prometheus needs to know which endpoints to scrape for metrics. We already provided a basic prometheus.yml configuration with two scrape targets: Prometheus itself and the Node Exporter. Once Prometheus is running, you can verify it has picked up the targets by checking its interface:

Open the Prometheus UI by visiting http://localhost:9090 in your browser.

In the top menu, go to Status > Targets. You should see the configured targets:

One for prometheus (targeting localhost:9090) which should show as UP.

One for nodeexporter (targeting nodeexporter:9100) which should also be UP if Prometheus can reach the Node Exporter container.

You should see both targets listed and their state as UP. For example, Prometheus will report the last scrape time and any scrape errors here. If a target is misconfigured or unreachable, it would show as DOWN.

To further test, you can use Prometheus’s Graph feature to query some metrics:

Click Graph, then in the query box type up and hit Execute. This will show a line for each target with value 1 if it’s up (you should see a result for “prometheus” and “nodeexporter” jobs).

Try querying something like node_memory_MemAvailable_bytes (if using Node Exporter) to see the current available memory on your system as reported by Node Exporter.

Prometheus Targets Status: After configuration, the Targets page in Prometheus will list our two jobs. For example, it might show:


<img width="3024" height="1964" alt="image" src="https://github.com/user-attachments/assets/67d4d57c-e214-4a3e-bb3b-185f9e827fb8" />



job: prometheus endpoint: http://localhost:9090/metrics state: UP

job: nodeexporter endpoint: http://nodeexporter:9100/metrics state: UP

This confirms Prometheus is successfully scraping itself and the Node Exporter. If any target was down (state would be DOWN with an error message), double-check that the container is running and the hostname/port in prometheus.yml is correct. In our case, because Grafana and Prometheus are on the same Docker network as Node Exporter, and we used the service name nodeexporter, it resolves correctly. (If you see an error like “address not found” for nodeexporter, it means the networking isn’t set up – ensure you used --network monitoring for all relevant containers.)

If your application itself exposes metrics (for instance, if the backend had a /metrics endpoint for Prometheus), you could add another job in the Prometheus config to scrape it. For example, if the backend on port 5000 provides metrics, you’d add a job with target backend:5000. By default, our setup only captures system metrics and Prometheus’ own metrics.



## 5. Setting Up Grafana and Creating Dashboards

With Prometheus collecting data, we can visualize it using Grafana.

Access Grafana: Navigate to http://localhost:3000 and log in (default creds: admin/admin, then set a new password).

Add Prometheus Data Source: Grafana needs to know about our Prometheus server:




<img width="3024" height="1964" alt="image" src="https://github.com/user-attachments/assets/efae2a21-130a-4b7f-89c9-43eafc15726f" />






Click the gear icon (Settings) on the left sidebar, and choose Data Sources.

Click Add data source, then select Prometheus from the list.

In the Prometheus configuration:

Set the Name (e.g. Prometheus or prometheus-1).

Set the URL to Prometheus. Since Grafana is running in Docker on the same network as Prometheus, we can use the container name. For example: http://prometheus:9090. (If that doesn’t work due to networking, you could use http://localhost:9090 if Prometheus is exposed to the host. Remember that inside the Grafana container, localhost:9090 would refer to Grafana itself, not the host, unless you use special DNS. In Docker Desktop environments, you can use http://host.docker.internal:9090 to reference the host machine’s Prometheus port
grafana.com
.)

Leave other settings default and click Save & Test. You should see a “Data source is working” message.

Grafana is now connected to Prometheus. If Grafana has trouble connecting (e.g., a network error), check that the URL is correct:

If you see an error like “No such host” or “connection refused”, it likely means Grafana cannot resolve the hostname or reach the address. Ensure Grafana and Prometheus are on the same Docker network so that the hostname works. The special hostname host.docker.internal (on Docker for Mac/Windows) can be used as a last resort to point to the host’s Prometheus port if you bound Prometheus to the host
grafana.com
. For example, use http://host.docker.internal:9090 if Prometheus is reachable on the host but not via container DNS.

If using container names, double-check the spelling and network. In our setup, because we used the monitoring network and named the container “prometheus”, the URL http://prometheus:9090 should work from Grafana. If it didn’t, one fix would be to join Grafana to that network (--network monitoring which we did). Once fixed, click Save & Test again until it says it’s working.

Create or Import a Dashboard: Now the fun part – visualize the metrics.

Grafana allows you to import pre-built dashboards from its community. For example, to monitor system metrics from Node Exporter, you can use the official Node Exporter Full dashboard. In Grafana, click + (Create) > Import, enter the dashboard ID (for Node Exporter Full, it’s 1860), and click Load. Select your Prometheus data source when prompted, and import it. You’ll get a comprehensive dashboard of CPU, memory, disk, and network graphs using the Node Exporter metrics.

You can also create your own dashboard: click + (Create) > Dashboard, then Add new panel. In the panel editor, select a metric from Prometheus and a visualization type. For example, to see CPU load, you might use a query like avg(rate(node_cpu_seconds_total{mode!="idle"}[5m])) (which shows average CPU usage across cores). Customize the graph as needed and save the dashboard.

Once imported or created, you should see live graphs updating. Grafana dashboards can be changed to different time ranges (upper right corner) – try setting to “Last 5 minutes” or “Last 1 hour” to see recent data. If you imported the Node Exporter Full dashboard, you’ll see panels for things like CPU usage, Memory usage, Filesystem usage, etc., all updating in real time as Prometheus scrapes metrics. You can also set up alerts in Grafana for certain metrics (though that may require configuring an alert notifier).

At this point, we have a fully operational local environment:

Application (frontend, backend, DB) running in containers (or pods) – accessible at http://localhost (frontend) and performing its functions.

CI/CD Pipeline automated via Jenkins – ready to build, test, and deploy new changes.

Monitoring via Prometheus and Grafana – giving us visibility into the system.





## Common Errors and Troubleshooting

Even with careful setup, you might encounter issues. Here are some common problems and how to address them:

Jenkins Agent Disconnection: If you find that Jenkins jobs are failing because the agent (the Jenkins server itself or an external agent) disconnects, it could be due to resource constraints or a Java version mismatch. Ensure your Jenkins controller and any agents run the same Java version, as differing JDK versions are known to cause random disconnects
community.jenkins.io


Jenkins UI not reachable
  docker ps | grep jenkins; docker logs jenkins
  Ensure port 8080 not blocked; check jenkins_home permissions; restart container
  
Can't fetch initial admin password
  docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword


Prometheus target DOWN
  Open Prometheus UI (http://localhost:9090) → Status → Targets; docker logs prometheus
  docker exec -it prometheus sh -c "curl -s nodeexporter:9100/metrics" to test connectivity


Grafana cannot connect to Prometheus
 Ensure Grafana is on same Docker network; in Grafana set URL to http://prometheus:9090 or host.docker.internal if using host


Backend cannot connect to MongoDB
  docker network inspect monitoring; docker logs backend; ensure DB_HOST=mongodb and mongodb service name matches


Image build/push fails in Jenkins
Check Docker credentials in Jenkins; test docker login manually; ensure Jenkins has Docker permissions or use Docker-in-Docker agent




##Security (minimal)


Do not commit tokens, passwords or sensitive files. Use Jenkins Credentials and .env.example (not .env) in repo.

Run containers as non-root where possible. Avoid apt-get into official Jenkins container — prefer agent images or custom images built from a 
Dockerfile with required tools.

Limit resource usage with Docker Compose resource constraints or Kubernetes resource requests/limits.

If you run registry pushes, use tokens or service accounts with least privilege.

For  production, encrypt secrets and use a secrets manager (Vault, AWS Secrets Manager, etc.) or Kubernetes Secrets with proper RBAC.





Usage

With everything up and running, here’s how you can interact with the system:

Frontend Application: Open your browser to http://localhost (assuming you mapped the frontend to port 80). You should see the web application’s frontend. This is the user interface of the three-tier app. Try using the app – for example, if it’s a demo blog or e-commerce app, navigate through pages or perform some action. The frontend will make API calls to the backend (in our setup, likely to http://localhost:5000 via the browser, or directly to http://backend:5000 if configured internally).

Backend API: The backend is not directly accessed by end-users (the frontend uses it), but you can test it via API calls. For example, visit http://localhost:5000/api/health or whichever endpoint might exist to check if it returns a response. You can also use tools like curl or Postman to send requests to the backend. This can help verify that the backend can talk to the database and is functioning. Check the backend container logs (docker logs backend) for any runtime errors – it will log database connections or application errors if something is wrong.

Jenkins Pipeline: Access the Jenkins UI at http://localhost:8080. You can see the pipeline job (e.g. “three-tier-devsecops-pipeline”) and its build history. If you click on a build and then Console Output, you can review the logs of each stage. After running the pipeline, you might also see results of the SonarQube scan in the console (and you can log into your SonarQube server to see the detailed report). If you commit new changes to the repo, you can trigger the pipeline again to go through the whole process (or set up webhook/auto triggers). Jenkins will continuously integrate and deliver your application with each change.

Prometheus UI: While Grafana is the friendly way to view data, you can also use Prometheus’s web UI for quick queries or status. At http://localhost:9090, try querying some metrics or check Status > Targets as discussed. This interface is mostly for debugging and exploring metrics via PromQL.






<img width="3024" height="1964" alt="image" src="https://github.com/user-attachments/assets/e27cb154-226f-4588-9b03-21a9c8407e49" />







Grafana Dashboards: Go to http://localhost:3000 and view your dashboards. For instance, if you imported the Node Exporter Full dashboard, you’ll see panels like CPU usage (with per-core graphs), memory usage, network traffic, etc. This gives you real-time feedback on how your system is doing. If you stress-test your application (e.g. make many requests or use a load testing tool), you should see corresponding changes in the metrics (CPU usage rising, maybe memory or network I/O increasing, etc.). Grafana can be used to observe both the infrastructure and potentially application-level metrics (if you instrumented the app).





<img width="3024" height="1964" alt="image" src="https://github.com/user-attachments/assets/0e87fd0f-8c1a-4a67-b7b6-2eebc617013a" />






Stopping the Environment: To stop all containers, you can either stop them individually (docker stop <container>) or if you used Docker Compose, use docker-compose down. Since we ran containers individually, you’ll have to stop each or simply close Docker if you’re using Docker Desktop. Remember that the named volumes (like jenkins_home) and any data inside containers (like MongoDB data if not on a volume) will persist unless you remove them.

Cleaning Up: If you want to remove everything, stop and remove the containers (docker rm -f jenkins frontend backend mongodb prometheus nodeexporter grafana). Also remove any named volumes if you want a fresh start (docker volume rm jenkins_home will wipe Jenkins data, and if you created a volume for MongoDB data, remove that too). Be careful with removing volumes as you will lose data e.g., Jenkins configurations or database records. For a quick rebuild of the environment, it might be easier to keep volumes and just restart containers when needed.

By following these steps, you have a mini-production environment on your local machine. You can develop your application, commit code, and watch Jenkins automatically build/test/deploy it. You can then observe the app’s behavior and performance via Grafana dashboards. This setup not only demonstrates DevSecOps principles CI/CD, integrated security scanning, infrastructure as code, but also is a great sandbox for learning and improving the pipeline and monitoring before applying similar techniques to a cloud or production environment.




## Verify:

 Frontend → http://localhost  
 
 Backend API → http://localhost:5000 
 
 Jenkins → http://localhost:8080  
 
 Prometheus → http://localhost:9090  
 
 Grafana → http://localhost:3000  



