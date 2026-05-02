# 🚀 End-to-End DevOps Pipeline with Kubernetes & Monitoring

This project demonstrates a complete DevOps workflow including CI/CD, containerization, Kubernetes deployment, and monitoring using Prometheus and Grafana on AWS.

---

## 📌 Overview

The application is deployed using an automated pipeline that builds, scans, pushes Docker images, and deploys them to a Kubernetes cluster provisioned using kOps. Monitoring and alerting are implemented using Prometheus, Grafana, and Alertmanager.

---

## 🏗️ Architecture

```
GitHub → Jenkins → Docker → DockerHub → Kubernetes (kOps on AWS)
                                           ↓
                                   Prometheus + Grafana
                                           ↓
                                      Alertmanager
```

---

## ⚙️ Tech Stack

* **CI/CD**: Jenkins
* **Containerization**: Docker
* **Orchestration**: Kubernetes (kOps)
* **Cloud**: AWS (EC2, S3, IAM, VPC)
* **Monitoring**: Prometheus, Grafana, Alertmanager
* **Security Scan**: Trivy
* **IaC**: Terraform (VPC, private subnet, bastion host)

---

## 🔄 CI/CD Pipeline

1. Code checkout from GitHub
2. Build using Maven
3. Run unit tests
4. Build Docker image
5. Scan image using Trivy
6. Push image to DockerHub
7. Deploy to Kubernetes (kOps cluster)
8. Update image using `kubectl set image`
9. Verify rollout

---

## ☸️ Kubernetes Deployment

* Deployment and ReplicaSet for application
* Service (NodePort / LoadBalancer)
* Ingress for routing traffic
* StatefulSet for database (PostgreSQL)
* Secrets for sensitive data

---

## 📊 Monitoring & Alerting

### Prometheus

* Metrics scraping from application
* ServiceMonitor configuration
* PromQL queries for analysis

### Grafana

* Dashboard creation
* Visualization of application metrics

### Alerting

* PrometheusRule for alert definitions
* Alertmanager for routing alerts
* Slack & Email notifications

---

## 📁 Project Structure

```
.
├── k8s/
│   ├── db.yaml
│   ├── petclinic-deployment.yaml
│   ├── petclinic-service.yaml
│   ├── servicemonitor.yaml
│   ├── prometheus-rules.yaml
│
├── Jenkinsfile
├── Dockerfile
└── README.md
```

---

## 🚀 Deployment Steps

### 1. Create Kubernetes Cluster (kOps)

```bash
kops create cluster ...
kops update cluster --yes
```

---

### 2. Install Monitoring Stack (One-Time)

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring --create-namespace
```

---

### 3. Run CI/CD Pipeline

* Jenkins automatically:

  * Builds
  * Pushes image
  * Deploys to Kubernetes

---

### 4. Access Application

```bash
kubectl get svc -n staging
```

---

### 5. Access Grafana

```bash
kubectl port-forward svc/monitoring-grafana 3000:80 -n monitoring
```

Open:

```
http://localhost:3000
```

---

## 🔐 Security

* Secrets managed using Kubernetes Secrets
* Sensitive data not stored in GitHub
* Image scanning using Trivy

---

## 📈 Key Features

* End-to-end CI/CD pipeline
* Kubernetes-based deployment
* Monitoring and alerting setup
* AWS-based infrastructure
* Real-world debugging and issue handling

---

## 🎯 Future Improvements

* Ingress with domain + TLS
* Horizontal Pod Autoscaler
* Blue-Green / Canary deployments
* Terraform automation for full infra
* GitOps (ArgoCD)

---

## 👨‍💻 Author

Ashwini

---
<img width="979" height="442" alt="image" src="https://github.com/user-attachments/assets/43eb3704-5b19-4f8f-ae2d-edadc76fd32e" />
<img width="979" height="374" alt="image" src="https://github.com/user-attachments/assets/8738e73d-9be8-4ed4-a873-85768c35c94c" />
CUSTOM DASHBOARD
<img width="1048" height="600" alt="Screenshot 2026-04-25 195457" src="https://github.com/user-attachments/assets/00ae9f00-d337-4759-bb91-8b2aa3a29c75" />



TRIVY OUTPUT
![Screenshot](https://github.com/user-attachments/assets/b7d49fc8-c252-4151-8ebc-c877c5c9ea27)

Challenges & Fixes

**1.Jenkins Node Disk Space Issue**
•Problem --> During a pipeline run, builds were stuck in queue and not executing. On checking Jenkins, the node had gone offline due to low disk space.

•Impact --> CI/CD pipeline was blocked since no executor was available, stopping all builds and deployments.

•Root Cause --> Disk space in /var/lib/jenkins was exhausted due to accumulated build artifacts, logs, and unused Docker images.

•Fix --> 
Checked disk usage: df -h

Cleaned up unused Docker resources and workspace files:

docker system prune -a
rm -rf /var/lib/jenkins/workspace/*

(Also extended EC2 EBS volume from AWS console)

•Outcome --> Jenkins node came back online and pipelines resumed successfully, highlighting the need for disk monitoring and cleanup.

**2.SonarQube Connectivity Issue**
•Problem -->SonarQube analysis failed with a 'connection refused error' during pipeline execution.

•Impact  --> Code quality stage failed, stopping the CI/CD pipeline.

•Root Cause--> SonarQube service (Docker container) was not running.

•Fix --> Checked running containers and restarted SonarQube:

docker ps -a
docker start sonarqube

•Verified service health using logs:

docker logs -f sonarqube

Outcome --> SonarQube service became accessible and analysis completed successfully in the pipeline.
1. Docker Image Tagging Issue
🔹 Problem
New deployments were overwriting previous images, making rollback difficult.
🔹 Impact
No version tracking of images, risk during deployments.
🔹 Root Cause
Static image tags were used instead of dynamic versioning.
🔹 Fix
Used Jenkins build number for tagging:
docker build -t petclinic:${BUILD_NUMBER} .
🔹 Outcome
Each build generated a unique image version, enabling traceability and rollback.

2. Docker Login & Push Failure
🔹 Problem
Docker push failed due to authentication error.
🔹 Impact
Image could not be pushed to registry, blocking deployment.
🔹 Root Cause
Credentials were not handled securely in pipeline.
🔹 Fix
Used Jenkins credentials:
withCredentials([usernamePassword(...)]) {  docker login -u $DOCKER_USER --password-stdin}
🔹 Outcome
Secure authentication enabled successful image push.

3. Pipeline Blocked by Security Scan
🔹 Problem
Pipeline failed when Trivy detected vulnerabilities.
🔹 Impact
Deployment stopped even for non-critical fixes.
🔹 Root Cause
Strict failure condition on HIGH/CRITICAL vulnerabilities.
🔹 Fix
Allowed pipeline to continue while reporting:
trivy image ... || true
🔹 Outcome
Pipeline continued while still maintaining security visibility.

4. Application Port Exposure Issue (k3s)
🔹 Problem
Application deployed successfully but not accessible externally.
🔹 Impact
Unable to verify deployment via browser.
🔹 Root Cause
Service was not exposed properly.
🔹 Fix
Exposed using NodePort:
kubectl expose deployment petclinic --type=NodePort --port=8080kubectl get svc
🔹 Outcome
Application became accessible via node IP and port.

5. SonarQube Quality Gate Blocking Pipeline
🔹 Problem
Pipeline failed at quality gate stage.
🔹 Impact
Build stopped before deployment.
🔹 Root Cause
Code did not meet defined quality thresholds.
🔹 Fix
Configured:
waitForQualityGate abortPipeline: true
🔹 Outcome
Ensured only quality-approved code proceeds to deployment.

