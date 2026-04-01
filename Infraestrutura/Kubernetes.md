---
title: "Kubernetes"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - infraestrutura
  - devops
  - entrevista
publish: false
---

# Kubernetes

Orquestração de containers em escala — deploy, scaling, e self-healing automáticos.

## O que é

Kubernetes (K8s) é uma plataforma de orquestração de containers que automatiza deploy, scaling e gerenciamento. Criado pelo Google, é o padrão da indústria para rodar containers em produção. Para entrevistas, entender os conceitos core é mais importante que dominar `kubectl`.

## Como funciona

### Arquitetura

```text
Control Plane
  ├── API Server (gateway para tudo)
  ├── Scheduler (decide onde rodar pods)
  ├── Controller Manager (mantém estado desejado)
  └── etcd (banco de dados do cluster)

Worker Nodes
  ├── kubelet (agente do node)
  ├── kube-proxy (networking)
  └── Container Runtime (Docker, containerd)
```

### Objetos essenciais

| Objeto | O que faz |
| --- | --- |
| Pod | Menor unidade. 1+ containers que compartilham rede e storage |
| Deployment | Gerencia réplicas de pods. Rolling updates, rollbacks |
| Service | Expõe pods com IP estável e load balancing interno |
| Ingress | Roteamento HTTP externo (como nginx para o cluster) |
| ConfigMap | Configuração externalizada (não-sensitiva) |
| Secret | Dados sensitivos (passwords, tokens) |
| PersistentVolume | Storage persistente para pods |
| HPA | Horizontal Pod Autoscaler — escala baseado em métricas |

### Exemplo de Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
        - name: api
          image: myapp:1.0.0
          ports:
            - containerPort: 8080
          resources:
            requests: { memory: "256Mi", cpu: "250m" }
            limits: { memory: "512Mi", cpu: "500m" }
          readinessProbe:
            httpGet: { path: /health, port: 8080 }
```

### Conceitos importantes

- **Rolling Update:** atualiza pods gradualmente sem downtime
- **Self-healing:** pod morreu → K8s cria outro automaticamente
- **Service Discovery:** pods se encontram pelo nome do Service (DNS interno)
- **Namespaces:** isolamento lógico (dev, staging, prod)
- **Helm:** package manager para K8s (charts = templates de deploy)

## Quando usar

- **Microserviços em produção:** orquestração, scaling, service discovery
- **Auto-scaling:** HPA baseado em CPU, memória ou métricas custom
- **Multi-ambiente:** namespaces para dev/staging/prod no mesmo cluster
- **Alta disponibilidade:** réplicas + self-healing

## Quando NÃO usar

- **Aplicação simples:** docker compose ou PaaS (Heroku, Railway) é suficiente
- **Equipe pequena sem experiência:** K8s tem curva de aprendizado alta
- **Custo importa mais que escala:** cluster K8s tem overhead de resources

## How to explain in English

"Kubernetes is the orchestration layer I've worked with for running microservices in production. At Muvz, I collaborated with DevOps to deploy a Kafka cluster and Spring Boot microservices on Kubernetes. The key concepts I work with are Deployments for managing replicas with rolling updates, Services for internal load balancing, and Ingress for external HTTP routing.

What I appreciate about Kubernetes is the declarative model — you describe the desired state, and K8s works to maintain it. If a pod crashes, it's automatically replaced. If traffic increases, HPA scales up the replicas."

### Key vocabulary

- pod → pod: menor unidade de deploy
- nó → node: máquina no cluster
- implantação → deployment: gerencia réplicas
- serviço → service: load balancer interno
- entrada → ingress: roteamento HTTP externo
- escala automática → autoscaling (HPA)
- auto-recuperação → self-healing

## Veja também

- [[Docker]]
- [[Nginx]]
- [[System Design]]
