---
title: "Redes e Protocolos"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - fundamentos
  - entrevista
publish: false
---

# Redes e Protocolos

Fundamentos de comunicação entre sistemas — do modelo OSI ao HTTP/3.

## O que é

Redes de computadores permitem a comunicação entre máquinas. Protocolos são as regras que governam essa comunicação. Para um fullstack developer, os protocolos de aplicação (HTTP, WebSocket, gRPC) são o dia a dia, mas entender as camadas inferiores (TCP/UDP, DNS, TLS) é o que diferencia um senior.

## Como funciona

### Modelo OSI (simplificado para entrevistas)

| Camada | Nome | Exemplo | O que importa |
|--------|------|---------|---------------|
| 7 | Aplicação | HTTP, gRPC, WebSocket | APIs, REST, GraphQL |
| 4 | Transporte | TCP, UDP | Confiabilidade vs velocidade |
| 3 | Rede | IP | Roteamento, endereçamento |
| 2 | Enlace | Ethernet, Wi-Fi | Frames, MAC address |

### TCP vs UDP

- **TCP:** confiável, ordenado, com handshake (SYN → SYN-ACK → ACK). Usado em HTTP, banco de dados, email.
- **UDP:** sem garantia de entrega ou ordem. Mais rápido, menos overhead. Usado em DNS, streaming, jogos, VoIP.

### DNS (Domain Name System)

Traduz nomes de domínio em endereços IP. Hierárquico: root → TLD (.com) → authoritative (google.com). Cache em cada nível. TTL controla quanto tempo o resultado fica em cache.

### HTTP/HTTPS

- **HTTP/1.1:** uma requisição por conexão (ou keep-alive com pipelining limitado). Text-based.
- **HTTP/2:** multiplexing (várias requisições na mesma conexão), compressão de headers (HPACK), server push. Binary.
- **HTTP/3:** baseado em QUIC (sobre UDP), elimina head-of-line blocking do TCP.
- **HTTPS:** HTTP + TLS. Criptografia em trânsito. TLS handshake: troca de certificados, negociação de cipher suite, geração de chave simétrica.

### REST vs GraphQL vs gRPC

| Aspecto | REST | GraphQL | gRPC |
|---------|------|---------|------|
| Protocolo | HTTP | HTTP | HTTP/2 |
| Formato | JSON | JSON | Protocol Buffers |
| Contrato | OpenAPI/Swagger | Schema | .proto files |
| Use case | APIs públicas, CRUD | Clientes com necessidades variadas | Microserviços internos, alta performance |

### WebSocket

Conexão bidirecional persistente sobre TCP. Handshake HTTP → upgrade para WebSocket. Útil para: chat, notificações em tempo real, dashboards live. Alternativas: Server-Sent Events (SSE) para comunicação unidirecional servidor→cliente.

## Quando usar

- **REST:** default para APIs públicas e CRUD simples
- **GraphQL:** quando clientes diferentes precisam de dados diferentes (mobile vs web)
- **gRPC:** comunicação entre microserviços internos, quando performance importa
- **WebSocket:** comunicação em tempo real bidirecional
- **SSE:** atualizações em tempo real unidirecionais (mais simples que WebSocket)

## Armadilhas comuns

- **Ignorar latência de rede:** em system design, a rede é sempre o gargalo. Minimizar round-trips.
- **Não entender o impacto de DNS:** DNS mal configurado pode adicionar centenas de ms. DNS caching importa.
- **CORS:** entender que é uma proteção do browser, não do servidor. O servidor configura os headers.
- **Confundir REST com HTTP:** REST é um estilo arquitetural, não um protocolo. Nem toda API HTTP é RESTful.
- **WebSocket para tudo:** se só precisa de push do servidor, SSE é mais simples e funciona com HTTP/2.

## Na prática (da minha experiência)

> Em aplicações com Spring Boot, uso REST para APIs externas e gRPC para comunicação entre microserviços internos onde latência é crítica. No MedEspecialista, a comunicação entre o serviço de agendamento e o serviço de notificações usa eventos Kafka (assíncrono) em vez de chamadas HTTP síncronas, o que desacopla os serviços e melhora a resiliência.

## How to explain in English

"As a fullstack developer, I work with networking at the application layer daily. For most web applications, I default to RESTful APIs over HTTP — they're well understood, cacheable, and have great tooling. When I'm building microservices that need to communicate efficiently, I reach for gRPC because Protocol Buffers are more compact than JSON and HTTP/2 multiplexing reduces latency.

One important distinction I always keep in mind is synchronous versus asynchronous communication. For request-response patterns, HTTP works great. But for event-driven architectures — like when a user action needs to trigger multiple downstream processes — I prefer message queues like Kafka. This decouples services and improves resilience: if the notification service is temporarily down, the messages are still in the queue.

For real-time features like live dashboards or chat, I use WebSockets. But if I only need server-to-client updates — like a progress bar or live feed — Server-Sent Events is simpler and works well with existing HTTP infrastructure."

### Key vocabulary

- camada de aplicação → application layer
- handshake de três vias → three-way handshake
- balanceamento de carga → load balancing
- resolução de nomes → name resolution (DNS)
- conexão persistente → persistent connection / keep-alive
- latência → latency: tempo de ida e volta de uma requisição
- throughput → throughput: volume de dados transferidos por unidade de tempo
- certificado digital → digital certificate / SSL cert

## Recursos

- [HTTP/2 explained](https://http2-explained.haxx.se/) — guia visual
- [High Performance Browser Networking](https://hpbn.co/) — livro gratuito de Ilya Grigorik

## Veja também

- [[API Design]]
- [[System Design]]
- [[Kafka]]
