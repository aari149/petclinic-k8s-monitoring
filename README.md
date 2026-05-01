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
