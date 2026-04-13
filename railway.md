# Railway — Guia de Uso (TypeScript + Kotlin + Java)

> **Fonte oficial:** https://docs.railway.com  
> **Última atualização do guia:** Abril/2026

---

## 1. Introdução

### O que é o Railway?

Railway é uma plataforma de cloud inteligente (PaaS — Platform as a Service) que simplifica o provisionamento de infraestrutura, o desenvolvimento local e o deploy em produção. Diferente de provedores tradicionais como AWS ou GCP, o Railway abstrai a complexidade da infraestrutura e permite que você foque no código.

**Filosofia:** "Your infrastructure should be as easy to use as the code you write."

### Problema que resolve

| Problema | Como o Railway resolve |
|----------|----------------------|
| Configurar servidores manualmente | Deploy automático via Git push |
| Gerenciar bancos de dados separadamente | Bancos integrados com 1 clique |
| Ambientes de preview caros | Preview environments automáticos por PR |
| Observabilidade fragmentada | Logs, métricas e webhooks integrados |
| Deploy de backends Java/Kotlin complexo | Detecção automática de Gradle/Maven |

### Quando faz sentido aplicar

- ✅ Projetos **fullstack** com frontend TypeScript + backend JVM (Java/Kotlin)
- ✅ APIs REST/GraphQL com **conexões persistentes** (WebSockets, SSE)
- ✅ Projetos que precisam de banco de dados gerenciado junto ao app
- ✅ Times pequenos/médios que querem **velocidade de entrega sem ops**
- ✅ Side projects e MVPs que precisam escalar depois
- ❌ Aplicações que já exigem Kubernetes customizado ou compliance enterprise específico

---

## 2. Conceitos Fundamentais

### Estrutura do Railway

```
Workspace
└── Project (conjunto de serviços)
    ├── Service A (backend Spring Boot)
    ├── Service B (frontend React/Next.js)
    ├── Service C (PostgreSQL)
    └── Service D (Redis)
```

- **Workspace:** Sua conta ou organização
- **Project (Canvas):** Agrupa todos os serviços relacionados — frontend, backend, banco, workers — em uma visão unificada
- **Service:** Uma unidade deployável (app, banco, cron job)
- **Environment:** Ambientes paralelos (production, staging, preview por PR)

### Modelo de Infraestrutura

Railway usa **hardware próprio** (não AWS), o que traz:
- Sem cold starts (servidores long-running)
- Conexões persistentes suportadas (WebSockets, SSE, chat em tempo real)
- Escalamento vertical + horizontal com réplicas
- Sem limite de tempo de execução (diferente do Vercel com 800s)

### Formas de Deploy

| Método | Quando usar |
|--------|------------|
| **GitHub** (recomendado) | Push automático, preview PRs |
| **CLI (`railway up`)** | Deploy local rápido, CI/CD customizado |
| **Docker Image** | Controle total da build |
| **Template** | Projetos pré-configurados com 1 clique |

---

## 3. Instalação e Configuração Inicial

### Pré-requisitos

- Conta no [railway.com](https://railway.com)
- Node.js instalado (para CLI)
- Conta no GitHub (para deploy automático)

### Instalando a CLI

```bash
# via npm
npm install -g @railway/cli

# via Homebrew (macOS/Linux)
brew install railway

# Autenticar
railway login
```

### Criando um projeto

```bash
# Inicializar novo projeto na pasta atual
railway init

# Verificar status do projeto vinculado
railway status

# Abrir o projeto no browser
railway open
```

### Variáveis de Ambiente

```bash
# Listar variáveis
railway variables

# Definir uma variável
railway variables --set "DATABASE_URL=postgres://..."

# Rodar localmente com as variáveis do Railway
railway run ./gradlew bootRun
railway run npm run dev
```

---

## 4. Deploy: Frontend TypeScript

### React / Vite / Angular

Railway detecta automaticamente projetos Node.js com `package.json`.

**Estrutura mínima esperada:**
```json
// package.json
{
  "scripts": {
    "build": "vite build",
    "start": "vite preview --port $PORT --host 0.0.0.0"
  }
}
```

> ⚠️ **Importante:** Sempre use `$PORT` (variável injetada pelo Railway) e bind em `0.0.0.0`.

**Deploy via CLI:**
```bash
cd meu-projeto-frontend
railway init
railway up
```

**Deploy via GitHub:**
1. Acesse [railway.com/new](https://railway.com/new)
2. Selecione **Deploy from GitHub repo**
3. Escolha o repositório
4. Clique em **Deploy Now**
5. Vá em **Settings > Networking > Generate Domain**

### Next.js

```bash
# next.config.js — Não precisa de configuração especial
# Railway detecta automaticamente e roda: next build && next start
```

### Railway.toml (Config as Code)

Arquivo opcional para customizar o comportamento:

```toml
# railway.toml — na raiz do projeto
[build]
builder = "nixpacks"
buildCommand = "npm run build"

[deploy]
startCommand = "npm start"
healthcheckPath = "/"
healthcheckTimeout = 30
restartPolicyType = "on_failure"
```

---

## 5. Deploy: Backend Java (Spring Boot)

Railway detecta automaticamente projetos **Maven** e **Gradle** via [Railpack](https://railpack.com/languages/java).

### Spring Boot — Via GitHub (Recomendado)

```bash
# Não precisa de configuração especial para Maven
# Railway executa automaticamente: ./mvnw package && java -jar target/*.jar
```

**Configuração da porta (obrigatório):**
```yaml
# application.properties
server.port=${PORT:8080}
```

```yaml
# application.yml
server:
  port: ${PORT:8080}
```

### Spring Boot — Via Dockerfile

Para projetos complexos ou com dependências específicas:

```dockerfile
# Dockerfile
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app
COPY .mvn/ .mvn
COPY mvnw pom.xml ./
RUN ./mvnw dependency:go-offline
COPY src ./src
RUN ./mvnw package -DskipTests

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### railway.toml para Spring Boot

```toml
[build]
buildCommand = "./mvnw package -DskipTests"

[deploy]
startCommand = "java -jar target/*.jar"
healthcheckPath = "/actuator/health"
healthcheckTimeout = 60
```

---

## 6. Deploy: Backend Kotlin (Ktor)

### Ktor — Configuração Obrigatória

O Ktor precisa escutar em `0.0.0.0` e usar a variável `PORT`:

```hocon
# src/main/resources/application.conf
ktor {
    deployment {
        port = 8080
        port = ${?PORT}
        host = "0.0.0.0"
    }
    application {
        modules = [ com.example.ApplicationKt.module ]
    }
}
```

Ou via código:

```kotlin
// Application.kt
fun main() {
    val port = System.getenv("PORT")?.toInt() ?: 8080
    embeddedServer(Netty, port = port, host = "0.0.0.0") {
        module()
    }.start(wait = true)
}
```

### Ktor — Via GitHub

Railway usa Railpack para detectar projetos Gradle/Kotlin automaticamente.

```kotlin
// build.gradle.kts — configuração mínima
plugins {
    kotlin("jvm") version "2.0.0"
    id("io.ktor.plugin") version "2.3.12"
    application
}

application {
    mainClass.set("com.example.ApplicationKt")
}

tasks.jar {
    manifest {
        attributes["Main-Class"] = "com.example.ApplicationKt"
    }
}
```

### Ktor — Via Dockerfile

```dockerfile
# Dockerfile
FROM gradle:8.5-jdk21-alpine AS builder
WORKDIR /app
COPY --chown=gradle:gradle . .
RUN gradle shadowJar --no-daemon

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=builder /app/build/libs/*-all.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## 7. Banco de Dados Integrado

Railway oferece bancos com **1 clique** dentro do mesmo projeto:

| Banco | Tipo | Casos de uso |
|-------|------|-------------|
| PostgreSQL | Relacional | APIs REST, dados estruturados |
| MySQL | Relacional | Alternativa ao Postgres |
| Redis | Key-value | Cache, sessões, filas |
| MongoDB | Documento | Dados sem esquema fixo |

### Adicionando PostgreSQL

1. No Canvas do projeto → **+ New Service**
2. Selecione **PostgreSQL**
3. A variável `DATABASE_URL` é injetada automaticamente nos outros serviços

### Usando no Spring Boot

```yaml
# application.yml
spring:
  datasource:
    url: ${DATABASE_URL}
  jpa:
    hibernate:
      ddl-auto: validate
```

### Usando no Ktor (Exposed)

```kotlin
// Database.kt
fun initDatabase() {
    val databaseUrl = System.getenv("DATABASE_URL")
        ?: "jdbc:postgresql://localhost:5432/mydb"
    
    Database.connect(databaseUrl, driver = "org.postgresql.Driver")
}
```

### Rede Privada entre Serviços

Serviços dentro do mesmo projeto Railway se comunicam via **rede privada** (sem passar pela internet):

```
# Variável automática gerada pelo Railway
POSTGRES_URL=postgresql://postgres.railway.internal:5432/railway

# Use internamente em vez da URL pública
DATABASE_URL=${POSTGRES_URL}
```

---

## 8. Projeto Fullstack: Frontend TS + Backend JVM

### Estrutura de um Projeto Completo no Railway Canvas

```
Meu App [Project]
├── 🌐 frontend-react     → GitHub repo (TypeScript/React)
├── ⚙️  backend-spring     → GitHub repo (Java/Spring Boot)
├── 🗄️  postgres           → Serviço PostgreSQL
└── ⚡  redis              → Serviço Redis
```

### Comunicação entre serviços

```typescript
// frontend (TypeScript) — usa URL pública do backend
const API_URL = process.env.NEXT_PUBLIC_API_URL || 'https://backend.up.railway.app';

async function fetchData() {
  const res = await fetch(`${API_URL}/api/users`);
  return res.json();
}
```

```kotlin
// backend Kotlin — lê URL do banco pela variável de ambiente
val dbUrl = System.getenv("DATABASE_URL")
val redisUrl = System.getenv("REDIS_URL")
```

### Variáveis de Referência (Service References)

No Railway, você pode referenciar variáveis de outros serviços:

```
# No serviço frontend, referencie a URL do backend:
NEXT_PUBLIC_API_URL=${{backend-spring.RAILWAY_PUBLIC_DOMAIN}}

# No serviço backend, referencie a URL do banco:
DATABASE_URL=${{postgres.DATABASE_URL}}
```

---

## 9. Ambientes e Preview Deployments

### Environments

```
Production  → branch main
Staging     → branch staging
Preview     → por Pull Request (automático)
```

Cada ambiente tem suas próprias variáveis, bancos e serviços **isolados**.

### Configurando Preview por PR

1. Conecte o repo ao GitHub
2. Habilite **GitHub Autodeploys** nas configurações do serviço
3. Cada PR gera automaticamente um ambiente preview com URL própria

---

## 10. Escalamento

### Vertical (automático)

Railway ajusta CPU e memória automaticamente conforme a carga.

### Horizontal (réplicas)

```toml
# railway.toml
[deploy]
numReplicas = 3
```

Ou via dashboard: **Service > Settings > Replicas**

### Serverless Mode

Para workloads esporádicos, ative o modo serverless:
- Serviço hiberna após 10 min sem requisições
- Acorda automaticamente na próxima requisição
- Sem cobrança durante o sono

> ⚠️ **Não recomendado** para APIs de baixa latência (ex.: backends JVM com conexão DB persistente)

---

## 11. Monitoramento e Observabilidade

### Logs

```bash
# CLI
railway logs

# Com filtro (últimas 100 linhas)
railway logs --tail 100
```

### Métricas Disponíveis

- CPU usage
- Memory usage
- Network I/O
- Deployment history

### Healthchecks

```toml
# railway.toml
[deploy]
healthcheckPath = "/health"
healthcheckTimeout = 30  # segundos
```

```kotlin
// Ktor — endpoint de health
routing {
    get("/health") {
        call.respond(mapOf("status" to "UP"))
    }
}
```

```java
// Spring Boot — com Actuator
// application.properties
management.endpoints.web.exposure.include=health
management.endpoint.health.show-details=always
```

---

## 12. Boas Práticas

### Configuração Geral

- ✅ **Sempre** use a variável `$PORT` e bind em `0.0.0.0`
- ✅ Configure `healthcheckPath` para deploys confiáveis
- ✅ Use **Service References** para conectar serviços (evita hardcode de URLs)
- ✅ Separe segredos por ambiente (Production vs Staging)
- ✅ Use `railway.toml` para versionar configurações de deploy

### Para JVM (Java/Kotlin)

```toml
# railway.toml — Spring Boot/Ktor
[deploy]
# Aumenta timeout do healthcheck para JVM warmup
healthcheckTimeout = 60

# Para Spring Boot com muitas dependências
startCommand = "java -Xmx512m -jar target/app.jar"
```

- ✅ Configure `-Xmx` para controlar memória da JVM
- ✅ Use GraalVM Native Image para reduzir tempo de startup (opcional)
- ✅ Ative Spring Actuator para healthchecks robustos
- ✅ Use Dockerfile para controle total da versão do JDK

### Para TypeScript/Frontend

- ✅ Use variáveis de ambiente com prefixo `NEXT_PUBLIC_` para expor ao cliente (Next.js)
- ✅ Configure `output: 'standalone'` no `next.config.js` para otimizar imagem Docker
- ✅ Gerencie assets estáticos via CDN/Storage Bucket para produção

---

## 13. Limitações e Armadilhas

### Limitações Técnicas

| Limitação | Detalhe | Solução |
|-----------|---------|---------|
| Sem CDN nativo | Railway não tem edge network global como Vercel | Use Cloudflare na frente |
| Região limitada | Menos regiões que AWS/GCP | Escolha a mais próxima do usuário |
| Plano Free restrito | 0.5 GB RAM, 1 vCPU | Use apenas para testes |
| Storage em Volume | Volumes não replicam entre réplicas | Use S3-compatible ou Storage Bucket |
| Build lento para JVM | Maven/Gradle pode demorar no primeiro build | Use cache de dependências ou Dockerfile |

### Armadilhas Comuns

**❌ Erro: Aplicação não responde**
```
# ERRADO: bind apenas em localhost
server.port=8080
# (sem especificar host)

# CORRETO: bind em 0.0.0.0 + PORT do ambiente
server.port=${PORT:8080}
server.address=0.0.0.0
```

**❌ Erro: Banco não conecta entre serviços**
```
# ERRADO: usar URL pública internamente
DATABASE_URL=postgresql://postgres.up.railway.app:5432/railway

# CORRETO: usar rede privada
DATABASE_URL=postgresql://postgres.railway.internal:5432/railway
```

**❌ Erro: Ktor não lê PORT**
```kotlin
// ERRADO
embeddedServer(Netty, port = 8080).start()

// CORRETO
val port = System.getenv("PORT")?.toInt() ?: 8080
embeddedServer(Netty, port = port, host = "0.0.0.0").start(wait = true)
```

**❌ Cold start em modo serverless com JVM**
- JVMs têm startup lento (2-10s)
- Não use serverless mode para backends Java/Kotlin de baixa latência
- Use sempre-ativo (padrão) ou considere GraalVM Native Image

**❌ Variáveis de ambiente não disponíveis no build time**
- Variáveis do Railway são injetadas em **runtime**, não em build time
- Para Next.js, use `NEXT_PUBLIC_` apenas para valores que podem ser expostos

---

## 14. Railway vs. Vercel — Comparativo Completo

### Visão Geral

| Feature | Railway | Vercel |
|---------|---------|--------|
| **Infraestrutura** | Hardware próprio, servidores long-running | AWS, funções serverless |
| **Ideal para** | Fullstack, backends, real-time, JVM | Frontend-first, SPAs, SSR simples |
| **Suporte a Docker** | ✅ Sim | ❌ Não |
| **Conexões persistentes** | ✅ WebSockets, SSE, TCP | ❌ Sem suporte |
| **Cold starts** | ❌ Sem cold starts | ⚠️ Possível (com otimizações) |
| **Limite de memória** | Até capacidade total da máquina | 4 GB por função |
| **Limite de tempo** | Sem limite (processo contínuo) | 800 segundos (~13 min) |
| **Estrutura de projeto** | 1 projeto = N serviços + bancos | 1 serviço por projeto |
| **Bancos de dados** | Integrados com 1 clique | Via marketplace (providers externos) |

### Modelo de Preços

| Plano | Railway | Vercel |
|-------|---------|--------|
| **Free** | $0/mês ($1 crédito de uso) | $0/mês (limites rígidos) |
| **Hobby/Pro** | $5/mês (inclui $5 uso) | $20/mês |
| **Pro** | $20/mês (inclui $20 uso) | $20/mês + uso |
| **Cobrança de uso** | CPU + RAM por minuto de uso real | CPU ativo + memória provisionada + invocações |

**Railway pricing formula:**
```
Custo = (CPU vCPU × $20/mês) + (RAM GB × $10/mês) + (Egress GB × $0.05)
```

### Para o seu Stack Específico

| Cenário | Railway | Vercel |
|---------|---------|--------|
| **React/Angular (SPA)** | ✅ Funciona | ✅ Melhor DX para frontend puro |
| **Next.js (SSR)** | ✅ Funciona bem | ✅ Integração nativa (mesma empresa) |
| **Spring Boot (Java)** | ✅ Suporte oficial, detecção automática | ❌ Não suportado |
| **Ktor (Kotlin)** | ✅ Suporte oficial, detecção automática | ❌ Não suportado |
| **WebSockets** | ✅ Nativo | ❌ Sem suporte |
| **Jobs agendados (Cron)** | ✅ Nativo | ⚠️ Limitado (via vercel.json) |
| **Banco de dados gerenciado** | ✅ PostgreSQL, MySQL, Redis (integrado) | ⚠️ Via marketplace externo |
| **Projeto fullstack unificado** | ✅ Todos os serviços no mesmo Canvas | ❌ Projetos separados |

### Decisão para seu caso

```
Você usa Java/Kotlin no backend?
├── SIM → Railway (Vercel não suporta JVM)
│
Você precisa de WebSockets/conexões persistentes?
├── SIM → Railway
│
Você precisa de banco de dados junto ao app?
├── SIM → Railway (mais simples e integrado)
│
Frontend TypeScript puro (sem backend próprio)?
├── SIM + prioridade em DX de frontend → Vercel
├── SIM + backend JVM no mesmo projeto → Railway
```

**Conclusão para seu stack:** Use **Railway** para tudo. Ele suporta TypeScript no frontend E Java/Kotlin no backend, num único projeto unificado. A Vercel não suporta JVM de forma nativa.

---

## 15. Quando Usar e Quando Evitar

### ✅ Use Railway quando:

- Backend em Java (Spring Boot) ou Kotlin (Ktor, Spring)
- Precisa de WebSockets, SSE ou conexões TCP persistentes
- Quer banco de dados (Postgres, Redis) gerenciado no mesmo projeto
- Precisa de jobs agendados (Cron)
- Quer todo o stack (frontend + backend + banco) em 1 lugar
- ETL, processamento de mídia, tarefas de longa duração

### ❌ Evite Railway quando:

- Seu frontend é estático puro e você quer o melhor CDN global (use Vercel ou Cloudflare Pages)
- Precisa de latência de edge computing global (use Cloudflare Workers)
- Tem requisitos de compliance muito específicos (SOC2, HIPAA) → use Enterprise
- Precisa de features avançadas de observabilidade APM (use Datadog + Railway integrado)

---

## 16. Alternativas Modernas

| Plataforma | Foco | Prós | Contras |
|-----------|------|------|---------|
| **Vercel** | Frontend/SSR | DX excepcional, Next.js nativo, CDN global | Sem suporte JVM, sem conexões persistentes |
| **Render** | Similar ao Railway | Bom para Node/Python | Suporte JVM limitado, mais caro |
| **Fly.io** | Docker/Edge | Deploy global, controle fino | Curva de aprendizado maior |
| **Heroku** | PaaS clássico | Simples, muitos addons | Caro, dynos com cold start no plano free |
| **AWS Elastic Beanstalk** | PaaS na AWS | Integração AWS nativa | Complexidade, custo, configuração manual |
| **Google Cloud Run** | Containers serverless | Auto-scale para zero, paga por uso | Cold starts em JVM, configuração necessária |
| **Coolify** | Self-hosted | Gratuito, controle total | Precisa gerenciar servidor próprio |

### Comparativo rápido para JVM

```
Facilidade de uso:    Railway > Heroku > Render > Fly.io > AWS/GCP
Custo (pequenos apps): Railway ≈ Render < Fly.io < Heroku < AWS
Controle/flexibilidade: AWS/GCP > Fly.io > Railway > Heroku
Suporte JVM nativo:   Railway ✅ | Render ✅ | Fly.io ✅ | Vercel ❌
```

---

## 17. Resumo Final

### O que o Railway resolve para você

| Você quer | Railway oferece |
|-----------|----------------|
| Deploy de Spring Boot sem ops | ✅ Detecção automática Maven/Gradle |
| Deploy de Ktor sem ops | ✅ Detecção automática Gradle/Kotlin |
| Deploy de React/Angular/Next.js | ✅ Detecção automática Node.js |
| Banco de dados junto ao app | ✅ PostgreSQL, Redis, MySQL com 1 clique |
| Ambientes de preview por PR | ✅ Automático com GitHub |
| Logs e métricas integrados | ✅ Dashboard de observabilidade |
| Variáveis de ambiente seguras | ✅ Por ambiente, referências entre serviços |
| Escalar quando precisar | ✅ Vertical automático + horizontal manual |

### Guia Rápido de Comandos CLI

```bash
# Setup inicial
npm install -g @railway/cli
railway login
railway init                    # criar projeto
railway link                    # vincular a projeto existente

# Deploy
railway up                      # deploy do diretório atual
railway up --detach             # deploy sem aguardar logs

# Variáveis
railway variables               # listar
railway variables --set KEY=val # definir

# Desenvolvimento local
railway run ./gradlew bootRun   # Spring Boot com env do Railway
railway run npm run dev         # Frontend com env do Railway

# Logs e status
railway logs                    # logs em tempo real
railway status                  # status do projeto
railway open                    # abrir no browser
```

### Checklist de Deploy (JVM + TypeScript)

**Backend Java/Kotlin:**
- [ ] Configurar `server.port=${PORT:8080}` e bind em `0.0.0.0`
- [ ] Adicionar healthcheck endpoint (`/health` ou `/actuator/health`)
- [ ] Configurar `railway.toml` com `healthcheckTimeout = 60`
- [ ] Usar variáveis de ambiente para todas as conexões (DB, Redis)
- [ ] Testar localmente com `railway run`

**Frontend TypeScript:**
- [ ] Script `start` usando `$PORT` e `--host 0.0.0.0`
- [ ] Variáveis de API URL como env vars (não hardcoded)
- [ ] Build otimizado para produção configurado
- [ ] Gerar domínio público após primeiro deploy

**Projeto Fullstack:**
- [ ] Serviços no mesmo Railway Project (Canvas)
- [ ] Service References para URLs entre serviços
- [ ] Ambientes separados (production, staging)
- [ ] Healthchecks configurados em todos os serviços

---

## Referências

- [Documentação oficial Railway](https://docs.railway.com)
- [Railway vs Vercel (oficial)](https://docs.railway.com/platform/compare-to-vercel)
- [Guia Spring Boot no Railway](https://docs.railway.com/guides/spring-boot)
- [Guia Ktor no Railway](https://docs.railway.com/guides/ktor)
- [Planos e Preços](https://docs.railway.com/pricing/plans)
- [Config as Code Reference](https://docs.railway.com/config-as-code/reference)
- [Templates Railway](https://railway.com/templates)
