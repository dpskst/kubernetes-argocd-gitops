# Kubernetes GitOps with ArgoCD

Kubernetes 환경에서 ArgoCD를 활용하여 Git Repository의 Kubernetes Manifest를 기준으로 애플리케이션을 배포하고 관리하는 GitOps 프로젝트입니다.

## 📌 Project Overview

Kubernetes 리소스를 YAML Manifest로 GitHub에서 관리하고, ArgoCD가 Git Repository의 변경사항을 Kubernetes Cluster에 자동으로 반영하도록 구성했습니다.

Git Repository를 Kubernetes 배포 상태의 기준점으로 사용하여 애플리케이션의 배포 상태를 추적하고 관리하는 것을 목표로 합니다.

## 🏗️ Architecture

```text
GitHub
  │
  │ Kubernetes Manifest
  ▼
ArgoCD
  │
  │ Sync
  ▼
Kubernetes Cluster
  │
  ├── Deployment
  │      └── my-devops-app
  │
  └── Service
         └── my-devops-app
```

## 🛠️ Tech Stack

| Category                | Technology   |
| ----------------------- | ------------ |
| Container Orchestration | Kubernetes   |
| GitOps                  | ArgoCD       |
| Cluster                 | Kind         |
| Container               | Docker       |
| Configuration           | YAML         |
| Version Control         | Git / GitHub |

## 📂 Project Structure

```text
my-k8s-gitops
├── k8s/
│   ├── argocd-application.yaml
│   ├── deployment.yaml
│   └── service.yaml
├── app/
│   ├── Dockerfile
│   └── index.html
├── .github/
│   └── workflows/
│       └── ci-cd.yml
└── README.md
```

## ☸️ Kubernetes Resources

### Deployment

애플리케이션 Pod를 관리하기 위한 Deployment를 구성했습니다.

현재 3개의 Replica로 구성했습니다.

```yaml
spec:
  replicas: 3
```

이를 통해 Kubernetes의 Replica 관리와 Pod 확장 구성을 확인했습니다.

### Service

Kubernetes 내부에서 애플리케이션 Pod에 접근할 수 있도록 Service를 구성했습니다.

```text
Service
   │
   ├── Pod
   ├── Pod
   └── Pod
```

Service의 Selector를 통해 `my-devops-app` Pod와 연결됩니다.

## 🔄 GitOps Workflow

### 1. Kubernetes Manifest 수정

Git Repository의 `k8s/` 디렉터리에 있는 Kubernetes Manifest를 수정합니다.

예:

```yaml
spec:
  replicas: 3
```

### 2. Git Commit & Push

변경사항을 GitHub에 Push합니다.

```bash
git add .
git commit -m "Scale application to 3 replicas"
git push origin main
```

### 3. ArgoCD 감지

ArgoCD가 Git Repository의 변경사항을 확인합니다.

### 4. Kubernetes Sync

ArgoCD가 Git Repository의 Manifest와 Kubernetes Cluster의 현재 상태를 비교하고 변경사항을 Cluster에 반영합니다.

```text
GitHub
   │
   ▼
ArgoCD
   │
   ▼
Kubernetes
   │
   ▼
Deployment
   │
   ▼
Pods
```

## 🔧 ArgoCD Application

ArgoCD Application은 다음 Git Repository를 바라보도록 구성했습니다.

```text
Repository:
https://github.com/dydcjsrjaror/my-k8s-gitops.git

Branch:
main

Path:
k8s
```

`k8s/` 디렉터리의 Manifest를 기준으로 Kubernetes Cluster와 동기화합니다.

## 🧪 Kubernetes Environment

Kind를 이용하여 로컬 Kubernetes Cluster를 구성했습니다.

```text
devops-cluster

├── control-plane
├── worker
└── worker
```

Cluster 상태 확인:

```bash
kubectl get nodes
```

Deployment 확인:

```bash
kubectl get deployment
```

Pod 확인:

```bash
kubectl get pods
```

Service 확인:

```bash
kubectl get svc
```

## 🔎 ArgoCD 상태 확인

ArgoCD Application 상태를 확인할 수 있습니다.

```bash
kubectl get application -n argocd
```

정상적으로 동기화된 경우:

```text
SYNC STATUS    HEALTH STATUS
Synced         Healthy
```

## 📈 Replica 변경 테스트

Git Repository의 Deployment Manifest를 수정하여 Replica 수를 변경할 수 있습니다.

예:

```yaml
replicas: 3
```

변경 후 Git Push를 수행하면 ArgoCD가 변경사항을 감지하고 Kubernetes Cluster에 반영합니다.

```bash
kubectl get pods
```

Pod 수를 확인하여 변경 결과를 검증할 수 있습니다.

## 🎯 Project Goals

* Kubernetes 기본 리소스 구성
* Deployment / Service 이해
* Replica 기반 Pod 관리
* Git 기반 Kubernetes Manifest 관리
* ArgoCD GitOps 환경 구성
* Git Repository와 Kubernetes Cluster의 상태 동기화
* Kubernetes 배포 상태 확인 및 검증

## 📚 What I Learned

* Kubernetes Deployment와 Service 구성
* Replica를 이용한 Pod 관리
* YAML 기반 Kubernetes 리소스 관리
* ArgoCD Application 구성
* GitOps의 기본 동작 방식
* Git Repository와 Kubernetes Cluster의 상태 관리
* ArgoCD Sync / Health 상태 확인
* Kubernetes 리소스 변경 및 검증
