---
title: "Docker"
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

# Docker

Containerização de aplicações — empacota código + dependências em ambientes isolados e reproduzíveis.

## O que é

Docker é uma plataforma de containerização que empacota aplicações com todas as suas dependências em containers leves e portáveis. Diferente de VMs, containers compartilham o kernel do OS, sendo muito mais leves e rápidos para iniciar.

## Como funciona

### Conceitos-chave

- **Image:** template imutável com o código, runtime e dependências. Construída a partir de um Dockerfile.
- **Container:** instância em execução de uma image. Isolado, efêmero.
- **Dockerfile:** receita para construir uma image.
- **Docker Compose:** orquestração local de múltiplos containers.
- **Registry:** repositório de images (Docker Hub, ECR, GCR).

### Dockerfile (exemplo Spring Boot)

```dockerfile
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY target/app.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Multi-stage build (reduz tamanho)

```dockerfile
# Build stage
FROM eclipse-temurin:21-jdk AS build
WORKDIR /app
COPY . .
RUN ./mvnw package -DskipTests

# Runtime stage
FROM eclipse-temurin:21-jre-alpine
COPY --from=build /app/target/app.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Docker Compose

```yaml
services:
  api:
    build: .
    ports: ["8080:8080"]
    environment:
      DATABASE_URL: postgresql://db:5432/app
    depends_on: [db, redis]

  db:
    image: postgres:16-alpine
    volumes: [pgdata:/var/lib/postgresql/data]
    environment:
      POSTGRES_DB: app
      POSTGRES_PASSWORD: secret

  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]

volumes:
  pgdata:
```

### Comandos essenciais

```bash
docker build -t myapp .            # Construir image
docker run -p 8080:8080 myapp      # Rodar container
docker compose up -d               # Subir todos os serviços
docker compose logs -f api         # Ver logs
docker exec -it container_id bash  # Entrar no container
docker ps                          # Listar containers rodando
```

## Quando usar

- **Desenvolvimento local:** ambientes reproduzíveis (banco, cache, message broker)
- **CI/CD:** build e teste em ambientes consistentes
- **Deploy:** container = artefato de deploy (mesma image em staging e prod)
- **Microserviços:** cada serviço em seu container

## Armadilhas comuns

- **Images grandes:** usar alpine como base, multi-stage builds, `.dockerignore`
- **Rodar como root:** usar `USER` no Dockerfile para criar usuário non-root
- **Não usar .dockerignore:** `node_modules`, `.git`, `target/` não devem ir para a image
- **Volumes para dados persistentes:** containers são efêmeros — dados vão em volumes

## How to explain in English

"Docker is fundamental to my development workflow. I use Docker Compose for local development — spinning up PostgreSQL, Redis, and Kafka with a single command gives every developer the same environment. For production, I build multi-stage Docker images that are minimal and secure, using alpine-based images and non-root users.

The key principle is: the same image that passes CI is the one deployed to production. This eliminates 'works on my machine' problems entirely."

### Key vocabulary

- contêiner → container: instância isolada de execução
- imagem → image: template para criar containers
- orquestração → orchestration: gerenciamento de múltiplos containers
- registro → registry: repositório de images
- volume → volume: dados persistentes

## Veja também

- [[Kubernetes]]
- [[Nginx]]
- [[WSL, Docker e Kubernetes]]
