---
title: "Nginx"
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

# Nginx

Web server e reverse proxy — serving estático, load balancing, e TLS termination.

## O que é

Nginx é um web server de alto desempenho usado como reverse proxy, load balancer, e servidor de arquivos estáticos. Baseado em event-driven architecture (similar ao Node.js), consegue servir milhares de conexões simultâneas com baixo consumo de memória.

## Como funciona

### Casos de uso

```text
Internet → Nginx (reverse proxy + TLS)
              ├── /api/* → Backend (Spring Boot :8080)
              ├── /       → Frontend (React SPA, arquivos estáticos)
              └── /ws     → WebSocket backend
```

### Configuração típica

```nginx
server {
    listen 443 ssl;
    server_name app.example.com;

    ssl_certificate /etc/ssl/cert.pem;
    ssl_certificate_key /etc/ssl/key.pem;

    # Frontend SPA
    location / {
        root /var/www/html;
        try_files $uri $uri/ /index.html;  # SPA fallback
    }

    # API proxy
    location /api/ {
        proxy_pass http://backend:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # WebSocket
    location /ws {
        proxy_pass http://backend:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Load Balancing

```nginx
upstream backend {
    server api1:8080 weight=3;
    server api2:8080 weight=1;
    server api3:8080 backup;
}
```

Algoritmos: round-robin (default), least_conn, ip_hash, weight.

### Features

- **Reverse Proxy:** esconde servidores backend, centraliza entrada
- **TLS Termination:** HTTPS no Nginx, HTTP interno (mais simples para backends)
- **Static Files:** serve HTML/CSS/JS/imagens com alta performance
- **Caching:** cache de respostas do backend
- **Rate Limiting:** `limit_req_zone` para proteção contra abuse
- **Gzip:** compressão automática de respostas

## Quando usar

- **Reverse proxy:** frente para qualquer aplicação web
- **Static file serving:** SPAs (React, Vue), assets
- **Load balancer:** distribuir tráfego entre múltiplas instâncias
- **TLS termination:** certificados HTTPS centralizados
- **Ingress controller:** em Kubernetes, Nginx Ingress é o mais popular

## How to explain in English

"Nginx is the front door of most web applications I deploy. I use it as a reverse proxy that handles TLS termination, serves static frontend assets, and routes API requests to backend services. In Kubernetes, Nginx Ingress Controller provides the same functionality as a cluster-level resource."

### Key vocabulary

- proxy reverso → reverse proxy
- balanceamento de carga → load balancing
- terminação TLS → TLS termination
- servidor web → web server
- conteúdo estático → static content

## Veja também

- [[Docker]]
- [[Kubernetes]]
- [[System Design]]
- [[Redes e Protocolos]]
