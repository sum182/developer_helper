# Vercel — Guia de Uso (JavaScript / TypeScript + Next.js)

---

## 1. Introdução

### Contexto do tema

Vercel é uma plataforma de deploy e hospedagem em nuvem criada pela mesma equipe do Next.js. Ela foi projetada para simplificar o ciclo completo de desenvolvimento front-end moderno: do commit ao deploy em produção, passando por ambientes de preview, monitoramento e otimização de performance.

A plataforma opera sobre uma **Edge Network global** (CDN distribuída), suporta **Serverless Functions** e **Edge Functions**, e possui integração nativa e profunda com o ecossistema Next.js.

### Problema que resolve

Antes de plataformas como a Vercel, fazer deploy de uma aplicação Next.js com SSR, ISR e API Routes exigia configurar servidores Node.js, gerenciar infraestrutura, configurar proxies reversos (Nginx/Caddy), lidar com SSL manualmente e criar pipelines de CI/CD do zero.

A Vercel abstrai toda essa complexidade:

- **Zero configuração** para projetos Next.js
- **Deploy automático** a cada push no repositório
- **Preview por pull request**, sem esforço adicional
- **Escalabilidade automática** via Serverless

### Quando faz sentido aplicar

| Cenário | Vercel é boa opção? |
|---|---|
| Apps Next.js com SSR/SSG/ISR | ✅ Sim — suporte nativo |
| Sites estáticos (Jamstack) | ✅ Sim — deploy rápido e CDN global |
| APIs leves e endpoints serverless | ✅ Sim — com ressalvas de cold start |
| SPA pura (React sem Next.js) | ✅ Sim — funciona bem |
| Backend pesado (longa duração, WebSockets persistentes) | ❌ Não ideal |
| Banco de dados self-hosted | ❌ Não — use serviços externos |
| Aplicações com requisitos de compliance rígidos (dados on-premise) | ⚠️ Avaliar com cautela |

---

## 2. Sintaxe Básica

### Conceitos essenciais

**Projeto:** Unidade de deploy na Vercel. Cada repositório Git conectado vira um projeto.

**Deployment:** Cada push gera um deployment único com URL imutável (ex: `meu-app-abc123.vercel.app`).

**Production Deployment:** O deployment vinculado ao branch principal (`main`/`master`), acessível pelo domínio de produção.

**Preview Deployment:** Gerado automaticamente para branches e pull requests.

**Serverless Function:** Função backend executada sob demanda, sem servidor dedicado.

**Edge Function:** Função executada na borda da rede (edge), mais próxima do usuário, com latência ultrabaixa — mas com runtime limitado (sem Node.js completo).

---

### Exemplo mínimo funcional

**Estrutura mínima de um projeto Next.js para Vercel:**

```
meu-projeto/
├── app/
│   ├── page.tsx          # Página principal
│   └── api/
│       └── hello/
│           └── route.ts  # API Route
├── public/
├── next.config.ts
├── package.json
└── vercel.json           # Opcional
```

**`app/page.tsx`**
```tsx
export default function Home() {
  return <h1>Hello, Vercel!</h1>;
}
```

**`app/api/hello/route.ts`**
```ts
import { NextResponse } from 'next/server';

export async function GET() {
  return NextResponse.json({ message: 'Hello from Vercel API!' });
}
```

**Deploy via CLI:**
```bash
# Instalar CLI
npm i -g vercel

# Login
vercel login

# Deploy (na raiz do projeto)
vercel

# Deploy direto para produção
vercel --prod
```

---

### Setup completo da CLI no Windows

Passo a passo testado em Windows 11 com PowerShell, do zero até estar autenticado.

#### Pré-requisitos

A CLI é distribuída via npm, então o Node.js precisa estar instalado. Confirme antes de prosseguir:

```powershell
node --version   # Esperado: v20.x (LTS) ou superior
npm --version    # Esperado: 10.x ou superior
```

Se algum dos comandos falhar, instale o Node.js LTS em https://nodejs.org. No Windows, é recomendado usar `nvm-windows` (https://github.com/coreybutler/nvm-windows) para evitar problemas de permissão em instalações globais.

#### Onde abrir o terminal

Qualquer um destes funciona:

- `Win + X` → **Terminal** (abre PowerShell)
- Explorador de Arquivos → botão direito dentro da pasta → **Abrir no Terminal**
- VS Code com a pasta aberta → `` Ctrl + ` `` (crase) abre terminal integrado na raiz

Para os comandos de instalação e login a pasta não importa (são globais). A partir do `vercel link` em diante, é necessário estar **dentro da pasta do projeto**.

#### Instalação

```powershell
# Instala a CLI globalmente
npm i -g vercel

# Confirma a versão instalada
vercel --version
```

Se aparecer erro de permissão (`EACCES` / `EPERM`):

- Abra o PowerShell **como administrador** (`Win` → digite `PowerShell` → botão direito → **Executar como administrador**) e rode `npm i -g vercel` de novo.
- Ou troque para `nvm-windows`, que instala o Node por usuário e elimina esse atrito.

#### Autenticação

```powershell
# Inicia o fluxo de login (abre o navegador)
vercel login
```

Alternativamente, qualquer comando que exija autenticação (como `vercel whoami`) detecta a ausência de credenciais e inicia o login automaticamente.

O fluxo é via **device code**:

1. O terminal exibe uma URL no formato `https://vercel.com/oauth/device?user_code=XXXX-XXXX`
2. Abra a URL no navegador (o `user_code` já vem preenchido)
3. Faça login com a conta Vercel (GitHub, GitLab, Bitbucket ou email)
4. Aprove o acesso para o dispositivo
5. Volte ao terminal — a sessão é detectada automaticamente

Confirmação após login:

```powershell
vercel whoami
# Saída: nome-de-usuario
#        Active team: nome-do-time
```

#### Integração com Claude Code (MCP)

Durante o primeiro `vercel whoami` ou `vercel login`, a CLI pergunta:

```
? Working with Vercel is easier with the Vercel Plugin for Claude Code.
  Would you like to install it? (Y/n)
```

Responder `yes` instala o **Vercel Plugin for Claude Code** (MCP server). Isso permite que o Claude consulte seus projetos, deployments e logs diretamente na conversa, sem precisar copiar e colar saídas do terminal.

Após instalar, é necessário **reiniciar o Claude Code** para que as novas ferramentas `mcp__vercel__*` fiquem disponíveis na sessão.

Esse passo é opcional — a CLI funciona normalmente sem o plugin. Só vale instalar se você usa o Claude Code e quer delegar consultas à conta Vercel para o assistente.

---

## 3. Exemplos Simples e Práticos

### Exemplo 1 — Deploy automático via Git

Conecte seu repositório GitHub à Vercel pelo painel em [vercel.com/new](https://vercel.com/new).

Após conectado, cada push no branch `main` aciona deploy automático em produção. Cada push em outros branches gera um preview com URL única.

```bash
git add .
git commit -m "feat: nova feature"
git push origin main
# → Deploy automático em produção iniciado
```

---

### Exemplo 2 — Variáveis de ambiente

**No arquivo `.env.local` (desenvolvimento local):**
```env
DATABASE_URL=postgresql://localhost:5432/mydb
NEXT_PUBLIC_API_URL=http://localhost:3000/api
```

> `NEXT_PUBLIC_` expõe a variável para o client-side (bundle do browser).  
> Sem esse prefixo, a variável fica disponível apenas no servidor.

**No painel da Vercel:**  
`Project > Settings > Environment Variables`

Configure por ambiente:
- **Production** → branch `main`
- **Preview** → demais branches
- **Development** → uso com `vercel env pull`

**Puxar variáveis de ambiente da Vercel para local:**
```bash
vercel env pull .env.local
```

**Uso no código:**
```ts
// Somente server-side
const dbUrl = process.env.DATABASE_URL;

// Client-side e server-side
const apiUrl = process.env.NEXT_PUBLIC_API_URL;
```

---

### Exemplo 3 — ISR (Incremental Static Regeneration) com Next.js

ISR permite regenerar páginas estáticas em background sem rebuild completo — recurso que a Vercel suporta nativamente.

```tsx
// app/products/[id]/page.tsx

interface Props {
  params: { id: string };
}

// Revalida a página a cada 60 segundos
export const revalidate = 60;

async function getProduct(id: string) {
  const res = await fetch(`https://api.exemplo.com/products/${id}`, {
    next: { revalidate: 60 }, // Cache granular por fetch
  });
  return res.json();
}

export default async function ProductPage({ params }: Props) {
  const product = await getProduct(params.id);

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
    </div>
  );
}
```

---

### Exemplo 4 — Edge Function com middleware

Edge Functions rodam no middleware do Next.js, antes do request chegar ao servidor. Ideal para autenticação, redirecionamentos e A/B testing.

```ts
// middleware.ts (raiz do projeto)
import { NextRequest, NextResponse } from 'next/server';

export function middleware(request: NextRequest) {
  const token = request.cookies.get('auth-token');

  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*'],
};
```

Este middleware roda **na Edge Network da Vercel**, com latência mínima e sem cold start perceptível.

---

### Exemplo 5 — `vercel.json` com configurações de roteamento e headers

```json
{
  "version": 2,
  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [
        { "key": "Cache-Control", "value": "no-store" },
        { "key": "X-Content-Type-Options", "value": "nosniff" }
      ]
    }
  ],
  "redirects": [
    {
      "source": "/blog",
      "destination": "/artigos",
      "permanent": true
    }
  ],
  "rewrites": [
    {
      "source": "/proxy/:path*",
      "destination": "https://api-externa.com/:path*"
    }
  ]
}
```

---

## 4. Boas Práticas

### Organização de variáveis de ambiente

- **Nunca** commite arquivos `.env` com secrets no repositório.
- Use `.env.local` para desenvolvimento local (já ignorado pelo `.gitignore` padrão do Next.js).
- Separe claramente variáveis por ambiente no painel da Vercel.
- Prefixe com `NEXT_PUBLIC_` apenas o que realmente precisa ir ao client-side.

```bash
# .gitignore — garanta que esses arquivos estejam listados
.env
.env.local
.env.*.local
```

---

### Cache e revalidação explícitos

Evite deixar o comportamento de cache implícito. Seja explícito nas chamadas `fetch`:

```ts
// Sem cache (sempre fresco)
fetch(url, { cache: 'no-store' });

// Cache permanente (build time)
fetch(url, { cache: 'force-cache' });

// Revalidação periódica
fetch(url, { next: { revalidate: 3600 } }); // 1 hora

// Revalidação por tag (on-demand)
fetch(url, { next: { tags: ['products'] } });
```

---

### Separar lógica de Edge e Node.js

Edge Functions têm runtime limitado (sem acesso a `fs`, sem drivers de banco nativos, sem pacotes Node.js puros). Separe claramente:

```ts
// ✅ Adequado para Edge (middleware.ts)
import { NextResponse } from 'next/server';

// ✅ Adequado para Serverless (app/api/data/route.ts)
import { PrismaClient } from '@prisma/client';
```

---

### Preview Deployments como ambiente de QA

Use preview deployments como etapa obrigatória no fluxo de PR:

1. Abra um PR → Vercel gera URL de preview automaticamente
2. Compartilhe a URL com stakeholders para validação
3. Aprovação → merge → deploy em produção

Configure comentários automáticos de preview no GitHub pela integração nativa da Vercel.

---

### Monitoramento com Vercel Analytics

Ative **Vercel Analytics** e **Speed Insights** para monitorar Core Web Vitals em produção:

```tsx
// app/layout.tsx
import { Analytics } from '@vercel/analytics/react';
import { SpeedInsights } from '@vercel/speed-insights/next';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
        <SpeedInsights />
      </body>
    </html>
  );
}
```

```bash
npm install @vercel/analytics @vercel/speed-insights
```

---

## 5. Limitações e Armadilhas

### Cold Start em Serverless Functions

**Problema:** A primeira requisição após um período de inatividade pode ser lenta (100ms a 1s+) porque a função precisa ser "acordada".

**Como mitigar:**
- Use **Edge Functions** para endpoints críticos de latência (sem cold start perceptível).
- Evite importar pacotes pesados desnecessariamente nas funções.
- Considere o plano Pro da Vercel para menor frequência de cold starts.
- Prefira lógica leve nas API Routes; delegue processamento pesado para serviços externos (filas, workers).

```ts
// ❌ Evitar — importação pesada aumenta cold start
import * as _ from 'lodash';

// ✅ Prefira imports específicos
import { debounce } from 'lodash';
```

---

### Limites do plano Hobby (gratuito)

| Recurso | Limite Hobby |
|---|---|
| Serverless Function duration | 10 segundos |
| Edge Function duration | 30 segundos (CPU time) |
| Serverless Function size | 50 MB |
| Bandwidth | 100 GB/mês |
| Builds por dia | 100 |
| Projetos | Ilimitados |
| Team members | Apenas 1 (uso pessoal) |
| Analytics | Limitado |
| **Uso comercial** | **❌ Não permitido** |

> ⚠️ O plano Hobby **não permite uso comercial**. Para projetos de clientes ou empresas, use o plano Pro (a partir de $20/mês por membro).

---

### Armadilha: Cache agressivo sem intenção

A Vercel aplica cache automaticamente em rotas SSG e ISR. Se você não configurar `cache: 'no-store'` em API Routes dinâmicas, pode receber dados antigos inesperadamente.

```ts
// ❌ Pode ser cacheado sem intenção no Next.js 14+
export async function GET() {
  const data = await fetchDynamicData();
  return NextResponse.json(data);
}

// ✅ Explicitamente sem cache
export const dynamic = 'force-dynamic';

export async function GET() {
  const data = await fetchDynamicData();
  return NextResponse.json(data);
}
```

---

### Armadilha: Variáveis de ambiente no client-side

```ts
// ❌ Isso retorna undefined no browser
const secret = process.env.SECRET_KEY;

// ✅ Apenas variáveis NEXT_PUBLIC_ chegam ao client
const publicUrl = process.env.NEXT_PUBLIC_API_URL;
```

---

### Armadilha: Tamanho do bundle de Serverless Functions

Cada API Route é empacotada individualmente. Imports pesados (ex: SDKs completos da AWS) podem ultrapassar o limite de 50MB.

**Solução:** Use imports dinâmicos ou SDKs modulares:

```ts
// ✅ Import dinâmico — só carrega quando necessário
const { S3Client } = await import('@aws-sdk/client-s3');
```

---

### Armadilha: `fs` e acesso ao sistema de arquivos

O filesystem em Serverless Functions é **somente leitura** na Vercel (exceto `/tmp`, com limite de 512MB).

```ts
// ❌ Falha em produção
import fs from 'fs';
fs.writeFileSync('/data/output.json', JSON.stringify(data));

// ✅ Apenas /tmp é gravável
fs.writeFileSync('/tmp/output.json', JSON.stringify(data));
```

---

## 6. Quando Usar e Quando Evitar

### Quando usar ✅

**1. Projetos Next.js de qualquer porte**  
A Vercel foi criada pelos autores do Next.js. SSR, SSG, ISR, Server Components, Server Actions e API Routes funcionam sem configuração adicional.

**2. Times que precisam de agilidade**  
Preview deployments por PR eliminam a necessidade de ambientes de staging manuais. QA e stakeholders validam features diretamente pela URL gerada.

**3. Projetos com tráfego variável ou imprevisível**  
O modelo serverless escala automaticamente de zero a picos de tráfego sem intervenção manual.

**4. Projetos com foco em performance**  
CDN global, Edge Functions e Speed Insights nativas facilitam atingir boas pontuações em Core Web Vitals.

**5. Projetos open source e pessoais**  
O plano Hobby é generoso para experimentação, portfólios e projetos não comerciais.

---

### Quando evitar ❌

**1. Aplicações com processos de longa duração**  
Websockets persistentes, jobs de background contínuos, processamento de vídeo/áudio pesado — o modelo serverless não é adequado. Use serviços como Railway, Render, ou VMs dedicadas.

**2. Uso comercial sem orçamento para plano Pro**  
O plano Hobby proíbe uso comercial explicitamente nos termos de serviço.

**3. Aplicações com banco de dados self-hosted**  
A Vercel não hospeda bancos de dados. Se você precisa de controle total sobre o banco (on-premise, VPC privada), a integração com a Vercel pode ser complexa ou inviável.

**4. Backend-heavy com muita lógica de servidor**  
APIs com alta complexidade, autenticação customizada avançada, ou microsserviços podem ficar caros e difíceis de depurar no modelo serverless.

**5. Requisitos regulatórios de localização de dados**  
Se sua aplicação precisa garantir que dados nunca saiam de uma região geográfica específica (LGPD, GDPR com restrição de transferência), valide cuidadosamente as opções de região da Vercel.

---

## 7. Alternativas Modernas

### Comparativo rápido

| Plataforma | Melhor para | Next.js | Deploy | Preço inicial |
|---|---|---|---|---|
| **Vercel** | Next.js, front-end moderno | ✅ Nativo | Git / CLI | Gratuito (Hobby) |
| **Netlify** | Jamstack, sites estáticos | ✅ Bom | Git / CLI | Gratuito |
| **Cloudflare Pages** | Edge-first, Workers | ⚠️ Parcial | Git / CLI | Gratuito generoso |
| **Railway** | Full-stack, containers | ✅ Bom | Git / Docker | $5/mês |
| **Render** | Web services, workers | ✅ Bom | Git / Docker | Gratuito (limitado) |
| **AWS Amplify** | Ecossistema AWS | ✅ Bom | Git / CLI | Pay-as-you-go |
| **Fly.io** | Containers, latência baixa | ✅ Via Docker | CLI / Docker | Gratuito (limitado) |

---

### Netlify

**Quando preferir:** Projetos Jamstack com formulários, funções simples e fluxos de CMS headless. A Netlify tem um ecossistema de plugins rico e suporte a frameworks além do Next.js.

**Limitação vs. Vercel:** Suporte a Server Components e ISR do Next.js é menos otimizado.

---

### Cloudflare Pages + Workers

**Quando preferir:** Projetos edge-first onde performance máxima e custo baixo são prioritários. Workers têm praticamente zero cold start e o plano gratuito é muito generoso.

**Limitação vs. Vercel:** Next.js tem suporte parcial via adaptador `@cloudflare/next-on-pages`. Nem todos os recursos do Next.js App Router são suportados.

```bash
# Deploy Next.js para Cloudflare Pages
npm install -D @cloudflare/next-on-pages
npx @cloudflare/next-on-pages
```

---

### Railway

**Quando preferir:** Quando você precisa de um servidor Node.js persistente, banco de dados integrado (PostgreSQL, Redis, MySQL), ou processos de longa duração.

**Limitação vs. Vercel:** Não tem CDN global nativa nem preview deployments automáticos por PR.

---

### AWS Amplify

**Quando preferir:** Projetos já dentro do ecossistema AWS que precisam de integração com Cognito, DynamoDB, AppSync, etc.

**Limitação vs. Vercel:** Curva de configuração mais alta, experiência de DX menos polida.

---

## 8. Resumo Final

### Principais aprendizados

1. **Vercel é a plataforma de referência para Next.js** — zero configuração, suporte nativo a todos os modos de renderização (SSR, SSG, ISR, Server Components).

2. **Preview Deployments são um diferencial real** — cada PR gera uma URL de preview automaticamente, acelerando ciclos de revisão e QA.

3. **Serverless Functions têm limitações importantes** — cold start, timeout de 10s no plano Hobby, filesystem somente leitura (exceto `/tmp`), e limite de 50MB por função.

4. **Edge Functions são ideais para lógica de baixa latência** — middleware de autenticação, redirecionamentos, A/B testing. Sem cold start perceptível, mas sem Node.js completo.

5. **Variáveis de ambiente têm escopo definido** — `NEXT_PUBLIC_` vai para o browser; sem o prefixo, fica no servidor. Gerencie por ambiente (production/preview/development) no painel.

6. **Cache precisa ser explícito** — o Next.js 14+ tem comportamentos de cache agressivos. Seja explícito com `force-dynamic`, `no-store`, `revalidate` ou tags.

7. **Plano Hobby não permite uso comercial** — para projetos de clientes, use o plano Pro.

---

### Guia rápido de consulta

```bash
# === INSTALAÇÃO E SETUP ===
node --version            # Pré-requisito: Node.js LTS instalado
npm i -g vercel           # Instalar CLI (admin se der EACCES/EPERM)
vercel --version          # Confirmar instalação
vercel login              # Autenticar (fluxo device code via browser)
vercel whoami             # Conferir usuário e time ativo
vercel link               # Vincular a pasta atual a um projeto Vercel
vercel env pull .env.local # Puxar variáveis de ambiente para local
vercel                    # Deploy de preview
vercel --prod             # Deploy em produção
vercel ls                 # Listar deployments do projeto atual
vercel logs <url>         # Logs de um deployment específico
vercel logout             # Encerrar sessão

# === NEXT.JS — CONTROLE DE CACHE ===
export const dynamic = 'force-dynamic';     # Sem cache (SSR puro)
export const revalidate = 60;               # ISR a cada 60s
fetch(url, { cache: 'no-store' })           # Sem cache por fetch
fetch(url, { next: { revalidate: 3600 } }) # Cache com TTL

# === VARIÁVEIS DE AMBIENTE ===
# Server-side only:
process.env.SECRET_KEY

# Client + Server:
process.env.NEXT_PUBLIC_API_URL

# === EDGE vs SERVERLESS ===
# Edge (middleware.ts) → sem Node.js completo, latência ultra-baixa
# Serverless (app/api/**/route.ts) → Node.js completo, até 10s (Hobby)

# === FILESYSTEM ===
# Somente /tmp é gravável em produção
fs.writeFileSync('/tmp/arquivo.json', dados);
```

**Checklist de deploy para produção:**
- [ ] Variáveis de ambiente configuradas no painel da Vercel (não no `.env` commitado)
- [ ] `NEXT_PUBLIC_` apenas nas variáveis que devem ir ao browser
- [ ] Cache configurado explicitamente nas rotas dinâmicas
- [ ] Middleware de autenticação em Edge Function
- [ ] Analytics e Speed Insights ativados
- [ ] Domínio customizado configurado (SSL automático pela Vercel)
- [ ] Limites de runtime validados para cada Serverless Function
- [ ] Plano adequado ao uso (Hobby apenas para projetos pessoais não comerciais)
