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

1.Jenkins Node Disk Space Issue
Problem --> During a pipeline run, builds were stuck in queue and not executing. On checking Jenkins, the node had gone offline due to low disk space.

Impact --> CI/CD pipeline was blocked since no executor was available, stopping all builds and deployments.

Root Cause --> Disk space in /var/lib/jenkins was exhausted due to accumulated build artifacts, logs, and unused Docker images.

Fix --> 
Checked disk usage: df -h

Cleaned up unused Docker resources and workspace files:

docker system prune -a
rm -rf /var/lib/jenkins/workspace/*

(Also extended EC2 EBS volume from AWS console)

Outcome --> Jenkins node came back online and pipelines resumed successfully, highlighting the need for disk monitoring and cleanup.

2.SonarQube Connectivity Issue
Problem -->SonarQube analysis failed with a 'connection refused error' during pipeline execution.

Impact  --> Code quality stage failed, stopping the CI/CD pipeline.

Root Cause--> SonarQube service (Docker container) was not running.

Fix --> Checked running containers and restarted SonarQube:

docker ps -a
docker start sonarqube

Verified service health using logs:

docker logs -f sonarqube

Outcome --> SonarQube service became accessible and analysis completed successfully in the pipeline.
