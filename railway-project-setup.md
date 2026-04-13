# Railway — Project Setup (Prompt Manager)

> **Última atualização:** Abril/2026  
> **Referência:** `railway.md` (guia geral de uso do Railway)

---

## Decisão Estratégica: Por que Railway?

O projeto atual é uma SPA React conectada diretamente ao Supabase. Para esse perfil isolado, plataformas de hosting estático como Vercel ou Cloudflare Pages seriam mais baratas e com CDN melhor.

Ainda assim, a escolha pelo Railway faz sentido pelo roadmap do produto. O projeto deve receber backend Java/Kotlin no futuro, e centralizar o stack desde agora reduz atrito operacional.

Benefícios esperados:

- **Canvas unificado** — frontend, backend e banco no mesmo projeto Railway
- **Service References** — comunicação entre serviços via variáveis gerenciadas pelo Railway
- **Preview environments por PR** — ambientes isolados por pull request
- **Rede privada** — comunicação interna via `*.railway.internal` quando houver múltiplos serviços
- **Sem migração futura** — evita mover o frontend quando o backend chegar

O custo extra em relação a um hosting estático continua sendo o tradeoff aceitável pela centralização.

---

## Estado Atual do Projeto

### Serviços existentes

| Serviço | Tecnologia | Status |
|---------|-----------|--------|
| `prompt-manager` | React 19 + Vite 8 + TypeScript | SPA, sem backend próprio |
| Banco de dados | Supabase (PostgreSQL gerenciado externo) | Ativo, fora do Railway |

### Arquitetura atual

```
Browser
  → SPA React (Railway)
      → Supabase (externo) — acesso direto via @supabase/supabase-js
```

### Arquitetura futura

```
Railway Canvas: Prompt Manager
├── frontend-react     → SPA React/Vite
├── backend-spring     → Spring Boot / Kotlin (futuro)
└── postgres           → PostgreSQL Railway (futuro, ou manter Supabase)
```

---

## Estado da Migração

### O que já está resolvido

- O frontend já é compatível com deploy como SPA no Railway.
- O projeto usa Supabase diretamente no browser, sem backend intermediário.
- O login Google já está preparado para trocar de origem entre local e produção.

O ponto-chave está em `src/contexts/AuthContext.tsx`, que usa:

```ts
redirectTo: window.location.origin
```

Isso faz o fluxo funcionar com o domínio atual em cada ambiente:

- `http://localhost:2001` em desenvolvimento
- `https://seu-app.up.railway.app` ou domínio customizado em produção

Nenhuma mudança de código é necessária para o redirecionamento OAuth por causa da migração para o Railway.

### O que ainda precisa ser feito no repositório

- Adicionar `railway.toml` na raiz
- Adicionar script `start` em `package.json`
- Publicar com as variáveis `VITE_*` definidas antes do build

### O que continua sendo manual

- Configurar o projeto e o domínio no Railway Dashboard
- Atualizar allowlists e URLs no Supabase Dashboard
- Atualizar credenciais e origens no Google Cloud Console

---

## Configuração Necessária para Deploy do Frontend

### 1. Ajustar `package.json`

O deploy passou a usar um servidor Node simples para servir `dist/` com fallback para `index.html`. Isso evita o bloqueio de host header do `vite preview` em domínios do Railway durante o healthcheck.

Adicionar em `scripts`:

```json
"start": "node server.mjs"
```

O `preview` local continua disponível separadamente com `npm run preview`.

### 2. Criar `railway.toml` na raiz do projeto

```toml
[build]
buildCommand = "npm run build"

[deploy]
startCommand = "node server.mjs"
healthcheckPath = "/"
healthcheckTimeout = 30
restartPolicyType = "on_failure"
```

Isso versiona a configuração de deploy no repositório e reduz dependência de ajustes manuais no dashboard.

### 3. Definir variáveis de ambiente no Railway

Definir no serviço do frontend em **Variables**:

| Variável | Valor |
|----------|-------|
| `VITE_SUPABASE_URL` | `https://YOUR_PROJECT_REF.supabase.co` |
| `VITE_SUPABASE_PUBLISHABLE_KEY` | `sb_publishable_your-key-here` |

> **Importante:** em Vite, variáveis `VITE_*` são embutidas em **build time**. No caso desta SPA, isso significa que as variáveis precisam estar disponíveis antes do build do Railway. Se mudar qualquer `VITE_*`, é necessário fazer novo deploy.

### 4. Porta do serviço no Railway

Ao gerar o domínio público, configurar a porta alvo como `8080`.

O `server.mjs` lê `PORT` quando ela existir e usa `8080` como fallback local.

### 5. Gerar domínio público

Após o primeiro deploy: **Service > Settings > Networking > Generate Domain**.

---

## Google OAuth no Railway

## Como o fluxo funciona

1. O usuário clica em **Entrar com Google** no app hospedado no Railway.
2. O Supabase redireciona o usuário para o Google.
3. O Google autentica e redireciona para o callback do Supabase.
4. O Supabase redireciona o usuário de volta para o frontend.

Fluxo resumido:

```
App no Railway
  → Supabase Auth
      → Google OAuth
          → https://fwkpzfmguweykwkpqald.supabase.co/auth/v1/callback
              → volta para o domínio do app
```

O callback do Google continua no Supabase. O que muda com a migração é a URL final do frontend para a qual o Supabase devolve o usuário.

### O que não muda

- O callback OAuth do Google continua sendo o callback do Supabase.
- O código do frontend não precisa mudar para `redirectTo`.

### O que precisa ser configurado

#### Supabase Dashboard → Authentication → URL Configuration

- Atualizar o **Site URL** para o domínio de produção no Railway
- Adicionar o domínio Railway em **Redirect URLs**
- Manter também a URL local de desenvolvimento, por exemplo `http://localhost:2001/**`

Exemplos:

```text
Site URL:
https://seu-app.up.railway.app

Redirect URLs:
http://localhost:2001/**
https://seu-app.up.railway.app/**
```

Se forem usados preview environments por PR, adicionar também o padrão correspondente ao Railway em `Redirect URLs`.

#### Google Cloud Console → OAuth Client

- Adicionar o domínio Railway em **Authorized JavaScript origins**
- Manter o callback do Supabase em **Authorized redirect URIs**

Exemplo:

```text
Authorized JavaScript origins:
http://localhost:2001
https://seu-app.up.railway.app

Authorized redirect URIs:
https://fwkpzfmguweykwkpqald.supabase.co/auth/v1/callback
```

O domínio Railway nao substitui o redirect URI do Supabase.

### Conclusão prática sobre OAuth

Não há impeditivo técnico para publicar no Railway com Google OAuth. A migração exige configuração manual de dashboard, não refatoração do fluxo de autenticação.

---

## SPA Routing

O projeto usa React Router com rotas client-side, por exemplo `/prompts/:slug`.

Se alguém acessar uma rota diretamente, o servidor precisa devolver `index.html` em vez de `404`. O `server.mjs` implementa esse fallback para manter o React Router funcionando em produção.

---

## Armadilhas Relevantes

**Aplicação não responde após deploy:**

```bash
# ERRADO
npx vite preview --port 8080 --host 0.0.0.0

# CORRETO
node server.mjs
```

Motivo: o `vite preview` responde `403 Forbidden` para host headers do domínio Railway durante o healthcheck.

**Variáveis Supabase não funcionam em produção:**

- Variáveis `VITE_*` precisam estar definidas antes do build
- Alterações em `VITE_*` exigem novo deploy
- Configurar as variáveis antes do primeiro deploy evita build inválido

**Google OAuth falha após publicar:**

- O domínio Railway precisa estar em `Redirect URLs` no Supabase
- O domínio Railway precisa estar em `Authorized JavaScript origins` no Google Cloud
- O callback do Supabase precisa continuar cadastrado em `Authorized redirect URIs`

**Preview deploy quebra login:**

- Se preview por PR for usado, a URL de preview também precisa estar permitida no Supabase Auth
- Em produção, prefira URL exata; para previews, pode ser necessário usar wildcard

**CDN global ausente:**

- O Railway não oferece edge global como plataformas focadas em frontend estático
- Se latência global virar requisito, colocar Cloudflare na frente do domínio Railway é uma opção simples

---

## Checklist de Deploy

**Antes do primeiro deploy:**

- [ ] Criar projeto no [railway.com](https://railway.com)
- [ ] Conectar repositório GitHub ao serviço
- [ ] Adicionar `railway.toml` na raiz do projeto
- [ ] Adicionar script `start` apontando para `node server.mjs`
- [ ] Confirmar que o domínio público do serviço aponta para a porta `8080`
- [ ] Definir `VITE_SUPABASE_URL` nas variáveis do serviço
- [ ] Definir `VITE_SUPABASE_PUBLISHABLE_KEY` nas variáveis do serviço
- [ ] Confirmar que as variáveis foram definidas antes do build inicial

**Após o primeiro deploy:**

- [ ] Gerar domínio público em **Settings > Networking > Generate Domain**
- [ ] Atualizar o **Site URL** no Supabase para o domínio Railway de produção
- [ ] Adicionar o domínio Railway em **Redirect URLs** no Supabase
- [ ] Adicionar o domínio Railway em **Authorized JavaScript origins** no Google Cloud
- [ ] Confirmar que `https://fwkpzfmguweykwkpqald.supabase.co/auth/v1/callback` permanece em **Authorized redirect URIs**
- [ ] Verificar que a aplicação carrega e conecta ao Supabase
- [ ] Verificar que o login com Google conclui e retorna ao app
- [ ] Verificar logs com `railway logs`

**Se preview environments por PR forem habilitados:**

- [ ] Identificar o padrão de URL gerado pelo Railway
- [ ] Adicionar esse padrão em `Redirect URLs` no Supabase Auth
- [ ] Validar login OAuth em um ambiente preview

**Quando o backend for adicionado:**

- [ ] Criar novo serviço no mesmo Railway Project
- [ ] Configurar `server.port=${PORT:8080}` no backend
- [ ] Adicionar `VITE_API_URL=https://${{backend.RAILWAY_PUBLIC_DOMAIN}}` no frontend
- [ ] Configurar healthcheck no backend (`/actuator/health` para Spring Boot)
- [ ] Ativar rede privada para comunicação interna entre serviços

---

## Referências

- [Guia geral Railway](./railway.md)
- [Documentação oficial Railway](https://docs.railway.com)
- [Guia Deploy Frontend no Railway](https://docs.railway.com/guides/nodejs)
- [Supabase Auth — Redirect URLs](https://supabase.com/docs/guides/auth/redirect-urls)
- [Supabase Auth — Login com Google](https://supabase.com/docs/guides/auth/social-login/auth-google)
- [Config as Code Reference](https://docs.railway.com/config-as-code/reference)
