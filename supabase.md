# Supabase — Guia de Uso (JavaScript / TypeScript / Node.js / Python)

> **Supabase** é uma alternativa open-source ao Firebase, construída sobre o PostgreSQL. Oferece banco de dados relacional, autenticação, storage de arquivos, realtime e edge functions — tudo como serviço, sem necessidade de gerenciar infraestrutura.

---

## Índice

1. [Introdução](#1-introdução)
2. [Conceitos Fundamentais e Chaves de Acesso](#2-conceitos-fundamentais-e-chaves-de-acesso)
3. [Formas de Conexão](#3-formas-de-conexão)
4. [Autenticação (Auth)](#4-autenticação-auth)
5. [Autorização e Row Level Security (RLS)](#5-autorização-e-row-level-security-rls)
6. [Aplicação Conversando Diretamente com o Banco (Frontend-Only App)](#6-aplicação-conversando-diretamente-com-o-banco-frontend-only-app)
7. [Operações com o Banco de Dados (Query Builder)](#7-operações-com-o-banco-de-dados-query-builder)
8. [Realtime](#8-realtime)
9. [Storage (Armazenamento de Arquivos)](#9-storage-armazenamento-de-arquivos)
10. [Edge Functions](#10-edge-functions)
11. [Boas Práticas](#11-boas-práticas)
12. [Limitações e Armadilhas](#12-limitações-e-armadilhas)
13. [Quando Usar e Quando Evitar](#13-quando-usar-e-quando-evitar)
14. [Alternativas Modernas](#14-alternativas-modernas)
15. [Ambiente Local e Self-Hosting com Docker](#15-ambiente-local-e-self-hosting-com-docker)
16. [Supabase CLI vs Flyway vs Docker](#16-supabase-cli-vs-flyway-vs-docker)
17. [Resumo Final](#17-resumo-final)

---

## Documentação Oficial

### Ponto de entrada
- [Supabase Docs — página inicial](https://supabase.com/docs)
- [Dashboard (console web)](https://supabase.com/dashboard)
- [Status dos serviços](https://status.supabase.com)
- [Changelog](https://supabase.com/changelog)

### Produtos
- [Database — visão geral](https://supabase.com/docs/guides/database/overview)
- [Auth — visão geral](https://supabase.com/docs/guides/auth)
- [Storage — visão geral](https://supabase.com/docs/guides/storage)
- [Realtime — visão geral](https://supabase.com/docs/guides/realtime)
- [Edge Functions — visão geral](https://supabase.com/docs/guides/functions)

### Conexão e APIs
- [API Keys e tipos de chave](https://supabase.com/docs/guides/api/api-keys)
- [REST API (PostgREST)](https://supabase.com/docs/guides/api)
- [GraphQL API](https://supabase.com/docs/guides/graphql)
- [Conectar ao banco — visão geral](https://supabase.com/docs/guides/database/connecting-to-postgres)
- [Supavisor — connection pooler](https://supabase.com/docs/guides/database/connection-pooling)

### Autenticação
- [Visão geral do Auth](https://supabase.com/docs/guides/auth)
- [Login com email e senha](https://supabase.com/docs/guides/auth/passwords)
- [Magic Link (OTP por email)](https://supabase.com/docs/guides/auth/auth-email-passwordless)
- [Login social (OAuth)](https://supabase.com/docs/guides/auth/social-login)
- [Auth com SSR (Next.js, Nuxt)](https://supabase.com/docs/guides/auth/server-side)
- [JWT e Signing Keys](https://supabase.com/docs/guides/auth/signing-keys)

### Segurança e RLS
- [Row Level Security — guia completo](https://supabase.com/docs/guides/database/postgres/row-level-security)
- [Proteger dados (secure data)](https://supabase.com/docs/guides/database/secure-data)
- [Roles do PostgreSQL no Supabase](https://supabase.com/docs/guides/database/postgres/roles)

### SDKs — Referência de API
- [JavaScript / TypeScript (`supabase-js`)](https://supabase.com/docs/reference/javascript/introduction)
- [Python (`supabase-py`)](https://supabase.com/docs/reference/python/introduction)
- [Flutter / Dart](https://supabase.com/docs/reference/dart/introduction)
- [Swift (iOS)](https://supabase.com/docs/reference/swift/introduction)
- [Kotlin (Android)](https://supabase.com/docs/reference/kotlin/introduction)
- [C#](https://supabase.com/docs/reference/csharp/introduction)

### Self-Hosting e CLI
- [Self-hosting com Docker](https://supabase.com/docs/guides/self-hosting/docker)
- [Desenvolvimento local com CLI](https://supabase.com/docs/guides/cli/local-development)
- [Referência completa do CLI](https://supabase.com/docs/reference/cli/introduction)
- [Migrations de banco](https://supabase.com/docs/guides/database/migrations)

### Quickstarts por framework
- [React](https://supabase.com/docs/guides/getting-started/quickstarts/reactjs)
- [Next.js](https://supabase.com/docs/guides/getting-started/quickstarts/nextjs)
- [Vue](https://supabase.com/docs/guides/getting-started/quickstarts/vue)
- [Flutter](https://supabase.com/docs/guides/getting-started/quickstarts/flutter)
- [Python / Flask](https://supabase.com/docs/guides/getting-started/quickstarts/flask)
- [Android (Kotlin)](https://supabase.com/docs/guides/getting-started/quickstarts/kotlin)

### Comunidade e suporte
- [GitHub — repositório oficial](https://github.com/supabase/supabase)
- [Discord](https://discord.supabase.com)
- [YouTube](https://youtube.com/c/supabase)
- [Suporte oficial](https://supabase.com/support)

---

## 1. Introdução

### O que é Supabase

**Supabase** é uma plataforma Backend-as-a-Service (BaaS) de código aberto que oferece um conjunto completo de ferramentas para construir aplicações modernas sem precisar gerenciar infraestrutura de servidor. É frequentemente descrito como "o Firebase open-source", mas com uma diferença fundamental: usa **PostgreSQL** como banco de dados, em vez de um banco NoSQL.

### Por que usar

- **Substituto ao Firebase**: mesma experiência de BaaS, porém com banco relacional e SQL completo
- **PostgreSQL gerenciado**: banco de dados poderoso, maduro e com suporte a tipos complexos, JOINs, transações, etc.
- **Open-source**: pode ser self-hosted, sem lock-in de fornecedor
- **APIs automáticas**: REST e GraphQL geradas automaticamente a partir do schema do banco
- **Ecossistema completo**: auth, storage, realtime e edge functions integrados

### Quando faz sentido aplicar

- ✅ Projetos que precisam de backend completo com pouca configuração
- ✅ MVPs e protótipos rápidos
- ✅ Aplicações frontend-only (React, Vue, Angular) que precisam de banco + auth
- ✅ Migração do Firebase para SQL
- ✅ Equipes pequenas ou startups que querem velocidade de desenvolvimento
- ✅ Projetos que precisam de sincronização em tempo real

### Principais produtos

| Produto | Descrição |
|---|---|
| **Database** | PostgreSQL completo com API REST e GraphQL automáticos |
| **Auth** | Autenticação com email/senha, magic link, OAuth e SSO |
| **Storage** | Upload e gestão de arquivos (imagens, PDFs, vídeos, etc.) |
| **Realtime** | Subscriptions a mudanças no banco e comunicação entre clientes |
| **Edge Functions** | Funções serverless em Deno, distribuídas globalmente |

---

## 2. Conceitos Fundamentais e Chaves de Acesso

### 2.1 Tipos de Chaves (API Keys)

O Supabase usa três tipos principais de credenciais para controlar o acesso à API e ao banco de dados:

#### anon key (publishable key) — Chave Pública

```
Formatos possíveis:
- Formato antigo (JWT): eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
- Formato novo:         sb_publishable_xxx
```

- **O que é**: chave pública do projeto, segura para incluir no código do frontend
- **Quando usar**: em aplicações cliente (browser, mobile) **COM RLS habilitado**
- **O que pode fazer**: o que as políticas RLS de `anon` e `authenticated` permitirem
- ⚠️ **Atenção**: sem RLS ativo, a `anon key` dá acesso a todos os dados!

#### service_role key (secret key) — Chave Secreta

```
Formato: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9... (JWT com role=service_role)
```

- **O que é**: chave secreta com privilégios administrativos totais
- **Quando usar**: apenas no backend/servidor (Node.js, Python, Edge Functions)
- **O que pode fazer**: bypassa completamente o RLS, acesso irrestrito ao banco
- 🚫 **NUNCA exponha no frontend/browser!**

#### JWT secret — Segredo de Assinatura

- Usado internamente para assinar e verificar tokens JWT
- Necessário quando você quer verificar tokens manualmente no backend
- Nunca expor publicamente

### 2.2 Exemplo: output do `supabase status` (ambiente local)

```text
API URL:     http://127.0.0.1:54321
GraphQL URL: http://127.0.0.1:54321/graphql/v1
DB URL:      postgresql://postgres:postgres@127.0.0.1:54322/postgres
Studio URL:  http://127.0.0.1:54323
anon key:    eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
service_role key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

> 💡 O **Studio** é a interface visual do Supabase (equivalente ao pgAdmin + dashboard). Localmente, acesse `http://127.0.0.1:54323`.

---

## 3. Formas de Conexão

### 3.1 Conexão via Cliente JavaScript (Frontend e Backend)

O SDK oficial do Supabase para JavaScript/TypeScript suporta tanto Node.js quanto browser.

**Instalação:**

```bash
npm install @supabase/supabase-js
```

**Inicialização básica — Frontend (usa anon key):**

```javascript
import { createClient } from '@supabase/supabase-js'

// A anon key é segura no frontend APENAS se o RLS estiver ativo em todas as tabelas
const supabase = createClient(
  'https://seu-projeto.supabase.co',  // URL do projeto
  'sua-anon-key'                       // chave pública
)
```

**Inicialização no Backend/Servidor (usa service_role key):**

```javascript
import { createClient } from '@supabase/supabase-js'

// NUNCA exponha a service_role key no browser!
// Carregue sempre de variáveis de ambiente
const supabaseAdmin = createClient(
  process.env.SUPABASE_URL,
  process.env.SUPABASE_SERVICE_ROLE_KEY,
  {
    auth: {
      persistSession: false,       // não persiste sessão no servidor (sem cookies)
      autoRefreshToken: false,     // não tenta renovar tokens automaticamente
      detectSessionInUrl: false,   // não lê tokens da URL (evita erros em SSR)
    }
  }
)
```

### 3.2 Conexão via Python (Backend)

```python
from supabase import create_client
from supabase.lib.client_options import ClientOptions

# Cliente para uso com usuário autenticado (respeita RLS)
supabase = create_client(
    "https://seu-projeto.supabase.co",
    "sua-anon-key"
)

# Cliente admin com privilégios totais (bypassa RLS)
supabase_admin = create_client(
    "https://seu-projeto.supabase.co",
    "sua-service-role-key",
    options=ClientOptions(
        auto_refresh_token=False,  # não renova token automaticamente
        persist_session=False,     # não salva sessão em disco
    )
)

# Acesso ao cliente de administração de usuários
admin_auth_client = supabase_admin.auth.admin

# Exemplo: listar todos os usuários (requer service_role)
response = admin_auth_client.list_users()
```

### 3.3 Conexão Direta ao PostgreSQL (ORMs, migrations, ferramentas)

O Supabase expõe o PostgreSQL diretamente, permitindo uso com qualquer ORM ou ferramenta SQL.

```
# Conexão direta ao banco (requer IPv6 ou IP liberado no painel)
# Porta padrão do PostgreSQL
postgresql://postgres:[SENHA]@db.[PROJECT_REF].supabase.co:5432/postgres

# Via Connection Pooler (Supavisor) — recomendado para serverless
# Porta 6543 com pooling de transações
postgresql://postgres.[PROJECT_REF]:[SENHA]@aws-0-[REGION].pooler.supabase.com:6543/postgres
```

**Quando usar conexão direta vs pooler:**

| Situação | Recomendação |
|---|---|
| Servidor Node.js/Python de longa duração | Conexão direta (porta 5432) |
| Funções serverless (Vercel, AWS Lambda) | Pooler Supavisor (porta 6543) |
| Migrations com Prisma/Drizzle | Conexão direta |
| APIs com alta concorrência | Pooler Supavisor |

> 💡 **Supavisor** é o connection pooler do Supabase. Em ambientes serverless, cada invocação cria uma nova conexão — sem pooler, você rapidamente esgota o limite de conexões do PostgreSQL.

### 3.4 Via REST API (sem SDK)

O Supabase gera automaticamente uma API REST para todas as tabelas do schema `public` via **PostgREST**.

```bash
# Buscar todos os produtos
curl 'https://seu-projeto.supabase.co/rest/v1/produtos?select=*'   -H "apikey: sua-anon-key"   -H "Authorization: Bearer sua-anon-key"

# Buscar com filtro (eq = igual)
curl 'https://seu-projeto.supabase.co/rest/v1/produtos?categoria=eq.eletronicos'   -H "apikey: sua-anon-key"   -H "Authorization: Bearer sua-anon-key"

# Inserir registro
curl -X POST 'https://seu-projeto.supabase.co/rest/v1/produtos'   -H "apikey: sua-anon-key"   -H "Authorization: Bearer sua-anon-key"   -H "Content-Type: application/json"   -d '{"nome": "Notebook", "preco": 3500}'
```

### 3.5 Via GraphQL

O Supabase oferece uma API GraphQL automática via **pg_graphql**.

```bash
curl -X POST 'https://seu-projeto.supabase.co/graphql/v1'   -H "apikey: sua-anon-key"   -H "Content-Type: application/json"   -d '{
    "query": "{ produtosCollection { edges { node { id nome preco } } } }"
  }'
```

---

## 4. Autenticação (Auth)

O Supabase Auth oferece múltiplos métodos de autenticação prontos para uso, integrados ao banco de dados via tabela `auth.users`.

### 4.1 Cadastro e Login com Email/Senha

```javascript
// Cadastro de novo usuário
const { data, error } = await supabase.auth.signUp({
  email: 'usuario@email.com',
  password: 'senha123',
  // Dados adicionais salvos em auth.users.user_metadata
  options: {
    data: {
      nome_completo: 'João Silva',
    }
  }
})

// Login com email e senha
const { data, error } = await supabase.auth.signInWithPassword({
  email: 'usuario@email.com',
  password: 'senha123',
})

// Logout (limpa sessão local e revoga token no servidor)
await supabase.auth.signOut()

// Obter usuário atualmente autenticado
const { data: { user } } = await supabase.auth.getUser()
console.log(user.id)    // UUID único do usuário
console.log(user.email) // email do usuário

// Obter sessão atual (com access_token e refresh_token)
const { data: { session } } = await supabase.auth.getSession()
```

### 4.2 Magic Link (Link mágico por email — sem senha)

O usuário recebe um link por email que, ao clicar, já o autentica sem precisar de senha.

```javascript
// Enviar magic link para o email
const { error } = await supabase.auth.signInWithOtp({
  email: 'usuario@email.com',
  options: {
    // URL para redirecionar após o clique no link
    emailRedirectTo: 'https://meuapp.com/auth/callback',
  }
})
```

**Exemplo completo em React:**

```javascript
import { useState } from 'react'
import { supabase } from './supabaseClient'

export default function Auth() {
  const [email, setEmail] = useState('')
  const [loading, setLoading] = useState(false)

  const handleLogin = async (e) => {
    e.preventDefault()
    setLoading(true)

    const { error } = await supabase.auth.signInWithOtp({ email })

    if (error) {
      alert('Erro: ' + error.message)
    } else {
      alert('Verifique seu email! O link de acesso foi enviado.')
    }

    setLoading(false)
  }

  return (
    <form onSubmit={handleLogin}>
      <label>Email:</label>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="seu@email.com"
        required
      />
      <button type="submit" disabled={loading}>
        {loading ? 'Enviando...' : 'Enviar Magic Link'}
      </button>
    </form>
  )
}
```

### 4.3 OAuth (Login Social — Google, GitHub, etc.)

```javascript
// Login com GitHub — redireciona para o OAuth do GitHub
const { data, error } = await supabase.auth.signInWithOAuth({
  provider: 'github',
  options: {
    redirectTo: 'https://meuapp.com/auth/callback', // URL de retorno
  }
})

// Login com Google
const { data, error } = await supabase.auth.signInWithOAuth({
  provider: 'google',
  options: {
    // Solicitar permissões adicionais
    scopes: 'https://www.googleapis.com/auth/calendar.readonly',
  }
})

// Outros providers suportados:
// 'facebook', 'twitter', 'discord', 'slack', 'spotify', 'apple', etc.
```

> ⚙️ Para cada provider OAuth, você precisa configurar as credenciais (Client ID e Client Secret) no painel do Supabase, em **Authentication → Providers**.

### 4.3.1 Configurar Google OAuth (passo a passo)

**No Google Cloud Console** (https://console.cloud.google.com/):

1. Crie ou selecione um projeto
2. **APIs & Services → OAuth consent screen** → selecione **Externo**
3. Preencha App name, emails de contato, avance sem adicionar scopes
4. Em **Test users**, adicione seu email (em modo Testing, só esses emails conseguem logar)
5. **APIs & Services → Credentials → + Create Credentials → OAuth client ID**
6. Application type: **Web application**
7. Preencha:
   ```
   Authorized JavaScript origins:
     http://localhost:5173

   Authorized redirect URIs:
     https://SEU_PROJECT_REF.supabase.co/auth/v1/callback
   ```
8. Copie o **Client ID** e o **Client Secret**

**No Supabase Dashboard**:

1. **Authentication → Providers → Google**
2. Ative **Enable Sign in with Google**
3. Cole o Client ID e Client Secret
4. Clique **Save**

> Para publicar para todos os usuários (sair do modo Testing), é necessário verificar o OAuth Consent Screen com o Google.

### 4.4 Escutar mudanças de sessão

```javascript
// Registra um listener para mudanças de estado de autenticação
// Útil para redirecionar o usuário após login/logout
const { data: { subscription } } = supabase.auth.onAuthStateChange((event, session) => {
  console.log('Evento:', event)    // SIGNED_IN, SIGNED_OUT, TOKEN_REFRESHED, etc.
  console.log('Sessão:', session)  // null se deslogado, objeto session se logado

  switch (event) {
    case 'SIGNED_IN':
      // Usuário acabou de logar
      break
    case 'SIGNED_OUT':
      // Usuário deslogou
      break
    case 'TOKEN_REFRESHED':
      // Token foi renovado automaticamente
      break
    case 'PASSWORD_RECOVERY':
      // Usuário clicou em link de recuperação de senha
      break
  }
})

// Para remover o listener quando não precisar mais
subscription.unsubscribe()
```

### 4.5 Admin Auth (Backend — criar/gerenciar usuários)

Estas operações requerem a `service_role key` e devem ser feitas **apenas no backend**.

```javascript
// Criar usuário via admin (sem enviar email de confirmação)
const { data, error } = await supabaseAdmin.auth.admin.createUser({
  email: 'novo@email.com',
  password: 'senha-segura',
  email_confirm: true,          // confirma o email automaticamente
  user_metadata: {              // dados extras salvos no perfil
    nome: 'Maria Santos',
    plano: 'premium',
  }
})

// Listar todos os usuários (com paginação)
const { data: { users }, error } = await supabaseAdmin.auth.admin.listUsers({
  page: 1,
  perPage: 50,
})

// Atualizar dados de um usuário
const { data, error } = await supabaseAdmin.auth.admin.updateUserById(
  'uuid-do-usuario',
  {
    email: 'novo-email@email.com',
    password: 'nova-senha',
    user_metadata: { plano: 'enterprise' },
  }
)

// Deletar usuário permanentemente
const { error } = await supabaseAdmin.auth.admin.deleteUser('uuid-do-usuario')

// Gerar link de confirmação de email (útil para onboarding customizado)
const { data } = await supabaseAdmin.auth.admin.generateLink({
  type: 'signup',
  email: 'usuario@email.com',
})
```

### 4.6 Implementar Auth no React (padrão AuthContext)

Estrutura recomendada para gerenciar sessão em toda a aplicação:

```
src/
  contexts/
    authTypes.ts       ← tipos + criação do contexto
    AuthContext.tsx     ← AuthProvider
  hooks/
    useAuth.ts         ← hook de consumo
  components/
    ProtectedRoute.tsx ← wrapper de rota protegida
```

**authTypes.ts**

```typescript
import { createContext } from 'react'
import type { User, Session } from '@supabase/supabase-js'

export interface AuthContextType {
  user: User | null
  session: Session | null
  loading: boolean
  signInWithGoogle: () => Promise<void>
  signOut: () => Promise<void>
}

export const AuthContext = createContext<AuthContextType | undefined>(undefined)
```

**AuthContext.tsx**

```typescript
import { useEffect, useState } from 'react'
import type { User, Session } from '@supabase/supabase-js'
import { supabase } from '../lib/supabaseClient'
import { AuthContext } from './authTypes'

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser]       = useState<User | null>(null)
  const [session, setSession] = useState<Session | null>(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    supabase.auth.getSession().then(({ data: { session } }) => {
      setSession(session)
      setUser(session?.user ?? null)
      setLoading(false)
    })

    const { data: { subscription } } = supabase.auth.onAuthStateChange(
      (_event, session) => {
        setSession(session)
        setUser(session?.user ?? null)
        setLoading(false)
      }
    )

    return () => subscription.unsubscribe()
  }, [])

  const signInWithGoogle = async () => {
    const { error } = await supabase.auth.signInWithOAuth({
      provider: 'google',
      options: { redirectTo: window.location.origin },
    })
    if (error) throw error
  }

  const signOut = async () => {
    const { error } = await supabase.auth.signOut()
    if (error) throw error
  }

  return (
    <AuthContext.Provider value={{ user, session, loading, signInWithGoogle, signOut }}>
      {children}
    </AuthContext.Provider>
  )
}
```

**useAuth.ts**

```typescript
import { useContext } from 'react'
import { AuthContext } from '../contexts/authTypes'

export function useAuth() {
  const context = useContext(AuthContext)
  if (!context) throw new Error('useAuth must be used within an AuthProvider')
  return context
}
```

**ProtectedRoute.tsx**

```typescript
import { Navigate } from 'react-router-dom'
import { useAuth } from '../hooks/useAuth'

export function ProtectedRoute({ children }: { children: React.ReactNode }) {
  const { user, loading } = useAuth()

  if (loading) return <div>Carregando...</div>
  if (!user)   return <Navigate to="/login" replace />

  return <>{children}</>
}
```

**App.tsx — montagem**

```typescript
import { BrowserRouter, Routes, Route } from 'react-router-dom'
import { AuthProvider } from './contexts/AuthContext'
import { ProtectedRoute } from './components/ProtectedRoute'

function App() {
  return (
    <BrowserRouter>
      <AuthProvider>
        <Routes>
          <Route path="/login" element={<LoginPage />} />
          <Route path="/*" element={
            <ProtectedRoute>
              <Routes>
                <Route path="/" element={<HomePage />} />
              </Routes>
            </ProtectedRoute>
          } />
        </Routes>
      </AuthProvider>
    </BrowserRouter>
  )
}
```

---

## 5. Autorização e Row Level Security (RLS)

### 5.1 O que é RLS

**Row Level Security (RLS)** é uma funcionalidade nativa do PostgreSQL que define, diretamente no banco de dados, quais linhas cada usuário pode **ver** e **modificar**. É o mecanismo central de segurança do Supabase.

**Por que isso é poderoso:**
- A segurança fica no banco de dados, não apenas na aplicação
- Mesmo que alguém bypasse sua API, os dados continuam protegidos
- O Supabase propaga automaticamente a identidade do usuário (via JWT) para o contexto do PostgreSQL

**Como funciona:**
1. Usuário faz login → recebe um JWT com seu `user_id`
2. Cliente envia o JWT em cada requisição
3. Supabase extrai o `user_id` do JWT e define como contexto (`auth.uid()`)
4. PostgreSQL aplica as políticas RLS usando esse contexto
5. Query retorna apenas as linhas permitidas

### 5.2 Ativando o RLS em uma tabela

```sql
-- Habilitar RLS na tabela (por padrão vem desabilitado)
ALTER TABLE public.tarefas ENABLE ROW LEVEL SECURITY;

-- Com RLS ativo e SEM políticas definidas:
-- Nenhum usuário consegue acessar nada (deny-all por padrão)

-- Forçar RLS inclusive para o dono da tabela (máxima segurança)
ALTER TABLE public.tarefas FORCE ROW LEVEL SECURITY;
```

### 5.3 Criando Políticas (Policies)

**Política: usuário vê apenas seus próprios dados (SELECT)**

```sql
-- O usuário autenticado só vê tarefas onde user_id = seu próprio ID
CREATE POLICY "Usuário vê próprias tarefas"
  ON public.tarefas
  FOR SELECT
  USING (auth.uid() = user_id);
  -- USING define a condição para leitura (WHERE implícito)
```

**Política: usuário só insere dados para si mesmo (INSERT)**

```sql
-- Garante que ao inserir, user_id seja sempre o do usuário logado
CREATE POLICY "Usuário insere suas próprias tarefas"
  ON public.tarefas
  FOR INSERT
  WITH CHECK (auth.uid() = user_id);
  -- WITH CHECK define a condição para escrita
```

**Política para todas as operações (ALL)**

```sql
-- Combina SELECT, INSERT, UPDATE e DELETE em uma única política
CREATE POLICY "Usuários acessam apenas seus dados"
  ON public.tarefas
  FOR ALL
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);
```

**Tabela pública (qualquer pessoa pode ler, sem autenticação)**

```sql
-- Permite leitura para usuários anônimos e autenticados
CREATE POLICY "Leitura pública"
  ON public.produtos
  FOR SELECT
  TO anon, authenticated  -- roles que se aplicam
  USING (true);           -- sem restrição adicional
```

**Política para administradores (baseada em role customizada)**

```sql
-- Apenas usuários com claim 'admin' no JWT podem deletar
CREATE POLICY "Apenas admins deletam"
  ON public.tarefas
  FOR DELETE
  USING (auth.jwt() ->> 'role' = 'admin');
```

### 5.4 Funções úteis nas Policies

| Função | Retorna | Uso |
|---|---|---|
| `auth.uid()` | `UUID` | ID do usuário autenticado |
| `auth.role()` | `text` | Role do usuário: `'anon'` ou `'authenticated'` |
| `auth.jwt()` | `jsonb` | JWT completo (para claims customizados) |
| `auth.jwt() ->> 'campo'` | `text` | Acessa campo específico do JWT |

### 5.5 Efeito prático das Policies

**Sem RLS (ou usando service_role):**

```javascript
// Retorna TODOS os registros da tabela, independente do usuário
const { data } = await supabase.from('tarefas').select('*')
// → [{ id: 1, user_id: 'abc', titulo: 'Tarefa do João' },
//    { id: 2, user_id: 'xyz', titulo: 'Tarefa da Maria' }, ...]
```

**Com RLS ativo e usuário autenticado:**

```javascript
// O WHERE é aplicado automaticamente pelo PostgreSQL
// Retorna APENAS os registros do usuário logado — sem nenhuma mudança no código!
const { data } = await supabase.from('tarefas').select('*')
// → [{ id: 1, user_id: 'abc', titulo: 'Minha tarefa' }]
// (apenas as tarefas do usuário atual)
```

---

## 6. Aplicação Conversando Diretamente com o Banco (Frontend-Only App)

Uma das grandes vantagens do Supabase é permitir que aplicações frontend se comuniquem **diretamente** com o banco de dados, sem necessidade de um servidor intermediário (backend próprio). Isso é seguro quando configurado corretamente.

### 6.1 Requisitos para funcionar com segurança

1. ✅ **RLS habilitado** em **todas** as tabelas com dados sensíveis
2. ✅ Usar apenas a `anon key` (chave pública) no frontend
3. ✅ Definir políticas precisas para cada operação (SELECT, INSERT, UPDATE, DELETE)
4. ✅ Nunca incluir a `service_role key` no código do frontend
5. ✅ Testar as políticas com usuários diferentes antes de fazer deploy

### 6.2 Exemplo completo de app frontend-only (React + Supabase)

**Arquivo de configuração do cliente:**

```javascript
// src/supabaseClient.js
import { createClient } from '@supabase/supabase-js'

// Variáveis de ambiente do Vite (prefixo VITE_)
// Em Next.js, usar NEXT_PUBLIC_ em vez de VITE_
export const supabase = createClient(
  import.meta.env.VITE_SUPABASE_URL,
  import.meta.env.VITE_SUPABASE_ANON_KEY
)
```

**Arquivo `.env` (nunca commitar com dados reais):**

```env
# .env.local
VITE_SUPABASE_URL=https://seu-projeto.supabase.co
VITE_SUPABASE_ANON_KEY=sua-anon-key-aqui
```

**Componente principal com CRUD completo:**

```javascript
// src/App.jsx — CRUD completo sem backend próprio
import { useEffect, useState } from 'react'
import { supabase } from './supabaseClient'

export default function App() {
  const [tarefas, setTarefas] = useState([])
  const [novaTarefa, setNovaTarefa] = useState('')
  const [carregando, setCarregando] = useState(false)

  // Buscar tarefas do usuário logado
  // O RLS garante que apenas as tarefas do usuário atual são retornadas
  const buscarTarefas = async () => {
    setCarregando(true)
    const { data, error } = await supabase
      .from('tarefas')
      .select('*')
      .order('created_at', { ascending: false }) // mais recentes primeiro

    if (error) console.error('Erro ao buscar:', error.message)
    else setTarefas(data)
    setCarregando(false)
  }

  // Criar nova tarefa
  const criarTarefa = async () => {
    if (!novaTarefa.trim()) return

    // Obter o usuário atual para preencher o user_id
    const { data: { user } } = await supabase.auth.getUser()

    const { error } = await supabase.from('tarefas').insert({
      titulo: novaTarefa,
      user_id: user.id,  // RLS vai validar que user_id == auth.uid()
    })

    if (error) {
      console.error('Erro ao criar:', error.message)
    } else {
      setNovaTarefa('')
      buscarTarefas() // recarregar lista
    }
  }

  // Marcar tarefa como concluída
  const concluirTarefa = async (id) => {
    const { error } = await supabase
      .from('tarefas')
      .update({ concluida: true })
      .eq('id', id)
      // RLS impede atualizar tarefas de outros usuários

    if (!error) buscarTarefas()
  }

  // Deletar tarefa
  const deletarTarefa = async (id) => {
    const { error } = await supabase
      .from('tarefas')
      .delete()
      .eq('id', id)
      // RLS impede deletar tarefas de outros usuários

    if (!error) buscarTarefas()
  }

  // Carregar tarefas ao montar o componente
  useEffect(() => {
    buscarTarefas()
  }, [])

  return (
    <div>
      <h1>Minhas Tarefas</h1>

      {/* Formulário de criação */}
      <div>
        <input
          type="text"
          value={novaTarefa}
          onChange={(e) => setNovaTarefa(e.target.value)}
          placeholder="Nova tarefa..."
          onKeyDown={(e) => e.key === 'Enter' && criarTarefa()}
        />
        <button onClick={criarTarefa}>Adicionar</button>
      </div>

      {/* Lista de tarefas */}
      {carregando ? (
        <p>Carregando...</p>
      ) : (
        <ul>
          {tarefas.map((t) => (
            <li key={t.id} style={{ textDecoration: t.concluida ? 'line-through' : 'none' }}>
              {t.titulo}
              {!t.concluida && (
                <button onClick={() => concluirTarefa(t.id)}>✓ Concluir</button>
              )}
              <button onClick={() => deletarTarefa(t.id)}>🗑 Excluir</button>
            </li>
          ))}
        </ul>
      )}
    </div>
  )
}
```

**Schema SQL necessário para o exemplo:**

```sql
-- Criar tabela de tarefas
CREATE TABLE public.tarefas (
  id         UUID        DEFAULT gen_random_uuid() PRIMARY KEY,
  titulo     TEXT        NOT NULL,
  concluida  BOOLEAN     DEFAULT false,
  user_id    UUID        REFERENCES auth.users(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Habilitar Row Level Security
ALTER TABLE public.tarefas ENABLE ROW LEVEL SECURITY;

-- Política: cada usuário gerencia apenas suas próprias tarefas
CREATE POLICY "Usuário gerencia suas tarefas"
  ON public.tarefas
  FOR ALL  -- SELECT, INSERT, UPDATE, DELETE
  USING (auth.uid() = user_id)           -- para leitura
  WITH CHECK (auth.uid() = user_id);     -- para escrita
```

---

## 7. Operações com o Banco de Dados (Query Builder)

O SDK do Supabase oferece um query builder fluente que se traduz em chamadas à API REST do PostgREST.

```javascript
// ─── SELECT ───────────────────────────────────────────────────────────────────

// Buscar todas as colunas
const { data, error } = await supabase
  .from('produtos')
  .select('*')

// Selecionar colunas específicas (mais eficiente)
const { data } = await supabase
  .from('produtos')
  .select('id, nome, preco')

// SELECT com múltiplos filtros
const { data } = await supabase
  .from('produtos')
  .select('*')
  .eq('categoria', 'eletronicos')   // WHERE categoria = 'eletronicos'
  .gte('preco', 100)                 // AND preco >= 100
  .lte('preco', 1000)                // AND preco <= 1000
  .ilike('nome', '%notebook%')       // AND nome ILIKE '%notebook%' (case-insensitive)
  .order('preco', { ascending: true }) // ORDER BY preco ASC
  .limit(10)                         // LIMIT 10
  .range(0, 9)                       // OFFSET 0 LIMIT 10 (paginação)

// SELECT com JOIN (relacionamentos)
const { data } = await supabase
  .from('pedidos')
  .select(`
    id,
    total,
    status,
    clientes ( nome, email ),
    itens_pedido (
      quantidade,
      produtos ( nome, preco )
    )
  `)
  // Retorna objetos aninhados automaticamente

// Filtros disponíveis:
// .eq('col', val)       → col = val
// .neq('col', val)      → col != val
// .gt('col', val)       → col > val
// .gte('col', val)      → col >= val
// .lt('col', val)       → col < val
// .lte('col', val)      → col <= val
// .like('col', '%val%') → col LIKE '%val%'
// .ilike('col', '%val%')→ col ILIKE '%val%'
// .in('col', [1,2,3])   → col IN (1, 2, 3)
// .is('col', null)      → col IS NULL
// .contains('col', val) → col @> val (arrays/jsonb)
// .or('col1.eq.val1,col2.eq.val2') → OR conditions

// ─── INSERT ───────────────────────────────────────────────────────────────────

// Inserir um registro
const { data, error } = await supabase
  .from('produtos')
  .insert({
    nome: 'Notebook Dell',
    preco: 3500,
    categoria: 'eletronicos'
  })
  .select() // retornar o registro criado

// Inserir múltiplos registros de uma vez
const { data, error } = await supabase
  .from('produtos')
  .insert([
    { nome: 'Mouse', preco: 80 },
    { nome: 'Teclado', preco: 150 },
    { nome: 'Monitor', preco: 900 },
  ])
  .select()

// ─── UPDATE ───────────────────────────────────────────────────────────────────

// Atualizar registro específico
const { data, error } = await supabase
  .from('produtos')
  .update({
    preco: 3200,
    atualizado_em: new Date().toISOString(),
  })
  .eq('id', 'uuid-do-produto')  // SEMPRE usar filtro para não atualizar tudo!
  .select()                      // retornar o registro atualizado

// ─── DELETE ───────────────────────────────────────────────────────────────────

// Deletar registro específico
const { error } = await supabase
  .from('produtos')
  .delete()
  .eq('id', 'uuid-do-produto')  // SEMPRE usar filtro!

// Deletar múltiplos com filtro
const { error } = await supabase
  .from('logs')
  .delete()
  .lt('created_at', '2024-01-01') // deletar logs antes de 2024

// ─── UPSERT ───────────────────────────────────────────────────────────────────

// Inserir ou atualizar (baseado na chave primária ou unique constraint)
const { data, error } = await supabase
  .from('perfis')
  .upsert({
    id: user.id,              // se existir, atualiza; se não, insere
    nome: 'João Silva',
    avatar_url: 'https://...',
    atualizado_em: new Date().toISOString(),
  })
  .select()

// ─── CONTAGEM ─────────────────────────────────────────────────────────────────

// Contar registros sem retornar dados
const { count, error } = await supabase
  .from('produtos')
  .select('*', { count: 'exact', head: true }) // head: true não retorna rows

// ─── EXECUTAR SQL CUSTOMIZADO ─────────────────────────────────────────────────

// Para queries complexas, usar RPC (Remote Procedure Call) com funções SQL
const { data, error } = await supabase
  .rpc('calcular_total_pedidos', {
    usuario_id: user.id,
    data_inicio: '2024-01-01',
  })
```

---

## 8. Realtime

O Supabase oferece subscriptions em tempo real para mudanças no banco de dados, comunicação entre clientes (broadcast) e rastreamento de presença online.

```javascript
// ─── POSTGRES CHANGES (mudanças no banco) ────────────────────────────────────

// Escutar inserções em uma tabela específica
const channel = supabase
  .channel('tarefas-changes')       // nome do canal (qualquer string)
  .on(
    'postgres_changes',
    {
      event: 'INSERT',              // INSERT | UPDATE | DELETE | *
      schema: 'public',
      table: 'tarefas',
      // filter: 'user_id=eq.abc'  // opcional: filtrar por coluna
    },
    (payload) => {
      console.log('Nova tarefa inserida:', payload.new)
      // payload.new → objeto com o novo registro
      // payload.old → objeto com o registro antigo (em UPDATE/DELETE)
    }
  )
  .subscribe()

// Escutar qualquer mudança em qualquer tabela
const channel = supabase
  .channel('todas-mudancas')
  .on(
    'postgres_changes',
    { event: '*', schema: 'public', table: '*' },
    (payload) => {
      console.log('Mudança detectada:', payload.eventType, payload)
    }
  )
  .subscribe()

// Parar de escutar e remover o canal
supabase.removeChannel(channel)

// ─── BROADCAST (comunicação entre clientes) ───────────────────────────────────

// Criar canal de broadcast
const canal = supabase.channel('sala-chat-001')

// Escutar mensagens
canal
  .on('broadcast', { event: 'nova-mensagem' }, (payload) => {
    console.log('Mensagem recebida:', payload.payload)
    // Mostrar mensagem na interface
  })
  .subscribe()

// Enviar broadcast para todos no canal
canal.send({
  type: 'broadcast',
  event: 'nova-mensagem',
  payload: {
    texto: 'Olá, mundo!',
    autor: 'João',
    timestamp: new Date().toISOString(),
  }
})

// ─── PRESENCE (rastreamento de usuários online) ───────────────────────────────

const canal = supabase.channel('sala-presenca')

// Escutar mudanças de presença
canal
  .on('presence', { event: 'sync' }, () => {
    // Chamado quando o estado de presença sincroniza
    const state = canal.presenceState()
    console.log('Usuários online:', Object.keys(state).length)
  })
  .on('presence', { event: 'join' }, ({ key, newPresences }) => {
    console.log('Usuário entrou:', newPresences)
  })
  .on('presence', { event: 'leave' }, ({ key, leftPresences }) => {
    console.log('Usuário saiu:', leftPresences)
  })
  .subscribe(async (status) => {
    if (status === 'SUBSCRIBED') {
      // Anunciar presença com dados customizados
      await canal.track({
        user_id: user.id,
        nome: 'João',
        online_at: new Date().toISOString(),
      })
    }
  })
```

> ⚠️ **Realtime e RLS**: as políticas de SELECT se aplicam ao Realtime também. Um usuário só receberá eventos de linhas que ele teria permissão de ler via SELECT.

---

## 9. Storage (Armazenamento de Arquivos)

O Supabase Storage permite upload, download e gerenciamento de arquivos, com integração ao sistema de autenticação e RLS.

```javascript
// ─── UPLOAD ───────────────────────────────────────────────────────────────────

// Upload de imagem de avatar do usuário
const { data, error } = await supabase.storage
  .from('avatares')                         // nome do bucket
  .upload(`${user.id}/avatar.png`, arquivo, { // caminho dentro do bucket
    contentType: 'image/png',
    upsert: true,                            // sobrescreve se já existir
    cacheControl: '3600',                    // cache por 1 hora
  })

// Upload de arquivo a partir de input HTML
const handleFileUpload = async (event) => {
  const arquivo = event.target.files[0]
  const nomeArquivo = `${Date.now()}_${arquivo.name}` // nome único

  const { data, error } = await supabase.storage
    .from('documentos')
    .upload(`${user.id}/${nomeArquivo}`, arquivo)

  if (error) console.error('Erro no upload:', error.message)
  else console.log('Arquivo salvo em:', data.path)
}

// ─── URLs DE ACESSO ───────────────────────────────────────────────────────────

// URL pública (funciona apenas em buckets públicos)
const { data } = supabase.storage
  .from('avatares')
  .getPublicUrl(`${user.id}/avatar.png`)

console.log('URL pública:', data.publicUrl)
// → https://seu-projeto.supabase.co/storage/v1/object/public/avatares/uuid/avatar.png

// URL assinada (para buckets privados — expira em X segundos)
const { data, error } = await supabase.storage
  .from('documentos')
  .createSignedUrl(`${user.id}/relatorio.pdf`, 3600) // expira em 1 hora

console.log('URL temporária:', data.signedUrl)

// Gerar múltiplas URLs assinadas de uma vez
const { data } = await supabase.storage
  .from('documentos')
  .createSignedUrls(['arquivo1.pdf', 'arquivo2.pdf'], 3600)

// ─── DOWNLOAD ─────────────────────────────────────────────────────────────────

// Baixar arquivo como Blob
const { data, error } = await supabase.storage
  .from('documentos')
  .download(`${user.id}/relatorio.pdf`)

// Criar link de download no browser
if (data) {
  const url = URL.createObjectURL(data)
  const link = document.createElement('a')
  link.href = url
  link.download = 'relatorio.pdf'
  link.click()
}

// ─── LISTAR ARQUIVOS ──────────────────────────────────────────────────────────

// Listar arquivos em um "diretório" (prefixo)
const { data, error } = await supabase.storage
  .from('avatares')
  .list(user.id, {
    limit: 100,
    offset: 0,
    sortBy: { column: 'created_at', order: 'desc' },
  })

// ─── MOVER / COPIAR / DELETAR ─────────────────────────────────────────────────

// Mover arquivo (renomear ou mover de pasta)
await supabase.storage
  .from('avatares')
  .move('pasta-antiga/foto.png', 'pasta-nova/foto.png')

// Copiar arquivo
await supabase.storage
  .from('avatares')
  .copy('original.png', 'backup/original.png')

// Deletar arquivo(s)
await supabase.storage
  .from('avatares')
  .remove([`${user.id}/avatar.png`, `${user.id}/banner.png`])
```

---

## 10. Edge Functions

Edge Functions são funções serverless escritas em **TypeScript/Deno** que rodam na borda (CDN da Supabase, powered by Deno), próximas geograficamente ao usuário. São ideais para lógica que não pode ficar no frontend (pagamentos, webhooks, operações com service_role, etc.).

### 10.1 Estrutura básica

```typescript
// supabase/functions/ola-mundo/index.ts

Deno.serve(async (req: Request) => {
  // Ler dados do request
  const { nome } = await req.json()

  // Lógica da função
  const resposta = {
    mensagem: `Olá, ${nome}! Bem-vindo ao Supabase!`,
    timestamp: new Date().toISOString(),
  }

  // Retornar resposta JSON
  return new Response(
    JSON.stringify(resposta),
    {
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*', // CORS
      },
    }
  )
})
```

### 10.2 Deploy e uso via CLI

```bash
# Fazer deploy da função
supabase functions deploy ola-mundo

# Testar localmente
supabase functions serve ola-mundo

# Chamar a função via curl
curl -X POST 'https://seu-projeto.supabase.co/functions/v1/ola-mundo'   -H "Authorization: Bearer sua-anon-key"   -H "Content-Type: application/json"   -d '{"nome": "Dev"}'

# Resposta esperada:
# {"mensagem": "Olá, Dev! Bem-vindo ao Supabase!", "timestamp": "..."}
```

### 10.3 Edge Function com autenticação de usuário

```typescript
// supabase/functions/meus-dados/index.ts
import { createClient } from 'npm:@supabase/supabase-js@2'

Deno.serve(async (req: Request) => {
  // Tratar preflight CORS
  if (req.method === 'OPTIONS') {
    return new Response(null, {
      headers: {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Headers': 'authorization, x-client-info, apikey, content-type',
      },
    })
  }

  // Extrair o token de autenticação do header
  const authHeader = req.headers.get('Authorization')

  // Criar cliente Supabase passando o token do usuário
  // Isso faz com que as queries respeitem o RLS do usuário autenticado
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_ANON_KEY')!,
    {
      global: {
        headers: { Authorization: authHeader! },
      },
    }
  )

  // Verificar usuário atual
  const { data: { user }, error: authError } = await supabase.auth.getUser()

  if (authError || !user) {
    return new Response(
      JSON.stringify({ erro: 'Não autorizado' }),
      { status: 401, headers: { 'Content-Type': 'application/json' } }
    )
  }

  // Queries respeitam o RLS do usuário autenticado
  const { data: tarefas, error } = await supabase
    .from('tarefas')
    .select('*')
    .order('created_at', { ascending: false })

  return new Response(
    JSON.stringify({ usuario: user.email, tarefas }),
    { headers: { 'Content-Type': 'application/json' } }
  )
})
```

### 10.4 Edge Function com operação privilegiada (service_role)

```typescript
// supabase/functions/admin-relatorio/index.ts
// Caso de uso: gerar relatório com dados de múltiplos usuários (requer service_role)
import { createClient } from 'npm:@supabase/supabase-js@2'

Deno.serve(async (req: Request) => {
  // Cliente com service_role bypassa RLS completamente
  const supabaseAdmin = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!, // variável de ambiente segura
    {
      auth: {
        persistSession: false,
        autoRefreshToken: false,
      }
    }
  )

  // Busca dados de todos os usuários (bypassa RLS)
  const { data, error } = await supabaseAdmin
    .from('tarefas')
    .select('*, auth.users(email)')

  return new Response(JSON.stringify({ total: data?.length, data }), {
    headers: { 'Content-Type': 'application/json' },
  })
})
```

> 💡 As variáveis `SUPABASE_URL`, `SUPABASE_ANON_KEY` e `SUPABASE_SERVICE_ROLE_KEY` são automaticamente injetadas pelo Supabase nas Edge Functions — não precisa configurar manualmente.

---

## 11. Boas Práticas

### 11.1 Segurança

- **Nunca** exponha a `service_role key` no frontend — use apenas no backend/servidor
- **Sempre** habilite RLS em tabelas com dados de usuários
- Use variáveis de ambiente (`.env`) para armazenar chaves — nunca hardcode no código
- Rotacione as chaves periodicamente, especialmente a `service_role key`
- Teste as RLS policies antes de ir para produção usando diferentes roles
- Use HTTPS em produção (nunca HTTP)
- Configure CORS corretamente nas Edge Functions

### 11.2 Performance

- Use `select('coluna1, coluna2')` em vez de `select('*')` para reduzir payload
- Crie índices nas colunas usadas em filtros e JOINs:

```sql
CREATE INDEX idx_tarefas_user_id ON public.tarefas(user_id);
CREATE INDEX idx_tarefas_created_at ON public.tarefas(created_at DESC);
```

- Use o **Connection Pooler** (Supavisor) em ambientes serverless (Vercel, Netlify, AWS Lambda)
- Prefira queries com `limit()` para paginação em vez de carregar tudo
- Use `count()` para contagens em vez de carregar todos os registros

### 11.3 Organização do Projeto

```
meu-projeto/
├── src/
│   ├── lib/
│   │   └── supabaseClient.ts    # cliente único, importado em toda a app
│   └── types/
│       └── supabase.ts          # tipos gerados pelo CLI
├── supabase/
│   ├── migrations/              # arquivos SQL de migration
│   ├── functions/               # Edge Functions
│   ├── seed.sql                 # dados iniciais para desenvolvimento
│   └── config.toml              # configuração local
├── .env                         # variáveis de ambiente (não commitar!)
└── .env.example                 # exemplo sem valores sensíveis (commitar)
```

### 11.4 TypeScript com tipos gerados

```bash
# Gerar tipos automaticamente a partir do schema do banco
supabase gen types typescript --local > src/types/supabase.ts
# ou para projeto remoto:
supabase gen types typescript --project-id SEU_PROJECT_ID > src/types/supabase.ts
```

```typescript
// Usar tipos gerados no cliente
import { createClient } from '@supabase/supabase-js'
import type { Database } from './types/supabase'

export const supabase = createClient<Database>(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
)

// Agora as queries têm autocomplete e type-safety!
const { data } = await supabase
  .from('tarefas')    // ← autocomplete de tabelas
  .select('titulo')   // ← autocomplete de colunas
```

### 11.5 Gerenciar schema com Migrations

```bash
# Criar nova migration
supabase migration new adicionar_coluna_status

# Aplicar localmente
supabase db reset

# Aplicar em produção
supabase db push
```

### 11.6 Supabase CLI como ferramenta principal de migrations

Para projetos baseados em Supabase, o fluxo mais natural e recomendável e usar o **Supabase CLI** como fonte principal de verdade do schema.

**Por que faz sentido:**
- entende o ecossistema Supabase nativamente
- integra migrations, `seed.sql`, Edge Functions e tipos gerados
- funciona muito bem com Docker no ambiente local
- evita depender de SQL solto no dashboard como processo principal

**Estrutura recomendada:**

```text
meu-projeto/
├── src/
├── supabase/
│   ├── config.toml
│   ├── migrations/
│   │   ├── 20260414191500_create_prompt_manager_schema.sql
│   │   ├── 20260414192000_copy_public_to_prompt_manager.sql
│   │   └── 20260414192500_prompt_manager_rls.sql
│   ├── functions/
│   └── seed.sql
└── package.json
```

**Fluxo prático:**

```bash
# iniciar ambiente local
supabase start

# criar nova migration versionada
supabase migration new create_prompt_manager_schema

# testar tudo do zero localmente
supabase db reset

# vincular ao projeto remoto
supabase link --project-ref SEU_PROJECT_REF

# aplicar no projeto remoto
supabase db push
```

Esse modelo fica conceitualmente próximo de um projeto backend com migrations versionadas, mas usando a ferramenta oficial da plataforma.

---

## 12. Limitações e Armadilhas

### Críticas (podem expor dados)

| Armadilha | Problema | Solução |
|---|---|---|
| RLS desabilitado | Qualquer pessoa com a `anon key` lê todos os dados | Sempre habilitar RLS + criar policies |
| `service_role key` no frontend | Bypass total de segurança | Usar APENAS no backend/servidor |
| Sem `WITH CHECK` no INSERT/UPDATE | Policy de `USING` não valida dados escritos | Sempre adicionar `WITH CHECK` nas policies de escrita |

### Atenção (afetam performance/estabilidade)

- **N+1 queries**: fazer múltiplas queries em loop — use JOINs no `select()` com relações
- **Conexão direta em serverless**: use a porta do pooler (6543) em vez da porta direta (5432) em funções Lambda/Vercel/Netlify
- **Realtime sem RLS**: as subscriptions também respeitam RLS — policies de SELECT se aplicam
- **SSR e cookies de sessão**: em Next.js/Nuxt com SSR, usar `@supabase/ssr` em vez do cliente padrão para gerenciar sessão via cookies

### Limitações do plano gratuito

| Recurso | Limite Free |
|---|---|
| Banco de dados | 500 MB |
| Storage | 1 GB |
| Transferência de dados | 5 GB/mês |
| Edge Functions | 500.000 invocações/mês |
| MAUs (usuários ativos) | 50.000/mês |
| Projetos ativos | 2 |
| Pausa por inatividade | Sim (após 7 dias sem acesso ao dashboard) |

> 💡 Projetos no plano gratuito pausam automaticamente após 7 dias sem acessos. O banco continua existindo, mas fica offline até você acessar o dashboard.

---

## 13. Quando Usar e Quando Evitar

### Use Supabase quando:

- Precisa de backend completo (banco + auth + storage) sem gerenciar infraestrutura
- Quer PostgreSQL gerenciado com APIs REST/GraphQL automáticas
- Está construindo MVP, protótipo ou app de tamanho médio
- Equipe pequena que precisa de velocidade de desenvolvimento
- Aplicação frontend-only que precisa de autenticação e banco seguro
- Precisa de sincronização em tempo real (chat, dashboards ao vivo, colaboração)
- Está migrando do Firebase mas quer SQL e relações reais
- Quer open-source e a opção de self-hosting no futuro

### Evite ou avalie com cuidado quando:

- Regras de negócio extremamente complexas que não se encaixam bem em RLS
- Precisa de banco diferente de PostgreSQL (MongoDB, Cassandra, etc.)
- Compliance rígido exige controle total sobre infraestrutura (considere self-hosting)
- Volume muito alto de conexões simultâneas sem avaliação do plano
- Time grande com necessidade de queries muito customizadas (ORMs como Prisma/Drizzle são complementares, mas adicionam complexidade)

---

## 14. Alternativas Modernas

| Ferramenta | Tipo | Banco | Destaque | Quando escolher |
|---|---|---|---|---|
| **Firebase** | BaaS Google | Firestore (NoSQL) | Maturidade mobile | Apps mobile já no ecossistema Google |
| **PocketBase** | BaaS self-hosted | SQLite | Leve, binário único | Projetos pequenos, self-hosted simples |
| **Appwrite** | BaaS self-hosted | MariaDB | Open source, similar ao Supabase | Quer alternativa open-source ao Supabase |
| **Neon** | DB serverless | PostgreSQL | Branches de banco | Quer só o banco, sem auth/storage |
| **Convex** | BaaS reativo | Proprietário | TypeScript nativo | Reatividade forte com tipagem |
| **Hasura** | GraphQL engine | PostgreSQL | GraphQL automático | Time já usa GraphQL |
| **AWS Amplify** | BaaS Amazon | DynamoDB/Aurora | Ecossistema AWS | Projetos já no AWS |

**Supabase vs Firebase (resumo):**

| | Supabase | Firebase |
|---|---|---|
| Banco | PostgreSQL (relacional, SQL) | Firestore (NoSQL, documentos) |
| Open source | Sim | Não |
| Self-hosting | Sim | Não |
| JOINs e relações | Nativo | Não suporta |
| Real-time | Sim | Sim |
| Auth | Completo | Muito maduro |
| Ecossistema mobile | Crescendo | Muito maduro |

---

## 15. Ambiente Local e Self-Hosting com Docker

O Supabase pode ser executado **completamente local** — útil para desenvolvimento sem depender de internet ou conta no Supabase Cloud. Há duas abordagens:

- **Via Supabase CLI** (recomendado para desenvolvimento): mais simples, gerencia tudo automaticamente usando Docker internamente
- **Via Docker Compose direto** (recomendado para self-hosting/produção): controle total sobre os serviços e configuração

### 15.1 Desenvolvimento Local com Supabase CLI

**Pré-requisitos:**
- Docker Desktop instalado e rodando
- Node.js instalado

```bash
# Instalar o CLI
npm install -g supabase

# Na raiz do seu projeto, inicializar o Supabase
supabase init

# Baixar imagens e subir todos os serviços localmente
supabase start
```

Após iniciar, você verá a saída com todas as URLs e chaves locais:

```
Started supabase local development setup.

         API URL: http://127.0.0.1:54321
     GraphQL URL: http://127.0.0.1:54321/graphql/v1
          DB URL: postgresql://postgres:postgres@127.0.0.1:54322/postgres
      Studio URL: http://127.0.0.1:54323
    Inbucket URL: http://127.0.0.1:54324
      JWT secret: super-secret-jwt-token-with-at-least-32-characters-long
        anon key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
service_role key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

| URL | O que é |
|---|---|
| `http://127.0.0.1:54321` | API REST, Auth, Storage (Kong gateway) |
| `http://127.0.0.1:54323` | **Supabase Studio** (interface visual local) |
| `http://127.0.0.1:54324` | **Inbucket** (servidor de email fake — testa Magic Links sem enviar email real) |
| `postgresql://...54322` | Conexão direta ao PostgreSQL local |

**Apontar sua aplicação para o ambiente local:**

```bash
# .env.local da sua aplicação
VITE_SUPABASE_URL=http://127.0.0.1:54321
VITE_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
# use as chaves exibidas pelo `supabase start`
```

**Comandos essenciais do CLI:**

```bash
# Ver status atual e chaves
supabase status

# Parar todos os serviços (preserva dados)
supabase stop

# Parar e apagar todos os dados locais
supabase stop --no-backup

# Criar nova migration
supabase migration new nome_da_migration

# Aplicar migrations + seed (reseta o banco do zero)
supabase db reset

# Ver diferença entre banco local e arquivos de migration
supabase db diff

# Vincular ao projeto remoto
supabase link --project-ref SEU_PROJECT_ID

# Aplicar migrations em produção
supabase db push

# Gerar tipos TypeScript a partir do schema atual
supabase gen types typescript --local > src/types/supabase.ts

# Deploy de Edge Function para produção
supabase functions deploy nome-da-funcao

# Testar Edge Function localmente
supabase functions serve nome-da-funcao
```

**Seed data (dados de exemplo para desenvolvimento):**

```sql
-- supabase/seed.sql
-- Executado automaticamente ao rodar `supabase db reset`
INSERT INTO public.tarefas (titulo, user_id) VALUES
  ('Estudar Supabase', '00000000-0000-0000-0000-000000000001'),
  ('Configurar RLS',   '00000000-0000-0000-0000-000000000001'),
  ('Fazer deploy',     '00000000-0000-0000-0000-000000000002');
```

**Workflow de desenvolvimento recomendado:**

```bash
# 1. Iniciar stack local
supabase start

# 2. Desenvolver — usar Studio em http://127.0.0.1:54323
#    para criar tabelas, testar queries e policies visualmente

# 3. Capturar as mudanças feitas no Studio como migration
supabase db diff -f nome_da_migration

# 4. Testar o reset completo (garante que as migrations funcionam do zero)
supabase db reset

# 5. Quando pronto, enviar para produção
supabase link --project-ref SEU_PROJECT_ID
supabase db push
```

---

### 15.2 Self-Hosting com Docker Compose

Para rodar o Supabase em um servidor próprio (VPS, servidor local, on-premise) com controle total sobre a infraestrutura.

**Requisitos de sistema:**

| Recurso | Mínimo | Recomendado |
|---|---|---|
| RAM | 4 GB | 8 GB+ |
| CPU | 2 cores | 4 cores+ |
| Disco | 50 GB SSD | 80 GB+ SSD |
| OS | Linux / macOS / Windows (Docker Desktop) | Linux |

**Passo 1 — Obter os arquivos de configuração oficiais:**

```bash
# Clonar apenas a pasta docker (sem baixar o repositório inteiro)
git clone --filter=blob:none --no-checkout https://github.com/supabase/supabase
cd supabase
git sparse-checkout set --cone docker && git checkout master
cd ..

# Criar seu diretório de projeto
mkdir meu-supabase
cp -rf supabase/docker/* meu-supabase/
cp supabase/docker/.env.example meu-supabase/.env

cd meu-supabase
```

**Passo 2 — Configurar o arquivo `.env`:**

```bash
# .env — NUNCA versionar com valores reais!

# Senha do banco (use apenas letras e números para evitar problemas de URL encoding)
POSTGRES_PASSWORD=minhasenha123

# JWT Secret (mínimo 32 caracteres)
# Gerar com: openssl rand -base64 32
JWT_SECRET=sua-chave-jwt-super-secreta-com-32-chars

# Chaves derivadas do JWT_SECRET
# Gerar via: https://supabase.com/docs/guides/self-hosting/docker#generate-api-keys
ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Chaves adicionais de criptografia
# Gerar com: openssl rand -base64 48  (mínimo 64 chars)
SECRET_KEY_BASE=sua-chave-base-64-chars-minimo

# Gerar com: openssl rand -hex 16  (exatamente 32 chars)
VAULT_ENC_KEY=sua-chave-vault-32-chars-exatos

# URLs públicas (para desenvolvimento local, use localhost)
SUPABASE_PUBLIC_URL=http://localhost:8000
API_EXTERNAL_URL=http://localhost:8000
SITE_URL=http://localhost:3000

# Credenciais de acesso ao Studio (painel visual)
DASHBOARD_USERNAME=admin
DASHBOARD_PASSWORD=minha-senha-do-painel

# SMTP para envio de emails de autenticação
SMTP_ADMIN_EMAIL=admin@exemplo.com
SMTP_HOST=smtp.exemplo.com
SMTP_PORT=587
SMTP_USER=usuario_smtp
SMTP_PASS=senha_smtp
SMTP_SENDER_NAME=Meu App
```

> 💡 **Atalho**: execute `sh ./utils/generate-keys.sh` para gerar automaticamente todas as chaves criptográficas de uma vez.

**Passo 3 — Subir os serviços:**

```bash
# Baixar todas as imagens Docker
docker compose pull

# Subir em background (modo detached)
docker compose up -d

# Verificar se todos os containers estão "healthy"
docker compose ps
```

**Passo 4 — Acessar os serviços:**

| Serviço | URL padrão | Descrição |
|---|---|---|
| **Studio** | `http://localhost:8000` | Interface visual (pede usuário/senha do `.env`) |
| **REST API** | `http://localhost:8000/rest/v1/` | API automática via PostgREST |
| **Auth API** | `http://localhost:8000/auth/v1/` | Endpoints de autenticação |
| **Storage API** | `http://localhost:8000/storage/v1/` | Upload e gestão de arquivos |
| **Realtime** | `http://localhost:8000/realtime/v1/` | WebSockets |
| **GraphQL** | `http://localhost:8000/graphql/v1` | API GraphQL |
| **PostgreSQL** | `localhost:5432` (session) / `localhost:6543` (pooled) | Banco de dados |

**Arquitetura dos containers:**

```
Kong (API Gateway) :8000
  ├── PostgREST    (REST API automática)
  ├── GoTrue       (Auth)
  ├── Storage API  (arquivos)
  ├── Realtime     (WebSockets, Elixir)
  ├── Studio       (dashboard visual)
  └── Edge Runtime (funções Deno)
        │
PostgreSQL + Supavisor (connection pooler)
        │
Logflare + Vector (logs e analytics)
```

**Conectar sua aplicação ao Supabase self-hosted:**

```javascript
// .env da sua aplicação
SUPABASE_URL=http://localhost:8000       // ou IP/domínio do servidor
SUPABASE_ANON_KEY=valor-do-ANON_KEY-do-env-docker

// supabaseClient.js — funciona exatamente igual ao Supabase Cloud!
import { createClient } from '@supabase/supabase-js'

export const supabase = createClient(
  process.env.SUPABASE_URL,
  process.env.SUPABASE_ANON_KEY
)
```

**Comandos de gerenciamento do dia a dia:**

```bash
# Parar todos os serviços (preserva dados nos volumes)
docker compose down

# Ver logs em tempo real de um serviço
docker compose logs -f auth
docker compose logs -f db
docker compose logs -f kong

# Reiniciar um serviço específico (ex: após adicionar nova Edge Function)
docker compose restart functions --no-deps

# Atualizar para versão mais recente
docker compose pull
docker compose down && docker compose up -d

# Alterar senha do banco após setup inicial
sh ./utils/db-passwd.sh
docker compose up -d --force-recreate

# Remover TUDO incluindo dados (irreversível!)
docker compose down -v
rm -rf volumes/db/data
rm -rf volumes/storage
```

**Adicionar Edge Function no self-hosting:**

```bash
# Criar a função em volumes/functions/
mkdir -p volumes/functions/minha-funcao

# Criar o arquivo index.ts
cat > volumes/functions/minha-funcao/index.ts << 'EOF'
Deno.serve(async (req) => {
  return new Response(
    JSON.stringify({ status: "ok", timestamp: new Date().toISOString() }),
    { headers: { "Content-Type": "application/json" } }
  )
})
EOF

# Reiniciar o serviço de functions para carregar a nova função
docker compose restart functions --no-deps

# Testar
curl http://localhost:8000/functions/v1/minha-funcao \
  -H "Authorization: Bearer SEU_ANON_KEY"
```

---

### 15.3 CLI local vs Docker Compose — quando usar cada um

| Situação | CLI (`supabase start`) | Docker Compose |
|---|---|---|
| Desenvolvimento local | Ideal | Funciona, mas mais complexo |
| Self-hosting em produção | Não adequado | Ideal |
| VPS / servidor dedicado | Não | Sim |
| Configuração inicial | Muito rápida | Requer configuração manual |
| Controle sobre versões dos serviços | Via CLI update | Total (editar docker-compose.yml) |
| Migrations e seed integrados | Nativo | Manual |
| CI/CD pipeline | Via `supabase db push` | Configuração adicional necessária |

---

## 16. Supabase CLI vs Flyway vs Docker

Essas ferramentas nao competem diretamente no mesmo nivel. Elas resolvem problemas diferentes e podem coexistir.

### 16.1 O papel de cada uma

| Ferramenta | Papel principal | Melhor uso |
|---|---|---|
| **Supabase CLI** | Tooling nativo do ecossistema Supabase | migrations, ambiente local, functions, tipos, seed |
| **Flyway** | Versionamento e execucao de migrations SQL | times com cultura forte de backend/Java e pipeline SQL-first |
| **Docker** | Runtime local / conteinerizacao | subir stack local do Supabase ou self-hosting |

### 16.2 Quando usar apenas Supabase CLI

Use apenas o Supabase CLI quando:
- o banco principal e Supabase/Postgres
- o projeto usa recursos nativos como Auth, RLS, Edge Functions e seed local
- voce quer o caminho mais simples e suportado pela plataforma
- o ambiente local em Docker ja atende sua necessidade

Esse e o melhor padrao para a maioria dos projetos frontend + Supabase.

### 16.3 Quando Flyway ainda faz sentido

Flyway continua fazendo sentido quando:
- seu modelo mental e de backend Java com migrations rigidamente controladas
- voce quer padronizar o mesmo processo em projetos Java e Supabase
- o time ja usa convenções como `V1__`, `V2__`, `R__` e quer manter consistencia
- o banco sera gerenciado por pipeline SQL corporativo fora do workflow nativo do Supabase

**Pontos de atencao ao usar Flyway com Supabase:**
- conecte via PostgreSQL, nao via REST API
- foque nos schemas da aplicacao (`public`, `app`, `prompt_manager` etc.)
- evite tentar gerenciar internals do Supabase (`auth`, `storage`) sem necessidade
- mantenha claro qual ferramenta e a fonte oficial da verdade

### 16.4 Comparativo prático para dev Java

| Tema | Supabase CLI | Flyway |
|---|---|---|
| Criar migration | `supabase migration new nome` | criar `Vx__descricao.sql` |
| Aplicar migrations | `supabase db push` | `flyway migrate` |
| Resetar ambiente local | `supabase db reset` | recriar banco + `flyway migrate` |
| Seed local | `supabase/seed.sql` | migration repeatable ou script separado |
| Integração com features Supabase | nativa | indireta |
| Familiar para time Java | media | alta |

Para um dev backend Java, o Supabase CLI entrega um fluxo parecido com Flyway, mas mais alinhado com a plataforma.

### 16.5 Docker no contexto certo

Docker nao substitui Flyway nem Supabase CLI.

Use Docker para:
- rodar o stack local do Supabase via `supabase start`
- self-hosting com Docker Compose
- ambientes reproduziveis em desenvolvimento

Nao use Docker sozinho como estrategia de migrations. Ele sobe os servicos, mas nao organiza o versionamento do schema por si so.

### 16.6 Recomendação objetiva

Para projetos Supabase mantidos por um desenvolvedor com background forte em Java:

1. **Padrão recomendado:** Supabase CLI + Docker
2. **Opcional:** Flyway apenas se houver necessidade real de padronizacao com projetos Java
3. **Evitar:** usar dashboard/manual SQL como fonte principal de schema em projetos que evoluem com frequencia

### 16.7 Estratégia recomendada para múltiplas POCs

Se voce pretende ter varias POCs no mesmo projeto Supabase:
- usar **um schema por POC**
- versionar a criacao de cada schema com migration
- testar localmente com CLI + Docker
- aplicar remotamente com `supabase db push`

Exemplo:

```text
supabase/migrations/
  20260414191500_create_prompt_manager_schema.sql
  20260414192000_copy_public_to_prompt_manager.sql
  20260414192500_prompt_manager_rls.sql
  20260415100000_create_outro_poc_schema.sql
```

---

## 17. Resumo Final

### O que o Supabase oferece:

| Produto | Descrição | Tecnologia subjacente |
|---|---|---|
| **Database** | PostgreSQL completo com API REST e GraphQL automáticos | PostgreSQL + PostgREST |
| **Auth** | Email/senha, magic link, OAuth, MFA | GoTrue (Go) |
| **Storage** | Upload e gestão de arquivos com RLS integrado | MinIO / S3 compatible |
| **Realtime** | Subscriptions a mudanças e broadcast entre clientes | Elixir (Phoenix) |
| **Edge Functions** | Funções serverless distribuídas globalmente | Deno |

### Cheat Sheet rápido:

```javascript
// INICIALIZAR
const supabase = createClient(SUPABASE_URL, ANON_KEY)

// AUTH
await supabase.auth.signUp({ email, password })
await supabase.auth.signInWithPassword({ email, password })
await supabase.auth.signInWithOtp({ email })               // Magic Link
await supabase.auth.signInWithOAuth({ provider: 'google' })
await supabase.auth.signOut()
const { data: { user } } = await supabase.auth.getUser()

// DATABASE
await supabase.from('t').select('*')
await supabase.from('t').select('col1, col2').eq('campo', valor).limit(10)
await supabase.from('t').insert({ campo: valor })
await supabase.from('t').update({ campo: valor }).eq('id', id)
await supabase.from('t').delete().eq('id', id)
await supabase.from('t').upsert({ id, campo: valor })

// STORAGE
await supabase.storage.from('bucket').upload('path', file)
supabase.storage.from('bucket').getPublicUrl('path')
await supabase.storage.from('bucket').download('path')
await supabase.storage.from('bucket').remove(['path'])

// REALTIME
supabase
  .channel('nome')
  .on('postgres_changes', { event: '*', schema: 'public', table: 't' }, callback)
  .subscribe()
```

### Regra de ouro das chaves:

```
anon key      → use no FRONTEND  (com RLS ativo)
service key   → use no BACKEND   (NUNCA no browser)
```

### Decisão rápida de ambiente:

```
Desenvolvendo localmente?
  → supabase start          (CLI sobe Docker automaticamente)

Fazer deploy de mudanças no banco?
  → supabase db push

Hospedar em servidor próprio?
  → docker compose up -d    (Docker Compose com .env configurado)

Usar como serviço gerenciado na nuvem?
  → supabase.com/dashboard  (plano free disponível)
```

---

*Documentação gerada com base na documentação oficial do Supabase (supabase.com/docs). Última revisão: abril/2026.*
