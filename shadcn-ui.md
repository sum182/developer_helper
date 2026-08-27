# shadcn/ui — Guia de Uso (frontend)

## 1. Introdução
- **Contexto do tema:** `shadcn/ui` é uma coleção de componentes reutilizáveis para React (com forte adoção em Next.js), baseada em Radix UI + Tailwind CSS. Em vez de instalar um pacote “fechado” de componentes, você copia os componentes para dentro do seu projeto e controla o código.
- **Problema que resolve:** acelera a construção de interfaces consistentes (forms, tabelas, modais, toasts, menus, etc.) sem perder flexibilidade de customização, algo crítico em ERPs, CRMs e sistemas com muitos CRUDs.
- **Quando faz sentido aplicar:** quando você quer velocidade de entrega + padronização visual + autonomia para ajustar UI conforme regras de negócio e integração com fluxos de IA no frontend.

## 2. Sintaxe Básica
- **Conceitos essenciais:**
  - Componentes ficam no seu código (`components/ui/*`).
  - Estilo orientado a utility classes (Tailwind).
  - Acessibilidade e comportamento baseados em Radix UI.
  - Variações visuais com `class-variance-authority (cva)`.
- **Exemplo mínimo funcional:** botão com variante.

```tsx
import { Button } from "@/components/ui/button"

export function SaveButton() {
  return <Button variant="default">Salvar</Button>
}
```

Exemplo de uso com feedback de estado:

```tsx
<Button disabled={isSaving}>
  {isSaving ? "Salvando..." : "Salvar alterações"}
</Button>
```

## 3. Exemplos Simples e Práticos

### Exemplo 1 — Formulário de CRUD com validação (cliente)
```tsx
import { useForm } from "react-hook-form"
import { z } from "zod"
import { zodResolver } from "@hookform/resolvers/zod"
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"

const schema = z.object({
  nome: z.string().min(3, "Nome muito curto"),
  email: z.string().email("E-mail inválido"),
})

type FormData = z.infer<typeof schema>

export function ClienteForm() {
  const form = useForm<FormData>({ resolver: zodResolver(schema) })

  const onSubmit = (data: FormData) => {
    // enviar para API
    console.log(data)
  }

  return (
    <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
      <Input placeholder="Nome" {...form.register("nome")} />
      <Input placeholder="E-mail" {...form.register("email")} />
      <Button type="submit">Salvar cliente</Button>
    </form>
  )
}
```

### Exemplo 2 — Tabela de listagem com ações
```tsx
import { Button } from "@/components/ui/button"

function AcoesLinha({ onEditar, onExcluir }: { onEditar: () => void; onExcluir: () => void }) {
  return (
    <div className="flex gap-2">
      <Button variant="outline" size="sm" onClick={onEditar}>Editar</Button>
      <Button variant="destructive" size="sm" onClick={onExcluir}>Excluir</Button>
    </div>
  )
}
```

### Exemplo 3 — Modal de confirmação para exclusão
```tsx
import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
  AlertDialogTrigger,
} from "@/components/ui/alert-dialog"
import { Button } from "@/components/ui/button"

export function ConfirmDelete({ onConfirm }: { onConfirm: () => void }) {
  return (
    <AlertDialog>
      <AlertDialogTrigger asChild>
        <Button variant="destructive">Excluir</Button>
      </AlertDialogTrigger>
      <AlertDialogContent>
        <AlertDialogHeader>
          <AlertDialogTitle>Tem certeza?</AlertDialogTitle>
          <AlertDialogDescription>
            Esta ação não poderá ser desfeita.
          </AlertDialogDescription>
        </AlertDialogHeader>
        <AlertDialogFooter>
          <AlertDialogCancel>Cancelar</AlertDialogCancel>
          <AlertDialogAction onClick={onConfirm}>Confirmar</AlertDialogAction>
        </AlertDialogFooter>
      </AlertDialogContent>
    </AlertDialog>
  )
}
```

### Exemplo 4 — Bloco para resposta de IA no frontend
```tsx
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"
import { ScrollArea } from "@/components/ui/scroll-area"

export function AgentResponse({ text }: { text: string }) {
  return (
    <Card>
      <CardHeader>
        <CardTitle>Sugestão da IA</CardTitle>
      </CardHeader>
      <CardContent>
        <ScrollArea className="h-64 rounded-md border p-3 text-sm leading-6">
          {text}
        </ScrollArea>
      </CardContent>
    </Card>
  )
}
```

### Exemplo 5 — Filtros de lista (ERP) com estado visual claro
```tsx
import { Badge } from "@/components/ui/badge"
import { Button } from "@/components/ui/button"

export function StatusFilter({ active, onChange }: { active: string; onChange: (v: string) => void }) {
  const options = ["todos", "ativos", "inativos", "pendentes"]
  return (
    <div className="flex items-center gap-2">
      {options.map((opt) => (
        <Button
          key={opt}
          variant={active === opt ? "default" : "outline"}
          size="sm"
          onClick={() => onChange(opt)}
        >
          {opt}
        </Button>
      ))}
      <Badge variant="secondary">Filtro atual: {active}</Badge>
    </div>
  )
}
```

## 4. Boas Práticas
- Defina um **design system interno** (tokens de cor, espaçamento, tipografia, radius) antes de escalar CRUDs.
- Crie wrappers de domínio: `CustomerTable`, `OrderStatusBadge`, `ConfirmDeleteDialog` em vez de repetir UI genérica.
- Use `react-hook-form + zod` para consistência de validação em todos os formulários.
- Padronize feedbacks: loading, sucesso, erro, vazio, sem permissão.
- Para IA no frontend, mantenha componentes separados para:
  - entrada de prompt,
  - visualização de resposta,
  - estado de execução (streaming/loading/erro),
  - ações pós-resposta (copiar, aplicar, revisar).

## 5. Limitações e Armadilhas
- **Não é um pacote pronto total:** você mantém o código dos componentes; isso dá poder, mas exige disciplina de manutenção.
- **Risco de divergência visual:** sem governança, cada tela pode customizar demais e perder consistência.
- **Acoplamento com Tailwind:** quem não domina utilitários pode gerar classes repetidas e difíceis de manter.
- **Upgrade manual:** mudanças da comunidade nem sempre entram automaticamente no seu código local.
- **Acessibilidade pode regredir:** ao alterar componentes sem cuidado, você pode quebrar foco, teclado e ARIA.

Como evitar:
- Definir guidelines de UI no projeto.
- Revisar componentes base periodicamente.
- Ter testes de fluxo (especialmente modais e formulários).

## 6. Quando Usar e Quando Evitar

### Quando usar
- Sistemas com muitos formulários, listas, tabelas e modais (ERP/CRM/backoffice).
- Times que precisam de velocidade sem abrir mão de customização.
- Projetos com roadmap longo, onde possuir o código dos componentes é vantagem.
- Interfaces com integração de IA, exigindo estados específicos (streaming, versões de resposta, confirmação humana).

### Quando evitar
- Projetos ultra simples e descartáveis.
- Times sem disponibilidade para manter padrão de design.
- Cenários em que um kit totalmente fechado e “plug-and-play” é prioridade absoluta.

## 7. Alternativas Modernas
- **MUI (Material UI):** muito completo e maduro, ótimo ecossistema, porém com identidade visual mais opinionada.
- **Chakra UI:** DX boa e acessível, customização simples, mas abordagem visual diferente de Tailwind-first.
- **Mantine:** kit amplo com muitos componentes prontos e utilitários.
- **Ant Design:** forte para dashboards/ERPs corporativos, com visual mais tradicional.
- **Headless UI + Tailwind:** alternativa mais “baixo nível”, com grande liberdade e mais trabalho de composição.

Comparativo rápido:
- **shadcn/ui:** melhor equilíbrio entre controle de código + velocidade + estética moderna para frontend agentico.
- **MUI/Antd:** maior prontidão imediata para telas corporativas complexas.
- **Headless UI:** mais liberdade, porém exige mais implementação manual.

## 8. Resumo Final
- `shadcn/ui` é uma escolha forte para padronizar e acelerar UI de ERPs, CRUDs e fluxos com IA no frontend.
- O principal diferencial é **possuir o código dos componentes**, permitindo adaptar rapidamente ao negócio.
- O sucesso depende de governança: tokens, padrões de composição, validação e acessibilidade.
- Para programação agentica, combine `shadcn/ui` com uma arquitetura de componentes orientada a estados de IA (input, execução, resposta, ação humana).

### Guia rápido de consulta
- Quer velocidade com personalização real? → `shadcn/ui`
- Quer tudo pronto e opinionado? → MUI/Antd
- Quer máximo controle do zero? → Headless UI + Tailwind

## 9. Repositórios, Templates e Estratégia de Escala

Com base no estudo adicional, o ponto mais relevante para o seu cenário (múltiplos ERPs) é este:

- `shadcn/ui` sozinho resolve a base visual, mas **escala de verdade** vem com:
  - **monorepo**,
  - **pacotes internos versionados**,
  - **composição (não herança)**,
  - **CRUD engine configurável por schema**.

### Arquitetura recomendada para vários sistemas

```txt
repo/
  apps/
    erp-financeiro/
    erp-comercial/
    erp-operacional/
  packages/
    ui/       -> componentes base (button, dialog, table, input)
    crud/     -> engine de CRUD (lista, filtro, paginação, modal/form)
    layout/   -> sidebar, header, breadcrumbs
    auth/     -> login, permissões, sessão
    ai/       -> copilot, sugestão, auto-preenchimento, histórico
    theme/    -> tokens, preset, tipografia, densidade
```

### Regra de ouro: composição > herança

Evite:

```ts
class ClienteModal extends BaseModal {}
```

Prefira:

```tsx
<BaseModal title="Cliente" footer={<Acoes />}>
  <ClienteForm />
</BaseModal>
```

### Pacotes privados e propagação de mudanças

- É viável usar repositório privado e pacotes internos (`@empresa/ui`, `@empresa/crud`, etc.).
- Em produção, prefira propagação via **versionamento controlado**:
  - publica nova versão,
  - cada sistema atualiza quando estiver pronto,
  - reduz risco de quebra global.
- Em monorepo único, a mudança é imediata para todos os apps do workspace.

### Preset ERP recomendado (ponto de partida)

```json
{
  "style": "default",
  "baseColor": "slate",
  "primaryColor": "blue",
  "radius": "0.4rem"
}
```

Densidade sugerida para ERP (mais dados por tela):

```css
:root { --spacing: 0.5rem; }
.table td, .table th { padding: 6px 8px; }
.input { height: 32px; }
body { font-family: Inter, system-ui; font-size: 13px; line-height: 1.4; }
```

## 10. Repositórios e Templates Relacionados

### Base oficial
- shadcn/ui (docs): https://ui.shadcn.com/
- shadcn/ui create/preset: https://ui.shadcn.com/create
- Componentes oficiais: https://ui.shadcn.com/docs/components
- Data Table (shadcn): https://ui.shadcn.com/docs/components/data-table
- Dialog: https://ui.shadcn.com/docs/components/dialog
- Alert Dialog: https://ui.shadcn.com/docs/components/alert-dialog
- Sheet (drawer): https://ui.shadcn.com/docs/components/sheet

### Stack complementar para ERP
- TanStack Table: https://tanstack.com/table/latest
- React Hook Form: https://react-hook-form.com/
- Zod: https://zod.dev/
- Refine (framework CRUD/headless): https://refine.dev/

### Ecossistema e exemplos
- Awesome Shadcn UI (coleções da comunidade): https://github.com/topics/shadcn-ui
- Templates admin com shadcn (busca GitHub): https://github.com/search?q=shadcn+admin+template&type=repositories
- CRUD com shadcn + Refine (busca): https://github.com/search?q=refine+shadcn&type=repositories

### Estratégia prática para seu contexto
1. Definir `@empresa/theme` (tokens + densidade + preset).
2. Consolidar `@empresa/ui` (componentes base).
3. Criar `@empresa/crud` (lista + form + modal + paginação + filtros).
4. Adicionar `@empresa/ai` (sugestões, auto-preenchimento, ações assistidas).
5. Replicar para ERPs por configuração (schema-driven), não por cópia de tela.

### Nota sobre mobile
- `shadcn/ui` é web (React + Tailwind + Radix), não nativo mobile.
- Para reaproveitamento:
  - Web + Capacitor (rápido, 1 base)
  - ou Web + React Native (melhor UX/performance no longo prazo).

## 11. Templates (Composites / Blocks)

> Esta seção é o coração do reuso no shadcn em projetos médios/grandes. Conceito derivado de análise de 3 projetos reais (1 prompt-manager, 2 ERPs) e da documentação oficial / Vercel Academy / artigos de boas práticas 2026.

### 11.1 O problema

Os primitivos do shadcn (`Button`, `Input`, `Dialog`, etc.) são **átomos visuais genéricos**. Em um app real, você nunca usa só `<Button>`: usa "botão de confirmar exclusão", "botão de copiar com tooltip e feedback", "header de página com título + descrição + ações". Se cada tela compõe esses padrões do zero, você acaba com:

- Duplicação visual (cada dev faz um header diferente).
- Inconsistência de UX (uns deletes com `confirm()` nativo, outros com `AlertDialog`).
- Mudança visual cara (alterar a tipografia do header obriga abrir 12 arquivos).
- Sem governança, o "design system" vira sugestão.

### 11.2 A solução: camada de templates

Convenção amplamente adotada na comunidade shadcn (2026), com 3 camadas:

```
src/components/
├── ui/          # primitivos crus do shadcn — gerados pelo CLI, kebab-case
│                # (button.tsx, dialog.tsx, alert-dialog.tsx, ...)
├── primitives/  # primitivos levemente modificados (opcional)
│                # ex.: button com loading state padrão da casa
└── templates/   # composições de aplicação — opinativas, PascalCase
                 # (PageHeader.tsx, EmptyState.tsx, ConfirmDialog.tsx, ...)
```

Aliases comuns para a pasta `templates/`: `blocks/`, `composites/`, `patterns/`. Escolha um e cravar a convenção no AGENTS.md. Em projetos onde "template" se confunde com domínio (ex.: um app de prompts/email templates), prefira `composites/`. Caso contrário, `templates/` é o nome mais reconhecido.

### 11.3 Critérios para criar um template

Um componente só vira template quando atende a **pelo menos um** destes:

1. **Reuso real ≥ 3 vezes** — não crie por antecipação. Comece inline; ao terceiro uso quase idêntico, extraia.
2. **Encapsula regra de UX da casa** — ex.: "toda confirmação de exclusão usa AlertDialog com botão vermelho à direita e Cancel à esquerda".
3. **Esconde complexidade orquestral** — junta 3-5 primitivos em uma API simples (`<PageHeader title="..." actions={...} />` vs 12 linhas de markup).
4. **Padroniza estado de UI** — empty / loading / error / success.

Anti-padrões (sinais de over-engineering):

- Template usado em 1 lugar só.
- Template que só adiciona uma `className` ao primitivo.
- Template "genérico para qualquer entidade" (volta a virar o ERP CRUD problem).
- Template com mais de 6-8 props — quebra em dois.

### 11.4 Os 5 templates mais reaproveitáveis

#### a) `PageHeader` — header padronizado de página

```tsx
import { ReactNode } from "react"

interface PageHeaderProps {
  title: string
  description?: string
  actions?: ReactNode
}

export function PageHeader({ title, description, actions }: PageHeaderProps) {
  return (
    <div className="flex items-start justify-between gap-4 pb-6 border-b">
      <div className="space-y-1">
        <h1 className="text-2xl font-semibold tracking-tight">{title}</h1>
        {description && (
          <p className="text-sm text-muted-foreground">{description}</p>
        )}
      </div>
      {actions && <div className="flex items-center gap-2">{actions}</div>}
    </div>
  )
}
```

Uso:
```tsx
<PageHeader
  title="Prompts"
  description="Gerencie seus templates de IA"
  actions={<Button onClick={onNew}>Novo prompt</Button>}
/>
```

#### b) `EmptyState` — estado vazio padronizado

> **Nota**: o shadcn lançou em 2025 um componente oficial `Empty` (com sub-componentes `EmptyHeader`, `EmptyMedia`, `EmptyTitle`, `EmptyDescription`, `EmptyContent`). Avalie usar o oficial via `shadcn add empty` antes de criar o seu. O template abaixo é uma versão enxuta caso queira API mais simples.

```tsx
import { ReactNode } from "react"
import { LucideIcon } from "lucide-react"

interface EmptyStateProps {
  icon?: LucideIcon
  title: string
  description?: string
  action?: ReactNode
}

export function EmptyState({ icon: Icon, title, description, action }: EmptyStateProps) {
  return (
    <div className="flex flex-col items-center justify-center py-12 text-center">
      {Icon && <Icon className="h-10 w-10 text-muted-foreground mb-4" />}
      <h3 className="text-lg font-semibold">{title}</h3>
      {description && (
        <p className="text-sm text-muted-foreground mt-1 max-w-sm">{description}</p>
      )}
      {action && <div className="mt-4">{action}</div>}
    </div>
  )
}
```

#### c) `ConfirmDialog` — substituto do `confirm()` nativo

```tsx
import {
  AlertDialog, AlertDialogAction, AlertDialogCancel,
  AlertDialogContent, AlertDialogDescription, AlertDialogFooter,
  AlertDialogHeader, AlertDialogTitle,
} from "@/components/ui/alert-dialog"

interface ConfirmDialogProps {
  open: boolean
  onOpenChange: (open: boolean) => void
  title: string
  description?: string
  confirmLabel?: string
  cancelLabel?: string
  destructive?: boolean
  onConfirm: () => void | Promise<void>
}

export function ConfirmDialog({
  open, onOpenChange, title, description,
  confirmLabel = "Confirmar", cancelLabel = "Cancelar",
  destructive = false, onConfirm,
}: ConfirmDialogProps) {
  return (
    <AlertDialog open={open} onOpenChange={onOpenChange}>
      <AlertDialogContent>
        <AlertDialogHeader>
          <AlertDialogTitle>{title}</AlertDialogTitle>
          {description && <AlertDialogDescription>{description}</AlertDialogDescription>}
        </AlertDialogHeader>
        <AlertDialogFooter>
          <AlertDialogCancel>{cancelLabel}</AlertDialogCancel>
          <AlertDialogAction
            onClick={onConfirm}
            className={destructive ? "bg-destructive text-destructive-foreground hover:bg-destructive/90" : ""}
          >
            {confirmLabel}
          </AlertDialogAction>
        </AlertDialogFooter>
      </AlertDialogContent>
    </AlertDialog>
  )
}
```

#### d) `FormSection` — agrupador de campo + label + helper + erro

```tsx
import { ReactNode } from "react"
import { Label } from "@/components/ui/label"

interface FormSectionProps {
  label: string
  htmlFor?: string
  helperText?: string
  error?: string
  required?: boolean
  children: ReactNode
}

export function FormSection({ label, htmlFor, helperText, error, required, children }: FormSectionProps) {
  return (
    <div className="space-y-2">
      <Label htmlFor={htmlFor}>
        {label}
        {required && <span className="text-destructive ml-1">*</span>}
      </Label>
      {children}
      {error ? (
        <p className="text-xs text-destructive">{error}</p>
      ) : helperText ? (
        <p className="text-xs text-muted-foreground">{helperText}</p>
      ) : null}
    </div>
  )
}
```

#### e) `StatusBadge` — badge com intenção semântica

```tsx
import { Badge } from "@/components/ui/badge"
import { cn } from "@/lib/utils"

type Status = "success" | "warning" | "error" | "info" | "neutral"

const styles: Record<Status, string> = {
  success: "bg-green-500/15 text-green-600 border-green-500/30",
  warning: "bg-yellow-500/15 text-yellow-600 border-yellow-500/30",
  error: "bg-red-500/15 text-red-600 border-red-500/30",
  info: "bg-blue-500/15 text-blue-600 border-blue-500/30",
  neutral: "bg-muted text-muted-foreground border-border",
}

export function StatusBadge({ status, label }: { status: Status; label: string }) {
  return <Badge variant="outline" className={cn(styles[status])}>{label}</Badge>
}
```

### 11.5 Templates avançados (apenas para apps grandes)

Para ERPs/backoffices com muitas listagens tabulares, vale construir:

- **`DataTable<T>`** — wrapper sobre `Table` + TanStack Table com paginação, ordenação, seleção de linhas, toolbar de filtros, density toggle.
- **`CrudDialog<T>` / `CrudSheet<T>`** — modal/drawer genérico de criar/editar com schema-driven form.
- **`AppSidebar`** — sidebar institucional com nav sections, user menu, role filtering (use o componente oficial `sidebar` do shadcn como base).
- **`PageShell`** — wrapper completo: breadcrumbs + PageHeader + tabs + content slot.

⚠️ **Cuidado**: esses são templates de **alto custo cognitivo**. Só compensam quando você tem ≥5 telas com a mesma estrutura. Em apps menores, eles viram complexidade desnecessária e dificultam onboarding de novos devs.

### 11.6 Templates × Blocks oficiais do shadcn

A comunidade shadcn diferencia:

| Conceito | Origem | Exemplo | Quando usar |
|---|---|---|---|
| **Primitivo** (`ui/`) | shadcn CLI | `Button`, `Dialog` | Sempre — base de tudo |
| **Block** (oficial) | shadcn.com/blocks | Hero, login form, sidebar-07 | Copiar e adaptar para seções de página inteiras |
| **Template/Composite** (seu) | seu projeto | `PageHeader`, `ConfirmDialog` | Padrões internos da aplicação |
| **Template/Boilerplate** (start kit) | shadcn-ui-kit, etc. | Admin dashboard pronto | Bootstrap rápido de novo projeto |

### 11.7 Estratégia para múltiplos projetos (monorepo / pacote interno)

Quando você tem 3+ aplicações compartilhando design system:

```
@empresa/ui          → primitivos shadcn customizados
@empresa/templates   → PageHeader, ConfirmDialog, EmptyState, FormSection...
@empresa/blocks      → AppSidebar, CrudDialog, DataTable (heavy composites)
@empresa/theme       → tokens, CSS variables, presets
```

Distribuição:
- **Monorepo (Turborepo, Nx, pnpm workspaces)** → mudança propaga imediato a todos os apps.
- **Pacote npm privado versionado** → cada app atualiza quando quiser (mais seguro em prod).
- **shadcn Registry privada** (ver seção 12) → híbrido: distribuição "shadcn-style" (cópia do código) com governança central.

### 11.8 Resumo da prática

- Templates são onde o seu **design system vira código aplicado**.
- A regra é "extrair no terceiro uso", não antecipar.
- Pasta `templates/` (ou `composites/`) é a fronteira entre primitivo genérico e UI opinada.
- Em apps pequenos, 3-5 templates resolvem 80% do problema. Em ERPs, vale a camada `blocks/` adicional.
- Para reuso entre projetos, migre para registry privada ou pacote versionado.

---

## 12. Ferramentas do Ecossistema shadcn

Visão consolidada das ferramentas oficiais e da comunidade que aceleram o uso do shadcn.

### 12.1 shadcn CLI

Comando principal do ecossistema. Sempre prefira `npx shadcn@latest` (ou `pnpm dlx shadcn@latest`).

| Comando | Para que serve |
|---|---|
| `init` | Bootstrap do projeto: cria `components.json`, `lib/utils.ts`, injeta CSS vars no `index.css`. Detecta framework (Next, Vite, Astro, Laravel, Remix). |
| `add <comp>` | Instala componentes (`button`, `dialog`, etc.). Flags: `-y` (skip prompt), `-o` (overwrite), `-a` (all), `--dry-run`. |
| `view <comp>` | Mostra o código do componente antes de instalar — útil para auditoria. |
| `search <reg> -q "..."` | Busca componentes em registries configuradas. |
| `list <reg>` | Lista tudo numa registry. |
| `build --output ./public/registry` | **Gera arquivos JSON da SUA registry** (para distribuir componentes próprios). |
| `apply <code> --only theme` | Aplica um preset (tema/fonte) sem reinstalar tudo. |
| `migrate rtl` / `migrate radix` / `migrate icons` | Codemods automáticos para refactors entre versões. |
| `mcp init --client claude` | Configura o MCP server local (ver 12.2). |

Exemplo de fluxo completo de migração:
```bash
npx shadcn@latest init                                # 1. Setup
npx shadcn@latest add button input dialog sonner      # 2. Primitivos
npx shadcn@latest view sidebar-07                     # 3. Inspeciona block
npx shadcn@latest add sidebar-07                      # 4. Instala block
npx shadcn@latest migrate radix                       # 5. Codemod p/ Radix nova
```

### 12.2 shadcn MCP Server

Servidor MCP (Model Context Protocol) que permite assistentes de IA (Claude Code, Cursor, VS Code, Codex) navegarem e instalarem componentes shadcn via **linguagem natural**.

**Instalação rápida (Claude Code):**
```bash
pnpm dlx shadcn@latest mcp init --client claude
```

Gera `.mcp.json` na raiz:
```json
{
  "mcpServers": {
    "shadcn": {
      "command": "npx",
      "args": ["shadcn@latest", "mcp"]
    }
  }
}
```

**Capacidades expostas como tools MCP:**

| Tool | Descrição |
|---|---|
| `get_project_registries` | Lê registries configuradas em `components.json`. |
| `list_items_in_registries` | Lista componentes disponíveis. |
| `search_items_in_registries` | Busca fuzzy por nome/descrição. |
| `view_items_in_registries` | Visualiza código antes de instalar. |
| `get_item_examples_from_registries` | Pega exemplos de uso oficiais. |
| `get_add_command_for_items` | Retorna o comando exato `shadcn add ...`. |
| `get_audit_checklist` | Checklist pós-instalação. |

**Exemplos de prompts no Claude Code:**
- "Mostre todos os componentes disponíveis no registry shadcn"
- "Encontre um formulário de login no registry oficial"
- "Adicione button, input, dialog e sonner ao projeto"
- "Construa uma página de configurações usando componentes shadcn"
- "Liste componentes do meu registry privado `@empresa`"

**Por que vale**: elimina a necessidade de saber de cor o nome de cada componente / decorar a doc. O assistente busca, sugere e instala.

### 12.3 Claude Code Skills oficiais (shadcn-ui)

O Claude Code carrega automaticamente skills específicas para shadcn quando o projeto tem `components.json`:

- **`shadcn-ui`** (plugin Claude Code): orienta o agente a usar primitivos da casa, compor com `cn()`, seguir kebab-case em `ui/`, preferir `Slot` para `asChild`, etc.
- **`vercel:shadcn`** (plugin Vercel): guidance específico para `init`, custom registries, theming, troubleshooting.

Como funciona: ao detectar `components.json` no repo, o Claude carrega instruções enxutas que evitam que o agente "reinvente" componentes que já existem, ou que estilize de forma inconsistente com o design system.

Ativação: instalar os plugins (`/plugin install ...` ou via marketplace do Claude Code).

### 12.4 Registries — distribuição de componentes próprios

**Registry oficial** (`@shadcn`): a default, sempre disponível.

**Registries customizadas**: você expõe seus próprios componentes via URL JSON.

```json
// components.json
{
  "registries": {
    "@empresa": "https://registry.empresa.com/{name}.json",
    "@privado": {
      "url": "https://internal.empresa.com/{name}.json",
      "headers": {
        "Authorization": "Bearer ${REGISTRY_TOKEN}"
      }
    }
  }
}
```

`.env.local`:
```
REGISTRY_TOKEN=seu_token_aqui
```

**Construindo sua própria registry:**
1. Cria estrutura `registry/<nome>.json` descrevendo arquivos, deps, css vars.
2. `shadcn build --output ./public/registry` gera os JSONs finais.
3. Hospeda em qualquer CDN/servidor estático (Vercel, S3, GitHub Pages).
4. Instala em qualquer projeto: `shadcn add @empresa/page-header`.

**Quando criar uma registry privada (cenário multi-projeto)**:
- ≥3 projetos compartilham os mesmos templates.
- Quer controle de versão centralizado.
- Precisa distribuir para repos sem acesso ao monorepo central.

### 12.5 Ferramentas de design system / theming

| Ferramenta | URL | O que faz |
|---|---|---|
| **TweakCN** | tweakcn.com | Editor visual de tema shadcn (gera CSS vars). Ideal para experimentar paletas. |
| **shadcn Theme Editor** | ui.shadcn.com/themes | Gerador oficial de tokens (presets prontos). |
| **shadcn/ui Blocks** | ui.shadcn.com/blocks | Blocks oficiais (dashboards, auth, sidebar). |
| **Aceternity UI** | ui.aceternity.com | Animações e effects (Framer Motion) — combina com shadcn. |
| **Magic UI** | magicui.design | Componentes "wow" (animados, hero sections). |
| **Origin UI** | originui.com | Coleção grande de blocks shadcn-compatíveis. |
| **shadcnspace** | shadcnspace.com | Blocks/templates premium + gratuitos. |
| **shadcn UI Kit** | shadcnuikit.com | Admin dashboards prontos. |

### 12.6 Stack complementar (não shadcn, mas se integra bem)

| Categoria | Ferramenta | Por quê |
|---|---|---|
| Form | React Hook Form + Zod | Padrão de fato no ecossistema shadcn. Tem `Form` component oficial. |
| Table | TanStack Table | shadcn `DataTable` é wrapper em cima. |
| Toast | sonner | Recomendado oficial (substituiu `toast` antigo). |
| Theme | next-themes | Suporte light/dark/system; mesmo fora de Next.js. |
| Animations | tailwindcss-animate (TW v3) / tw-animate-css (TW v4) | Keyframes que os componentes usam. |
| Icons | lucide-react | Padrão visual do shadcn. |
| Schema | Zod 4 | Validação + tipos. |
| Charts | recharts + shadcn `Chart` | Wrapper oficial. |
| Date | date-fns + react-day-picker | `Calendar` oficial usa essa stack. |
| Drawer mobile | vaul | `Drawer` oficial é wrapper. |
| OTP | input-otp | `InputOTP` oficial usa. |

### 12.7 Fluxo recomendado em projetos novos

1. `npx shadcn@latest init` — bootstrap.
2. `npx shadcn@latest mcp init --client claude` — habilita MCP.
3. Configurar `components.json` com registries privadas se for o caso.
4. Definir tokens/paleta em `src/index.css` (CSS vars) — fonte da verdade visual.
5. Instalar batch inicial: `shadcn add button input textarea dialog alert-dialog dropdown-menu popover command tooltip select separator scroll-area sonner card badge label`.
6. Criar `src/components/templates/` com 3-5 templates iniciais (PageHeader, EmptyState, ConfirmDialog, FormSection).
7. Documentar no `AGENTS.md` / `CLAUDE.md` a estrutura para guiar agentes de IA.

### 12.8 Cheat sheet de comandos

```bash
# Setup
npx shadcn@latest init
npx shadcn@latest mcp init --client claude

# Componentes
npx shadcn@latest add button input dialog
npx shadcn@latest view sidebar-07
npx shadcn@latest search @shadcn -q "form"
npx shadcn@latest list @shadcn

# Customização
npx shadcn@latest apply <preset-code> --only theme

# Registry própria
npx shadcn@latest build --output ./public/registry

# Codemods
npx shadcn@latest migrate radix
npx shadcn@latest migrate icons
```

---

## 13. Referências

- shadcn/ui docs: https://ui.shadcn.com/
- CLI reference: https://ui.shadcn.com/docs/cli
- MCP server: https://ui.shadcn.com/docs/mcp
- Registry: https://ui.shadcn.com/docs/registry
- Componentes oficiais: https://ui.shadcn.com/docs/components
- Empty (oficial, recente): https://ui.shadcn.com/docs/components/empty
- Blocks oficiais: https://ui.shadcn.com/blocks
- Vercel Academy — estendendo shadcn: https://vercel.com/academy/shadcn-ui/extending-shadcn-ui-with-custom-components
- Best practices 2026 (Medium): https://medium.com/write-a-catalyst/shadcn-ui-best-practices-for-2026-444efd204f44
- TweakCN: https://tweakcn.com/
- Origin UI: https://originui.com/
- shadcnspace: https://shadcnspace.com/
- Awesome Shadcn: https://github.com/topics/shadcn-ui
