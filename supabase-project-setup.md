# Supabase — Project Setup (React + TypeScript)

> Guia de configuração do Supabase para um projeto frontend React/TypeScript conectando diretamente ao banco, sem backend próprio. Baseado no projeto **prompt-manager** — substitua os dados da seção 1 para reutilizar em outros projetos.
>
> **Última atualização:** Abril 2026

---

## Índice

1. [Dados do Projeto](#1-dados-do-projeto)
2. [Chaves de API](#2-chaves-de-api)
3. [Criar Projeto no Supabase](#3-criar-projeto-no-supabase)
4. [Configurar o SDK no Frontend](#4-configurar-o-sdk-no-frontend)
5. [Criar Tabelas no Banco](#5-criar-tabelas-no-banco)
6. [Habilitar RLS e Policies](#6-habilitar-rls-e-policies)
7. [Configurar Google OAuth](#7-configurar-google-oauth)
8. [Implementar Auth no React](#8-implementar-auth-no-react)
9. [CRUD com supabase-js](#9-crud-com-supabase-js)
10. [Conexão via Cliente de Banco (DataGrip, DBeaver, psql)](#10-conexão-via-cliente-de-banco-datagrip-dbeaver-psql)
11. [Supabase CLI, Flyway e Docker](#11-supabase-cli-flyway-e-docker)
12. [Checklist Completo](#12-checklist-completo)
13. [Links Rápidos do Projeto](#13-links-rápidos-do-projeto)

---

## 1. Dados do Projeto

> Atualize esta seção ao reutilizar o guia em um novo projeto.

| Campo | Valor |
|---|---|
| **Project name** | `prompt-manager` |
| **Project ref** | `fwkpzfmguweykwkpqald` |
| **Project URL** | `https://fwkpzfmguweykwkpqald.supabase.co` |
| **Região** | South America (`sa-east-1`) |
| **Dashboard** | https://supabase.com/dashboard/project/fwkpzfmguweykwkpqald |

---

## 2. Chaves de API

> Obter em: **Dashboard → Settings → API Keys**

| Chave | Formato | Onde usar |
|---|---|---|
| **Publishable key** | `sb_publishable_...` | Frontend — segura com RLS ativo |
| **Anon key** (legacy) | JWT longo | Frontend — ainda funciona, formato antigo |
| **Secret key** | `sb_secret_...` | Backend/servidor **apenas** — bypassa RLS |
| **Service role** (legacy) | JWT longo | Backend/servidor **apenas** — bypassa RLS |

> ⚠️ **Nunca** exponha a secret/service_role key no frontend. Nunca commite valores reais no repositório.
>
> O Supabase recomenda migrar para `publishable`/`secret` keys (novo formato) pois permitem rotação independente do JWT secret. As `anon`/`service_role` ainda funcionam mas são consideradas legacy.

---

## 3. Criar Projeto no Supabase

1. Acesse https://supabase.com → **Start your project** (login com GitHub ou Google)
2. Clique em **New Project**
3. Preencha:
   - **Project name:** nome do projeto
   - **Database password:** senha forte — guarde-a, será usada em conexões diretas ao banco
   - **Region:** `South America (sa-east-1)` para Brasil
4. Clique **Create new project** e aguarde ~2 minutos

---

## 4. Configurar o SDK no Frontend

### Instalação

```bash
npm install @supabase/supabase-js
```

### Variáveis de ambiente

```bash
# .env
VITE_SUPABASE_URL=https://fwkpzfmguweykwkpqald.supabase.co
VITE_SUPABASE_PUBLISHABLE_KEY=sb_publishable_sua-key-aqui
```

```bash
# .env.example — commitar este, não o .env
VITE_SUPABASE_URL=https://SEU_PROJECT_REF.supabase.co
VITE_SUPABASE_PUBLISHABLE_KEY=sb_publishable_sua-key-aqui
```

> Para **Next.js**, use o prefixo `NEXT_PUBLIC_` em vez de `VITE_`.

### .gitignore

```
.env
.env.local
.env.production
```

### Cliente Supabase

```typescript
// src/lib/supabaseClient.ts
import { createClient } from '@supabase/supabase-js'

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL
const supabaseKey = import.meta.env.VITE_SUPABASE_PUBLISHABLE_KEY

if (!supabaseUrl || !supabaseKey) {
  throw new Error('Missing VITE_SUPABASE_URL or VITE_SUPABASE_PUBLISHABLE_KEY')
}

export const supabase = createClient(supabaseUrl, supabaseKey)
```

```typescript
// uso em qualquer arquivo
import { supabase } from '@/lib/supabaseClient'
```

---

## 5. Criar Tabelas no Banco

Use o **SQL Editor**: Dashboard → SQL Editor → New query

### Tabela com isolamento por usuário

```sql
CREATE TABLE public.prompts (
    id          BIGINT      GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    title       TEXT        NOT NULL,
    slug        TEXT        NOT NULL UNIQUE,
    content     TEXT        NOT NULL,
    group_name  TEXT        NOT NULL DEFAULT 'Geral',
    user_id     UUID        NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### Tabela compartilhada (sem user_id)

```sql
CREATE TABLE public.tags (
    id   BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name TEXT   NOT NULL UNIQUE
);
```

### Tabela de junção (many-to-many)

```sql
CREATE TABLE public.prompt_tags (
    prompt_id BIGINT NOT NULL REFERENCES public.prompts(id) ON DELETE CASCADE,
    tag_id    BIGINT NOT NULL REFERENCES public.tags(id)    ON DELETE CASCADE,
    PRIMARY KEY (prompt_id, tag_id)
);
```

### Índices recomendados

```sql
CREATE INDEX idx_prompts_user_id    ON public.prompts(user_id);
CREATE INDEX idx_prompts_slug       ON public.prompts(slug);
CREATE INDEX idx_prompts_group_name ON public.prompts(group_name);
CREATE INDEX idx_prompt_tags_tag    ON public.prompt_tags(tag_id);
```

### Tipos de dados comuns

| Tipo | Uso |
|---|---|
| `BIGINT GENERATED ALWAYS AS IDENTITY` | ID auto-incremento |
| `TEXT` | Strings de qualquer tamanho |
| `UUID` | IDs de usuário (`auth.users`) |
| `TIMESTAMPTZ` | Data/hora com timezone |
| `BOOLEAN` | true/false |
| `JSONB` | JSON indexável |

---

## 6. Habilitar RLS e Policies

> **RLS é obrigatório.** Sem ele, qualquer pessoa com a publishable key acessa todos os dados.

### Habilitar na tabela

```sql
ALTER TABLE public.prompts ENABLE ROW LEVEL SECURITY;
```

> Sem policies criadas após habilitar o RLS, **ninguém acessa nada** (default deny).

### Padrão A — Dados isolados por usuário (mais comum)

```sql
CREATE POLICY "prompts_select" ON public.prompts
    FOR SELECT TO authenticated
    USING (auth.uid() = user_id);

CREATE POLICY "prompts_insert" ON public.prompts
    FOR INSERT TO authenticated
    WITH CHECK (auth.uid() = user_id);

CREATE POLICY "prompts_update" ON public.prompts
    FOR UPDATE TO authenticated
    USING (auth.uid() = user_id)
    WITH CHECK (auth.uid() = user_id);

CREATE POLICY "prompts_delete" ON public.prompts
    FOR DELETE TO authenticated
    USING (auth.uid() = user_id);
```

### Padrão B — Tabela compartilhada (ex: tags)

```sql
CREATE POLICY "tags_select" ON public.tags FOR SELECT TO authenticated USING (true);
CREATE POLICY "tags_insert" ON public.tags FOR INSERT TO authenticated WITH CHECK (true);
CREATE POLICY "tags_update" ON public.tags FOR UPDATE TO authenticated USING (true) WITH CHECK (true);
CREATE POLICY "tags_delete" ON public.tags FOR DELETE TO authenticated USING (true);
```

### Padrão C — Tabela de junção (acesso via tabela pai)

```sql
CREATE POLICY "prompt_tags_select" ON public.prompt_tags
    FOR SELECT TO authenticated
    USING (
        EXISTS (
            SELECT 1 FROM public.prompts
            WHERE prompts.id = prompt_tags.prompt_id
              AND prompts.user_id = auth.uid()
        )
    );
```

### Padrão D — Acesso total para testes (não usar em produção)

```sql
CREATE POLICY "acesso_total_testes" ON public.minha_tabela
    FOR ALL USING (true) WITH CHECK (true);
```

### Referência rápida

| Cláusula | Controla |
|---|---|
| `USING (...)` | Leitura — SELECT, UPDATE (linhas existentes), DELETE |
| `WITH CHECK (...)` | Escrita — INSERT, UPDATE (valores novos) |
| `TO authenticated` | Só usuários logados |
| `TO anon` | Só requisições sem login |
| `auth.uid()` | UUID do usuário logado |

---

## 7. Configurar Google OAuth

### Passo 1 — Google Cloud Console

1. Acesse https://console.cloud.google.com/
2. Crie ou selecione um projeto

#### OAuth Consent Screen

3. **APIs & Services → OAuth consent screen**
   (direto: https://console.cloud.google.com/apis/credentials/consent)
4. Selecione **Externo**
5. Preencha: App name, User support email, Developer contact email
6. Avance nas telas de Scopes (sem adicionar nada)
7. Em **Test users**, adicione seu email (em modo Testing, só emails listados conseguem logar)
8. Finalize

#### Criar credenciais OAuth

9. **APIs & Services → Credentials**
   (direto: https://console.cloud.google.com/apis/credentials)
10. **+ Create Credentials → OAuth client ID**
11. Application type: **Web application**
12. Preencha:

    **Authorized JavaScript origins:**
    ```
    http://localhost:5173
    ```

    **Authorized redirect URIs:**
    ```
    https://fwkpzfmguweykwkpqald.supabase.co/auth/v1/callback
    ```

13. Clique **Create** e copie o **Client ID** e **Client Secret**

> Para publicar para todos os usuários (sair do modo Testing), é necessário verificar o OAuth Consent Screen com o Google.

### Passo 2 — Supabase Dashboard

1. **Authentication → Providers → Google**
   (direto: https://supabase.com/dashboard/project/fwkpzfmguweykwkpqald/auth/providers)
2. Ative **Enable Sign in with Google**
3. Cole o **Client ID** e o **Client Secret**
4. Clique **Save**

---

## 8. Implementar Auth no React

### Estrutura de arquivos

```
src/
  lib/
    supabaseClient.ts
  contexts/
    authTypes.ts         ← tipos + criação do contexto
    AuthContext.tsx       ← AuthProvider
  hooks/
    useAuth.ts           ← hook de consumo
  components/
    ProtectedRoute.tsx   ← wrapper de rota protegida
  pages/
    LoginPage.tsx        ← tela de login
```

### authTypes.ts

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

### AuthContext.tsx

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
    // Sessão inicial
    supabase.auth.getSession().then(({ data: { session } }) => {
      setSession(session)
      setUser(session?.user ?? null)
      setLoading(false)
    })

    // Escutar mudanças (login, logout, token refresh)
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

### useAuth.ts

```typescript
import { useContext } from 'react'
import { AuthContext } from '../contexts/authTypes'

export function useAuth() {
  const context = useContext(AuthContext)
  if (!context) throw new Error('useAuth must be used within an AuthProvider')
  return context
}
```

### ProtectedRoute.tsx

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

### LoginPage.tsx

```typescript
import { useEffect } from 'react'
import { useNavigate } from 'react-router-dom'
import { useAuth } from '../hooks/useAuth'

export function LoginPage() {
  const { user, loading, signInWithGoogle } = useAuth()
  const navigate = useNavigate()

  useEffect(() => {
    if (user) navigate('/', { replace: true })
  }, [user, navigate])

  if (loading) return <div>Carregando...</div>

  return (
    <div>
      <h1>Login</h1>
      <button onClick={signInWithGoogle}>Entrar com Google</button>
    </div>
  )
}
```

### App.tsx

```typescript
import { BrowserRouter, Routes, Route } from 'react-router-dom'
import { AuthProvider } from './contexts/AuthContext'
import { ProtectedRoute } from './components/ProtectedRoute'
import { LoginPage } from './pages/LoginPage'
import { HomePage } from './pages/HomePage'

function App() {
  return (
    <BrowserRouter>
      <AuthProvider>
        <Routes>
          <Route path="/login" element={<LoginPage />} />
          <Route
            path="/*"
            element={
              <ProtectedRoute>
                <Routes>
                  <Route path="/" element={<HomePage />} />
                  {/* outras rotas protegidas */}
                </Routes>
              </ProtectedRoute>
            }
          />
        </Routes>
      </AuthProvider>
    </BrowserRouter>
  )
}
```

### Acessar dados do usuário logado

```typescript
import { useAuth } from '../hooks/useAuth'

function Header() {
  const { user, signOut } = useAuth()

  return (
    <header>
      <img src={user?.user_metadata?.avatar_url} alt="Avatar" />
      <span>{user?.user_metadata?.full_name || user?.email}</span>
      <button onClick={signOut}>Sair</button>
    </header>
  )
}
```

### Obter user_id para inserts

```typescript
const { data: { user } } = await supabase.auth.getUser()
if (!user) throw new Error('Usuário não autenticado')

await supabase.from('prompts').insert({
  title:   'Meu prompt',
  content: 'Conteúdo...',
  user_id: user.id,   // UUID do usuário logado
})
```

---

## 9. CRUD com supabase-js

### SELECT

```typescript
// Todos os registros
const { data, error } = await supabase
  .from('prompts')
  .select('*')
  .order('updated_at', { ascending: false })

// Com filtro
const { data } = await supabase
  .from('prompts')
  .select('*')
  .eq('group_name', 'Desenvolvimento')

// Busca por texto
const { data } = await supabase
  .from('prompts')
  .select('*')
  .or('title.ilike.%termo%,content.ilike.%termo%')

// Registro único
const { data } = await supabase
  .from('prompts')
  .select('*')
  .eq('slug', 'meu-prompt')
  .single()

// Com JOIN
const { data } = await supabase
  .from('prompt_tags')
  .select('prompt_id, tag_id, tags(name)')
  .eq('prompt_id', 1)
```

### INSERT

```typescript
const { data, error } = await supabase
  .from('prompts')
  .insert({
    title:      'Novo prompt',
    slug:       'novo-prompt',
    content:    'Conteúdo aqui',
    group_name: 'Geral',
    user_id:    user.id,
  })
  .select()
  .single()
```

### UPDATE

```typescript
const { data, error } = await supabase
  .from('prompts')
  .update({
    title:      'Título atualizado',
    updated_at: new Date().toISOString(),
  })
  .eq('id', 42)
  .select()
  .single()
```

### DELETE

```typescript
const { error } = await supabase
  .from('prompts')
  .delete()
  .eq('id', 42)
```

---

## 10. Conexão via Cliente de Banco (DataGrip, DBeaver, psql)

> Para o **frontend com `supabase-js`**, esta seção não se aplica — o SDK usa HTTPS e funciona em qualquer rede sem configuração adicional.
>
> Esta seção é exclusiva para clientes de banco de dados (DataGrip, DBeaver, psql, ORMs rodando no servidor).

### Por que usar o Session Pooler

A conexão direta ao PostgreSQL (`db.[ref].supabase.co:5432`) exige **IPv6**, que a maioria das redes domésticas e corporativas não suporta. O Session Pooler aceita **IPv4** gratuitamente.

| Tipo | Host | Porta | IPv4 |
|---|---|---|---|
| Conexão direta | `db.fwkpzfmguweykwkpqald.supabase.co` | 5432 | ❌ Exige IPv6 |
| Session Pooler | `aws-1-sa-east-1.pooler.supabase.com` | 5432 | ✅ |
| Transaction Pooler | `aws-1-sa-east-1.pooler.supabase.com` | 6543 | ✅ |

> Para IDEs e ferramentas de banco, use sempre o **Session Pooler (porta 5432)**.

### Dados de conexão

| Campo | Valor |
|---|---|
| **Host** | `aws-1-sa-east-1.pooler.supabase.com` |
| **Port** | `5432` |
| **Database** | `postgres` |
| **User** | `postgres.fwkpzfmguweykwkpqald` |
| **Password** | senha definida na criação do projeto |
| **SSL** | Required |

```
# Connection string completa
postgresql://postgres.fwkpzfmguweykwkpqald:[SENHA]@aws-1-sa-east-1.pooler.supabase.com:5432/postgres
```

> ⚠️ O user no pooler é `postgres.fwkpzfmguweykwkpqald` — não apenas `postgres`.

---

## 11. Supabase CLI, Flyway e Docker

### 11.1 Recomendação prática

Para projetos React/TypeScript conectados ao Supabase, o melhor padrão tende a ser:

- **Supabase CLI** como ferramenta oficial de migrations e ambiente local
- **Docker** como runtime local da stack Supabase
- **Flyway** apenas se houver necessidade de alinhar o processo com projetos Java/backend do time

### 11.2 Setup mínimo na máquina

| Ferramenta | Obrigatória | Uso |
|---|---|---|
| **Supabase CLI** | Sim, se for usar migrations nativas | criar migrations, resetar banco, push remoto |
| **Docker Desktop** | Sim, para ambiente local completo | rodar Postgres, Studio, Auth, Storage, Realtime |
| **Flyway** | Opcional | padronização com times Java e pipelines SQL-first |

### 11.3 Fluxo recomendado com Supabase CLI + Docker

```bash
# inicializar estrutura local
supabase init

# subir stack local completa
supabase start

# criar migration nova
supabase migration new create_prompt_manager_schema

# testar migrations + seed do zero
supabase db reset

# vincular ao projeto remoto
supabase link --project-ref fwkpzfmguweykwkpqald

# aplicar migrations pendentes no remoto
supabase db push
```

### 11.4 Estrutura recomendada no repositório

```text
supabase/
  config.toml
  migrations/
    20260414191500_create_prompt_manager_schema.sql
    20260414192000_copy_public_to_prompt_manager.sql
    20260414192500_prompt_manager_rls.sql
  seed.sql
  functions/
```

### 11.5 Onde o Flyway entra

Se voce vem de Java e quer um workflow parecido com backend tradicional, o Flyway pode ser usado para controlar o schema do PostgreSQL do Supabase.

**Quando vale a pena:**
- varios projetos Java ja usam Flyway
- o time quer manter convencoes `V1__`, `V2__`, `R__`
- existe pipeline corporativo centralizado de banco

**Quando nao vale:**
- o projeto e essencialmente Supabase-first
- voce quer reduzir ferramentas e seguir o caminho nativo da plataforma
- Auth, RLS, seed, functions e ambiente local giram em torno do CLI

### 11.6 Comparativo direto

| Tema | Supabase CLI | Flyway |
|---|---|---|
| Experiencia nativa Supabase | Alta | Baixa |
| Familiaridade para Java backend | Media | Alta |
| Migrations SQL versionadas | Sim | Sim |
| Ambiente local integrado | Sim, com Docker | Nao |
| Edge Functions / tipos / seed | Sim | Nao |
| Melhor escolha para este tipo de projeto | Sim | Opcional |

### 11.7 Recomendação para este guia

Para o caso do `prompt-manager` e para futuras POCs:

1. usar `Supabase CLI + Docker` como padrão oficial
2. versionar cada mudança de schema em `supabase/migrations/`
3. manter `docs/*.sql` como documentação e referência, nao como fonte principal de verdade
4. considerar Flyway apenas se a prioridade for unificar o processo com projetos Java

---

## 12. Checklist Completo

### Supabase Dashboard
- [ ] Projeto criado com região `sa-east-1`
- [ ] Publishable key obtida (`sb_publishable_...`) em Settings → API Keys
- [ ] Tabelas criadas no SQL Editor
- [ ] RLS habilitado em **todas** as tabelas
- [ ] Policies RLS criadas para cada operação
- [ ] Google OAuth ativado em Authentication → Providers → Google

### Google Cloud Console
- [ ] Projeto criado
- [ ] OAuth Consent Screen configurado (Externo)
- [ ] Seu email adicionado como Test User
- [ ] OAuth Client ID criado (Web application)
- [ ] Authorized JavaScript origins: `http://localhost:5173`
- [ ] Authorized redirect URIs: `https://fwkpzfmguweykwkpqald.supabase.co/auth/v1/callback`
- [ ] Client ID e Client Secret salvos no Supabase Dashboard

### Frontend
- [ ] `@supabase/supabase-js` instalado
- [ ] `.env` com `VITE_SUPABASE_URL` e `VITE_SUPABASE_PUBLISHABLE_KEY`
- [ ] `.env` no `.gitignore`
- [ ] `.env.example` commitado com placeholders
- [ ] `supabaseClient.ts` criado e exportando o cliente
- [ ] `AuthProvider` envolvendo toda a aplicação no `App.tsx`
- [ ] `ProtectedRoute` protegendo rotas que exigem login
- [ ] `LoginPage` com botão de login Google
- [ ] `user_id` sendo preenchido nos inserts
- [ ] Logout funcionando

### Supabase CLI / Docker
- [ ] `supabase --version` funcionando na máquina
- [ ] Docker Desktop instalado e em execução
- [ ] `supabase init` executado no projeto
- [ ] `supabase start` sobe o ambiente local
- [ ] `supabase migration new ...` gera arquivos versionados em `supabase/migrations/`
- [ ] `supabase db reset` recria o banco local com sucesso
- [ ] `supabase link --project-ref ...` configurado para o projeto remoto
- [ ] `supabase db push` aplicado com sucesso no remoto

---

## 13. Links Rápidos do Projeto

| Recurso | URL |
|---|---|
| Dashboard | https://supabase.com/dashboard/project/fwkpzfmguweykwkpqald |
| SQL Editor | https://supabase.com/dashboard/project/fwkpzfmguweykwkpqald/sql/new |
| Table Editor | https://supabase.com/dashboard/project/fwkpzfmguweykwkpqald/editor |
| Auth / Usuários | https://supabase.com/dashboard/project/fwkpzfmguweykwkpqald/auth/users |
| Auth Providers | https://supabase.com/dashboard/project/fwkpzfmguweykwkpqald/auth/providers |
| Storage | https://supabase.com/dashboard/project/fwkpzfmguweykwkpqald/storage/buckets |
| API Keys | https://supabase.com/dashboard/project/fwkpzfmguweykwkpqald/settings/api |
| Logs | https://supabase.com/dashboard/project/fwkpzfmguweykwkpqald/logs/explorer |
| Google Cloud Console | https://console.cloud.google.com/ |
| OAuth Credentials | https://console.cloud.google.com/apis/credentials |
