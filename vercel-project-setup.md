# Vercel — Project Setup (Prompt Manager)

> **Última atualização:** Abril/2026  
> **Referência:** `vercel.md` (guia geral de uso da Vercel)

---

## Decisão Estratégica: Por que Vercel?

O projeto atual é uma SPA React conectada diretamente ao Supabase, sem backend próprio. Para esse perfil, a Vercel é uma das escolhas mais diretas: deploy de frontend estático com CDN global, zero configuração para Vite/React, e integração nativa com GitHub.

Se o roadmap incluir um backend Java/Kotlin, o stack precisará ser revisado — a Vercel não suporta JVM. Nesse caso, a alternativa natural é migrar para o Railway (que unifica frontend + backend + banco no mesmo canvas) ou manter o frontend na Vercel e adicionar o backend em outro serviço.

Benefícios esperados para o estado atual do projeto:

- **CDN global** — assets entregues da edge mais próxima do usuário
- **Preview deployments automáticos por PR** — cada pull request gera uma URL de preview isolada
- **Deploy zero-config** — a Vercel detecta Vite automaticamente, sem configurar nada
- **SSL automático** — certificados gerenciados sem intervenção manual
- **Plano Hobby generoso** — adequado para projetos não comerciais

O tradeoff é que, se o backend JVM chegar, o frontend precisará ser movido ou a comunicação entre plataformas diferentes precisará ser gerenciada.

---

## Estado Atual do Projeto

### Serviços existentes

| Serviço | Tecnologia | Status |
|---------|-----------|--------|
| `prompt-manager` | React 19 + Vite 8 + TypeScript | SPA, sem backend próprio |
| Banco de dados | Supabase (PostgreSQL gerenciado externo) | Ativo, fora da Vercel |

### Arquitetura atual

```
Browser
  → SPA React (Vercel CDN)
      → Supabase (externo) — acesso direto via @supabase/supabase-js
```

### Arquitetura futura (se backend for adicionado)

```
Vercel
└── frontend-react     → SPA React/Vite

Serviço externo (Railway, Render, etc.)
└── backend-spring     → Spring Boot / Kotlin

Supabase (externo)
└── postgres           → PostgreSQL gerenciado
```

> Se o backend JVM for adicionado, avaliar se migrar o frontend para o Railway faz mais sentido (stack unificado) do que manter comunicação entre Vercel + outra plataforma.

---

## Estado da Migração

### O que já está resolvido

- O frontend é uma SPA React com Vite, que a Vercel detecta e faz build automaticamente.
- O projeto usa Supabase diretamente no browser, sem backend intermediário.
- O login Google já está preparado para trocar de origem entre local e produção.

O ponto-chave está em `src/contexts/AuthContext.tsx`, que usa:

```ts
redirectTo: window.location.origin
```

Isso faz o fluxo funcionar com o domínio atual em cada ambiente:

- `http://localhost:2001` em desenvolvimento
- `https://seu-app.vercel.app` ou domínio customizado em produção

Nenhuma mudança de código é necessária para o redirecionamento OAuth por causa da migração para a Vercel.

### O que ainda precisa ser feito no repositório

- Adicionar `vercel.json` na raiz (para SPA routing)
- Confirmar que o `package.json` tem os scripts corretos (`build`, `preview`)

### O que continua sendo manual

- Criar o projeto e conectar o repositório no painel da Vercel
- Configurar as variáveis de ambiente no painel da Vercel
- Atualizar allowlists e URLs no Supabase Dashboard
- Atualizar credenciais e origens no Google Cloud Console

---

## Configuração Necessária para Deploy do Frontend

### 1. Criar `vercel.json` na raiz do projeto

A Vercel serve arquivos estáticos por padrão, mas uma SPA com React Router precisa que todas as rotas retornem `index.html`. Sem isso, acessar `/prompts/meu-slug` diretamente resulta em `404`.

```json
{
  "rewrites": [
    { "source": "/(.*)", "destination": "/index.html" }
  ]
}
```

Isso substitui qualquer necessidade de servidor Node customizado (como o `server.mjs` usado no Railway).

### 2. Verificar `package.json`

A Vercel detecta Vite automaticamente e executa `npm run build`. Confirmar que os scripts existem:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  }
}
```

Nenhum script `start` é necessário — a Vercel serve os assets estáticos diretamente do CDN após o build.

### 3. Definir variáveis de ambiente no painel da Vercel

Acessar o projeto → **Settings > Environment Variables** e configurar:

| Variável | Valor | Ambiente |
|----------|-------|----------|
| `VITE_SUPABASE_URL` | `https://YOUR_PROJECT_REF.supabase.co` | Production, Preview, Development |
| `VITE_SUPABASE_PUBLISHABLE_KEY` | `sb_publishable_your-key-here` | Production, Preview, Development |

> **Importante:** em Vite, variáveis `VITE_*` são embutidas em **build time**. Isso significa que as variáveis precisam estar definidas no painel da Vercel **antes** do build. Se alterar qualquer `VITE_*`, um novo deploy (redeploy) é necessário para que os novos valores entrem em vigor.

**Puxar variáveis para desenvolvimento local:**

```bash
vercel env pull .env.local
```

Isso sincroniza as variáveis do ambiente Development da Vercel para o `.env.local` local.

### 4. Conectar repositório e fazer deploy

**Via painel:**
1. Acesse [vercel.com/new](https://vercel.com/new)
2. Selecione **Import Git Repository**
3. Escolha o repositório `prompt-manager`
4. A Vercel detecta Vite automaticamente — nenhuma configuração adicional necessária
5. Confirme que as variáveis de ambiente estão definidas antes de clicar em **Deploy**

**Via CLI:**
```bash
npm i -g vercel
vercel login
vercel        # preview deploy
vercel --prod # deploy em produção
```

### 5. Domínio gerado

Após o primeiro deploy, a Vercel gera automaticamente um domínio no formato:

```
https://prompt-manager-HASH.vercel.app
```

O domínio de produção estável (sem o hash) é vinculado ao branch `main`:

```
https://prompt-manager.vercel.app
```

Domínios customizados podem ser configurados em **Project > Settings > Domains**.

---

## Google OAuth na Vercel

### Como o fluxo funciona

1. O usuário clica em **Entrar com Google** no app hospedado na Vercel.
2. O Supabase redireciona o usuário para o Google.
3. O Google autentica e redireciona para o callback do Supabase.
4. O Supabase redireciona o usuário de volta para o frontend.

Fluxo resumido:

```
App na Vercel
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

- Atualizar o **Site URL** para o domínio de produção na Vercel
- Adicionar o domínio Vercel em **Redirect URLs**
- Manter também a URL local de desenvolvimento

Exemplos:

```text
Site URL:
https://prompt-manager.vercel.app

Redirect URLs:
http://localhost:2001/**
https://prompt-manager.vercel.app/**
https://prompt-manager-*.vercel.app/**
```

> O padrão com wildcard `https://prompt-manager-*.vercel.app/**` cobre os preview deployments gerados por PR (que têm URLs com hash variável).

#### Google Cloud Console → OAuth Client

- Adicionar o domínio Vercel em **Authorized JavaScript origins**
- Manter o callback do Supabase em **Authorized redirect URIs**

Exemplo:

```text
Authorized JavaScript origins:
http://localhost:2001
https://prompt-manager.vercel.app

Authorized redirect URIs:
https://fwkpzfmguweykwkpqald.supabase.co/auth/v1/callback
```

O domínio Vercel não substitui o redirect URI do Supabase.

### Conclusão prática sobre OAuth

Não há impeditivo técnico para publicar na Vercel com Google OAuth. A migração exige configuração manual de dashboard, não refatoração do fluxo de autenticação.

---

## SPA Routing

O projeto usa React Router com rotas client-side, por exemplo `/prompts/:slug`.

Se alguém acessar uma rota diretamente, a Vercel por padrão tentaria encontrar o arquivo correspondente e retornaria `404`. O `vercel.json` com a rewrite para `index.html` resolve isso, entregando o bundle React para qualquer rota e deixando o React Router assumir o controle.

Diferente do Railway (onde foi necessário criar um `server.mjs` para o fallback), na Vercel isso é resolvido com uma linha de configuração no `vercel.json`, sem servidor Node intermediário.

---

## Preview Deployments

A Vercel gera automaticamente um deploy de preview para cada pull request aberto no repositório conectado.

- Cada PR tem uma URL única e imutável: `https://prompt-manager-git-BRANCH-USER.vercel.app`
- A URL de preview é postada automaticamente como comentário no PR pelo bot da Vercel
- O ambiente de preview tem suas próprias variáveis (configuradas em **Environment Variables > Preview**)

Para que o Google OAuth funcione nos previews:

- Adicionar o padrão `https://prompt-manager-*.vercel.app/**` no Supabase Redirect URLs
- O Google Cloud Console aceita apenas origens exatas — para previews com URLs variáveis, o OAuth pode falhar se a origem do preview não estiver cadastrada

> Se testar OAuth em preview for necessário, uma alternativa é usar um domínio customizado para previews via Vercel ou cadastrar os domínios de preview individualmente no Google Cloud Console.

---

## Armadilhas Relevantes

**Variáveis Supabase não funcionam em produção:**

- Variáveis `VITE_*` são embutidas no bundle em **build time**, não em runtime
- As variáveis precisam estar definidas no painel da Vercel **antes** do build
- Alterar uma variável `VITE_*` exige redeploy para entrar em vigor
- `vercel env pull .env.local` sincroniza as variáveis localmente para desenvolvimento

**SPA routing retorna 404 em rotas diretas:**

```json
// ERRADO — sem vercel.json ou sem rewrite
// Acessar /prompts/meu-slug diretamente → 404

// CORRETO — vercel.json com rewrite
{
  "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }]
}
```

**Google OAuth falha após publicar:**

- O domínio Vercel precisa estar em `Redirect URLs` no Supabase
- O domínio Vercel precisa estar em `Authorized JavaScript origins` no Google Cloud
- O callback do Supabase precisa continuar cadastrado em `Authorized redirect URIs`
- Para preview deployments, o wildcard `https://prompt-manager-*.vercel.app/**` no Supabase cobre as URLs variáveis de PR

**Cache agressivo em dados dinâmicos:**

- Em rotas de API (se forem adicionadas no futuro), o Next.js/Vercel pode cachear respostas sem intenção
- Para uma SPA pura com Vite, isso não é relevante — todos os dados são buscados no client-side via Supabase

**Plano Hobby não permite uso comercial:**

- O plano gratuito proíbe uso comercial explicitamente nos termos de serviço
- Para projetos de clientes ou uso profissional, usar o plano Pro ($20/mês por membro)

---

## Checklist de Deploy

**Antes do primeiro deploy:**

- [ ] Criar projeto em [vercel.com/new](https://vercel.com/new)
- [ ] Conectar repositório GitHub ao projeto
- [ ] Adicionar `vercel.json` na raiz com rewrite para SPA routing
- [ ] Definir `VITE_SUPABASE_URL` nas variáveis do projeto (Production + Preview + Development)
- [ ] Definir `VITE_SUPABASE_PUBLISHABLE_KEY` nas variáveis do projeto
- [ ] Confirmar que as variáveis foram definidas **antes** do build inicial

**Após o primeiro deploy:**

- [ ] Confirmar que o domínio de produção foi gerado (`https://prompt-manager.vercel.app`)
- [ ] Atualizar o **Site URL** no Supabase para o domínio Vercel de produção
- [ ] Adicionar o domínio Vercel em **Redirect URLs** no Supabase (incluindo wildcard para previews)
- [ ] Adicionar o domínio Vercel em **Authorized JavaScript origins** no Google Cloud
- [ ] Confirmar que `https://fwkpzfmguweykwkpqald.supabase.co/auth/v1/callback` permanece em **Authorized redirect URIs**
- [ ] Verificar que a aplicação carrega e conecta ao Supabase
- [ ] Verificar que o login com Google conclui e retorna ao app
- [ ] Verificar logs e deployments no painel da Vercel

**Para preview deployments por PR:**

- [ ] Confirmar que o padrão `https://prompt-manager-*.vercel.app/**` está em **Redirect URLs** no Supabase
- [ ] Validar login OAuth em um ambiente preview (observar limitação de origens no Google Cloud)

**Quando um backend for adicionado:**

- [ ] Avaliar se migrar o frontend para o Railway faz mais sentido (stack unificado)
- [ ] Se mantiver na Vercel, adicionar API URL via variável de ambiente: `VITE_API_URL=https://backend.up.railway.app`
- [ ] Configurar CORS no backend para aceitar o domínio Vercel como origem
- [ ] Validar comunicação frontend ↔ backend em ambiente de preview

---

## Referências

- [Guia geral Vercel](./vercel.md)
- [Documentação oficial Vercel](https://vercel.com/docs)
- [Guia Deploy Vite na Vercel](https://vercel.com/docs/frameworks/vite)
- [Supabase Auth — Redirect URLs](https://supabase.com/docs/guides/auth/redirect-urls)
- [Supabase Auth — Login com Google](https://supabase.com/docs/guides/auth/social-login/auth-google)
- [Vercel — Environment Variables](https://vercel.com/docs/projects/environment-variables)
- [Vercel — Preview Deployments](https://vercel.com/docs/deployments/preview-deployments)
