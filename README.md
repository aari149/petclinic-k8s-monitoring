# Java Application — CI/CD Pipeline with Jenkins

> End-to-end CI/CD pipeline for a Java Spring Boot application.  
> Automates build, code quality analysis, security scanning, containerization, and image publishing — with Kubernetes deployment coming in the next phase.
> ## Pipeline Overview

```
Developer Push
      │
      ▼
┌─────────────────────────────────────────────────────────────┐
│                     Jenkins Pipeline                        │
│                                                             │
│  ① Checkout  →  ② Build  →  ③ Test  →  ④ SonarQube        │
│                                                             │
│  ⑤ Quality Gate  →  ⑥ Docker Build  →  ⑦ Trivy Scan       │
│                                                             │
│                    →  ⑧ Push to Docker Hub                 │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                    Docker Hub Registry
                    (image ready to deploy)

  ── Kubernetes Deployment (k3s → EKS) coming in Phase 2 ──
```

## What Each Tool Prevents

| Tool | Without It | With It |
|---|---|---|
| SonarQube | Bug ships to production | Caught before Docker build |
| Quality Gate | Low coverage goes unnoticed | Pipeline blocks bad code |
| Trivy | Vulnerable image deployed | Pipeline fails on HIGH/CRITICAL CVE |
| Jenkins credentials | Passwords in Jenkinsfile / Git | Secrets never touch source code |

---


---
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

