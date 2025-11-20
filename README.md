# Three-Tier DevSecOps Full-Stack Web Application Deployment (Local with Docker, Prometheus, Grafana, and Jenkins)



### Overview


This project features a complete DevSecOps pipeline and local deployment for a three-tier web application (frontend, backend, database), which showcases how an application can be continually integrated, tested (with security scans), containerized, and deployed to your local machine using Docker with end-to-end monitoring via Prometheus and Grafana. The Jenkins CI/CD pipeline automates the build and deploy process (as well as the security scanning process with the SonarQube code analysis stage). When you follow this guide you will be able to replicate the production-like environment to your local machine with automated secured releases and real-time monitoring.


## Architecture

Three-Tier DevSecOps Full-Stack Web Application Deployment (Local with Docker, Prometheus, Grafana, and Jenkins): 

The application is divided into three tiers, each tier running in its own container. 

The tiers are

Frontend: user interface (e.g. React SPA), served through a web server (e.g. Nginx). 
Backend: API server (e.g. Node.js/Express app) which is the business logic layer and communicates with the frontend (through HTTP API calls) and the database. 
Database: the database server (e.g. MongoDB), which holds the application's data. Containerized Deployment: Each tier is packaged as a docker image. In a local deployment, each one of these layers is executed (e.g. via Docker Compose, or docker run,) and the layers are networked together. 

The frontend layer is making HTTP calls to the backend layer, while the backend is able to access the db via the network alias of the database container. CI/CD Pipeline is controlled by a Jenkins server.


Monitoring and Logging: The deployment includes a monitoring stack:

Prometheus: Collects metrics from various targets (e.g. the host machine via Node Exporter, or app-specific metrics if exposed).

Node Exporter: Exposes host machine metrics (CPU, memory, etc.) to Prometheus.

Grafana: Connects to Prometheus to visualize metrics on dashboards. This provides insight into the health and performance of the application and infrastructure in real time.

Overall, this architecture ensures the application is delivered in a repeatable, automated way (thanks to Jenkins and Docker), and that it’s observable via metrics (thanks to Prometheus and Grafana). Security is integrated by including steps like static code analysis in the pipeline (putting the “Sec” in DevSecOps).




## Tools Used and Why




<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/d1c0d7fc-7d3a-4a9a-a774-86c6afbb3e23" />





Jenkins: Used as the Continuous Integration/Continuous Deployment tool. Jenkins Pipeline (defined in a Jenkinsfile) automates building the code, running tests, performing security scans, and deploying the application. Jenkins orchestrates the DevSecOps workflow end-to-end, from pulling code to pushing containers and deploying updates.

Docker: Used for containerizing all parts of the application and infrastructure tools. Docker ensures environment consistency across deployments. We containerize the frontend, backend, and database, as well as Jenkins itself (running Jenkins in a Docker container), Prometheus, Grafana, etc. Docker Compose or Docker networks manage the multi-container setup and networking.

Prometheus: Used for monitoring and metrics collection. Prometheus (running as a container) scrapes metrics from targets like Node Exporter (for host metrics) and possibly the application (if the app exposes a /metrics endpoint). Prometheus stores these time-series metrics and allows querying them via PromQL. This provides data on system performance (CPU usage, memory usage, etc.) and application behavior.

Grafana: Used for metrics visualization and dashboards. Grafana (containerized) connects to Prometheus as a data source and displays the collected metrics on configurable dashboards. We use Grafana to visualize key metrics (e.g. server CPU load, memory usage, container status) to quickly identify issues. Grafana also supports setting up alerts on certain conditions (though in a local setup, this is mainly for demonstration).

SonarQube (Scanner): Used for static code analysis and security scanning in the CI pipeline. In our Jenkins pipeline, there is a SonarQube scan stage that analyzes the code for bugs and vulnerabilities. We utilize the SonarQube Scanner CLI to run the analysis. (In a full setup, a SonarQube server would be running as a separate service or container – the pipeline sends analysis data to that server. Here we assume a SonarQube server is accessible at http://sonar:9000 as configured in the Jenkinsfile.)


Additionally, other DevSecOps tools like Trivy (container vulnerability scanning) or Nexus (artifact repository) could be integrated, but the core focus here is on Jenkins, Docker, Prometheus, and Grafana. The SonarQube stage already covers code quality/security scanning in this project.)





**End-to-End DevSecOps Workflow:** Incorporate security scans, monitoring, and scalable infrastructure for a production-ready deployment.





To make learning even easier, This is a detailed guide, walking you through each step visually. Whether you're a DevOps enthusiast or a seasoned cloud professional, this tutorial will help you bridge the gap between theory and practical implementation.




Setup Instructions
1. Cloning the Repository

Begin by obtaining the project source code from the repository. Clone the Git repository to your local machine using Git:

```bash
git clone https://github.com/neycloud/three-tier-devsecops-main.git
cd three-tier-devsecops-main

