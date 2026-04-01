---
title: "Linux"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - infraestrutura
  - entrevista
publish: false
---

# Linux

Sistema operacional que roda 90%+ dos servidores — fundamentos que todo dev precisa.

## O que é

Linux é o kernel open-source criado por Linus Torvalds (1991). Distribuições (Ubuntu, Debian, Alpine, Amazon Linux) adicionam userspace tools. Para devs fullstack, dominar Linux é essencial — seus servidores, containers e CI/CD rodam nele.

## Conceitos essenciais

### Filesystem

```text
/           → raiz
/home       → diretórios dos usuários
/etc        → configuração do sistema
/var        → dados variáveis (logs, caches)
/tmp        → temporários
/usr        → programas do sistema
/opt        → software de terceiros
```

### Permissões

```bash
-rwxr-xr--  →  user: rwx | group: r-x | others: r--
chmod 755 script.sh    # rwxr-xr-x
chmod +x script.sh     # adicionar execução
chown user:group file  # mudar dono
```

### Comandos essenciais para devs

```bash
# Navegação e arquivos
ls -la                  # listar com detalhes
find . -name "*.log"    # buscar arquivos
grep -r "error" /var/log  # buscar em conteúdo

# Processos
ps aux                  # listar processos
top / htop              # monitorar recursos
kill -9 PID             # matar processo
lsof -i :8080           # quem usa a porta 8080

# Rede
curl -v http://localhost:8080  # testar HTTP
ss -tlnp                # portas abertas
ping / traceroute       # diagnóstico de rede

# Disco
df -h                   # espaço em disco
du -sh *                # tamanho de diretórios

# Logs
tail -f /var/log/app.log  # acompanhar log em tempo real
journalctl -u service     # logs do systemd
```

### Systemd

```bash
sudo systemctl start nginx     # iniciar serviço
sudo systemctl enable nginx    # iniciar no boot
sudo systemctl status nginx    # ver status
sudo journalctl -u nginx -f   # ver logs
```

### SSH

```bash
ssh user@server               # conectar
ssh-keygen -t ed25519         # gerar chave
ssh-copy-id user@server       # copiar chave pública
scp file.txt user@server:/path  # copiar arquivo
```

## Quando usar

- **Servidores:** praticamente todos rodam Linux
- **Containers:** images Docker são baseadas em Linux (Alpine, Debian)
- **Desenvolvimento local:** WSL2, Ubuntu nativo, ou macOS (Unix-like)
- **CI/CD:** runners geralmente são Linux

## How to explain in English

"Linux is the operating system that powers my development and deployment environment. I'm comfortable with the command line for file management, process monitoring, network debugging, and log analysis. I use Ubuntu as my daily driver and Alpine for Docker images. Understanding Linux fundamentals — permissions, processes, networking — is essential for debugging production issues."

### Key vocabulary

- sistema de arquivos → filesystem
- permissões → permissions (rwx)
- processo → process
- serviço → service / daemon
- kernel → kernel: núcleo do OS

## Veja também

- [[Terminal]]
- [[Docker]]
- [[WSL, Docker e Kubernetes]]
