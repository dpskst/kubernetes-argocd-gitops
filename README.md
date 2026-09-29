# Kubernetes ArgoCD GitOps

Kubernetes와 ArgoCD를 활용하여 Git Repository의 Kubernetes Manifest를 기준으로 애플리케이션 배포 상태를 자동으로 관리하는 GitOps 프로젝트입니다.

## Project Overview

Kubernetes 환경에서 애플리케이션의 Deployment 설정을 Git으로 관리하고, ArgoCD를 통해 Git Repository와 Kubernetes 클러스터의 상태를 지속적으로 동기화하도록 구성했습니다.

Git의 Manifest 변경을 기준으로 Kubernetes 리소스가 자동으로 반영되도록 구성하여 GitOps 기반의 배포 및 상태 관리 과정을 구현했습니다.

## Architecture

```text
Developer
    │
    │ git push
    ▼
GitHub Repository
    │
    │ Manifest 변경 감지
    ▼
ArgoCD
    │
    │ Sync
    ▼
Kubernetes Cluster
    │
    ├── Deployment
    │
    └── Pods
```

## Environment

| Category                | Technology     |
| ----------------------- | -------------- |
| Container Orchestration | Kubernetes     |
| Kubernetes Distribution | Kind           |
| GitOps                  | ArgoCD         |
| Version Control         | Git / GitHub   |
| Operating System        | Windows        |
| Application             | Nginx          |
| Cluster                 | devops-cluster |

## Kubernetes Cluster

Kind를 사용하여 Kubernetes 클러스터를 구성했습니다.

```text
devops-cluster
├── control-plane
├── worker
└── worker2
```

클러스터 노드 상태를 확인합니다.

```bash
kubectl get nodes
```

### Kubernetes Nodes

![Kubernetes Nodes](docs/screenshots/kubernetes-nodes.png)

Kind로 구성한 Kubernetes Cluster의 Control Plane과 Worker Node가 모두 `Ready` 상태로 동작하는 것을 확인했습니다.

## Project Structure

```text
kubernetes-argocd-gitops
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── application.yaml
├── docs/
│   └── screenshots/
└── README.md
```

## ArgoCD Installation

ArgoCD Namespace를 생성합니다.

```bash
kubectl create namespace argocd
```

ArgoCD를 설치합니다.

```bash
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

ArgoCD Pod 상태를 확인합니다.

```bash
kubectl get pods -n argocd
```

## ArgoCD UI

ArgoCD Server에 Port Forward를 설정합니다.

```bash
kubectl port-forward svc/argocd-server -n argocd 8443:443
```

브라우저에서 다음 주소로 접속합니다.

```text
https://localhost:8443
```

ArgoCD UI에서 Application의 Sync 상태와 Kubernetes 리소스 상태를 확인할 수 있습니다.

## GitOps Application

ArgoCD Application은 다음 GitHub Repository를 바라보도록 구성했습니다.

```text
https://github.com/dydcjsrjaror/kubernetes-argocd-gitops.git
```

```yaml
spec:
  source:
    repoURL: https://github.com/dydcjsrjaror/kubernetes-argocd-gitops.git
    targetRevision: main
    path: k8s
```

Git Repository의 `k8s` 디렉터리에 있는 Kubernetes Manifest를 기준으로 클러스터 상태를 관리합니다.

### ArgoCD Application

![ArgoCD Application](docs/screenshots/argocd-application.png)

ArgoCD Application이 Git Repository와 정상적으로 동기화되어 `Synced`, `Healthy` 상태로 동작하는 것을 확인했습니다.

## Deployment

애플리케이션 Deployment는 Kubernetes Manifest를 통해 관리합니다.

```yaml
spec:
  replicas: 3
```

Kubernetes에서 Deployment와 Pod 상태를 확인합니다.

```bash
kubectl get deployment
kubectl get pods
```

## GitOps Synchronization Test

GitOps 동작을 확인하기 위해 Deployment의 Replica 수를 변경했습니다.

기존 Git 설정:

```yaml
spec:
  replicas: 3
```

Git Repository의 Manifest를 수정하여:

```yaml
spec:
  replicas: 5
```

변경 사항을 Git에 반영합니다.

```bash
git add .
git commit -m "Scale application replicas to 5"
git push origin main
```

ArgoCD가 Git Repository의 변경 사항을 감지하고 Kubernetes Cluster에 변경 사항을 Sync합니다.

### GitOps Deployment

![Kubernetes Pods](docs/screenshots/kubernetes-pods-5.png)

Git Repository의 Replica 설정 변경이 ArgoCD를 통해 Kubernetes에 반영되어 `my-devops-app` Pod 5개가 실행되는 것을 확인했습니다.

```bash
kubectl get deployment my-devops-app
kubectl get pods
```

Deployment의 Replica가 5개로 변경된 것을 확인할 수 있습니다.

```text
NAME            READY   UP-TO-DATE   AVAILABLE
my-devops-app   5/5     5            5
```

## GitOps Reconciliation Test

GitOps의 핵심 기능인 Desired State와 Actual State의 일치 과정을 확인했습니다.

Git Repository에서 Replica를 3개로 관리하는 상태에서 Kubernetes에서 직접 Replica를 5개로 변경했습니다.

```bash
kubectl scale deployment my-devops-app --replicas=5
```

Kubernetes의 실제 상태가 Git Repository의 설정과 달라지면 ArgoCD가 이를 감지하고 Git에 정의된 상태로 다시 동기화합니다.

```text
Git Desired State
replicas: 3
        │
        ▼
Kubernetes Manual Change
replicas: 5
        │
        ▼
ArgoCD Reconciliation
        │
        ▼
Kubernetes
replicas: 3
```

### ArgoCD Reconciliation

![ArgoCD Reconciliation](docs/screenshots/argocd-reconciliation.png)

Kubernetes의 실제 상태와 Git Repository의 Desired State가 달라졌을 때 ArgoCD가 변경 사항을 감지하고 다시 동기화하는 과정을 확인했습니다.

## Kubernetes Application

Kubernetes Service를 통해 애플리케이션을 관리합니다.

```bash
kubectl get service
```

현재 애플리케이션은 Kubernetes Cluster 내부에서 Service를 통해 접근하도록 구성했습니다.

## GitHub Repository

Kubernetes Manifest는 GitHub Repository에서 버전 관리합니다.

```text
kubernetes-argocd-gitops
```

### GitHub Repository

![GitHub Repository](docs/screenshots/github-repository.png)

GitHub Repository에서 Kubernetes Manifest와 변경 이력을 관리하여 Kubernetes 설정 변경 사항을 추적할 수 있도록 구성했습니다.

## Project Goals

* Kubernetes 기반 애플리케이션 배포 경험
* ArgoCD 설치 및 운영 경험
* GitOps 기반 Kubernetes 배포 구성
* Git Repository 기반 Kubernetes Manifest 관리
* ArgoCD Sync 및 Health 상태 확인
* Git 변경 사항의 Kubernetes 자동 반영
* ArgoCD Reconciliation 동작 검증
* Kubernetes Desired State 관리 경험

## What I Learned

* Kubernetes Deployment와 Pod 관리
* Kubernetes Service 구성
* Kind 기반 Kubernetes 클러스터 구성
* ArgoCD 설치 및 Application 구성
* GitOps의 Desired State 개념
* Git Repository 기반 Kubernetes Manifest 관리
* ArgoCD Sync 동작
* ArgoCD Reconciliation 동작
* Git 변경 사항의 Kubernetes 자동 반영
* Kubernetes와 ArgoCD를 이용한 배포 상태 관리
