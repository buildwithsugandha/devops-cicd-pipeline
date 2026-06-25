# DevOps CI/CD Pipeline

> End-to-end CI/CD pipeline for a containerized Node.js app — GitHub Actions, Docker, Trivy security scanning, and EKS deployment with automated rollback.

## Pipeline Overview

```
Developer pushes code
         │
         ▼
┌─────────────────────────────────────────────────────────────┐
│                    GitHub Actions Pipeline                   │
│                                                             │
│  Stage 1         Stage 2         Stage 3         Stage 4   │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌─────────┐ │
│  │   Lint   │──▶│  Unit    │──▶│  Docker  │──▶│  Trivy  │ │
│  │  ESLint  │   │  Tests   │   │  Build   │   │ Security│ │
│  │  Prettier│   │  Jest    │   │  & Push  │   │  Scan   │ │
│  └──────────┘   └──────────┘   └────┬─────┘   └────┬────┘ │
│                                     │               │      │
│                               AWS ECR          ❌ FAIL:    │
│                               Registry         Pipeline    │
│                                     │          stops here  │
│                                     ▼                      │
│                              Stage 5                        │
│                           ┌──────────────┐                 │
│                           │  Deploy to   │                 │
│                           │  EKS         │                 │
│                           │  (Rolling)   │                 │
│                           └──────┬───────┘                 │
│                                  │                         │
│                           ✅ Success    ❌ Fail             │
│                                  │           │             │
│                           Traffic           Auto           │
│                           shifted     ──▶  Rollback        │
│                           to new           to previous     │
│                           pods            stable pods      │
└─────────────────────────────────────────────────────────────┘
```

## Problem Statement

Containerized applications deployed manually have no automated testing, no security scanning before production, and no rollback capability when a bad deployment reaches users.

## Solution

A fully automated CI/CD pipeline with security-first design:

- **Stage 1 — Lint**: ESLint + Prettier enforces code quality on every push
- **Stage 2 — Test**: Jest unit tests with coverage reporting
- **Stage 3 — Build**: Docker image built and pushed to AWS ECR with commit SHA tag
- **Stage 4 — Security Scan**: Trivy scans image for CVEs — pipeline fails on HIGH/CRITICAL findings
- **Stage 5 — Deploy**: Rolling deployment to EKS with automatic rollback on health check failure

## Security Gate

Trivy is configured to block deployments with HIGH or CRITICAL CVEs:

```yaml
- name: Run Trivy vulnerability scan
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: ${{ env.ECR_REGISTRY }}/${{ env.ECR_REPOSITORY }}:${{ env.IMAGE_TAG }}
    exit-code: '1'
    severity: 'HIGH,CRITICAL'
```

## Key Outcomes

- Security vulnerabilities caught pre-production — never reaches users
- Automated rollback prevents bad deployments from causing downtime
- Full pipeline runs in under 8 minutes from push to production
- Every deployed image is tagged with Git commit SHA for full traceability

## Technologies

| Category | Tools |
|---|---|
| CI/CD | GitHub Actions |
| Containers | Docker, AWS ECR |
| Orchestration | Kubernetes (EKS) |
| Security | Trivy (CVE scanning) |
| Testing | Jest, ESLint, Prettier |

## Author

**Sugandha Vashishtha** — Cloud & Site Reliability Engineer  
[LinkedIn](https://linkedin.com/in/sugandha-vashishtha) · [Portfolio](https://sugandha.dev)
