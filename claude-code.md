# Claude Code Desktop — Guia de Uso (Aplicativo Desktop)

## 1. Introdução

O ecossistema da Anthropic para desenvolvimento assistido por IA normalmente envolve duas peças:

- **Claude Desktop**: aplicativo desktop (chat + integrações locais via MCP).
- **Claude Code**: experiência focada em programação (edição/execução orientada por prompts, com contexto de projeto e automação de tarefas).

Na prática, muita gente usa os dois de forma complementar: Desktop para orquestração/contexto amplo e Claude Code para fluxo de implementação.

### Problema que resolve

- Reduz tempo em tarefas repetitivas (scaffold, refatoração, testes, documentação).
- Melhora descoberta em codebases grandes (busca semântica + leitura guiada).
- Acelera decisões técnicas com feedback em linguagem natural.

### Quando faz sentido aplicar

- Projetos com muitas regras internas e padrões de equipe.
- Tarefas de manutenção contínua (bugs, evolução de features, documentação viva).
- Times que já usam “AI pair programming” e querem padronizar fluxo.

---

## 2. Sintaxe Básica

Como o uso é conversacional, a “sintaxe” é o **formato de instrução**.

### Estrutura recomendada de prompt técnico

```text
Contexto: [stack, versão, restrições]
Objetivo: [resultado final]
Escopo: [arquivos/pastas afetadas]
Critérios de aceite: [testes, lint, comportamento]
Restrições: [não quebrar API, manter backward compatibility]
Saída esperada: [diff, resumo, checklist]
```

### Exemplo mínimo funcional

```text
Contexto: app desktop com Electron + React.
Objetivo: adicionar atalho Ctrl+K para abrir command palette.
Escopo: src/main, src/renderer.
Critérios: não quebrar atalhos existentes, cobrir com teste.
Saída: patch + explicação curta das decisões.
```

---

## 3. Exemplos Simples e Práticos

### Exemplo 1 — Entendimento rápido de código legado

```text
Mapeie como o módulo de autenticação funciona hoje.
Quero fluxo de login, refresh token e pontos de falha.
Entregue em 10 bullets com nomes de arquivos.
```

**Uso real:** onboarding técnico e diagnóstico de incidentes.

### Exemplo 2 — Refatoração segura

```text
Refatore este serviço para separar regra de negócio da camada de IO.
Mantenha assinatura pública.
Inclua testes de regressão para os cenários A, B e C.
```

**Uso real:** reduzir acoplamento sem quebrar contratos.

### Exemplo 3 — Geração de documentação

```text
Gere documentação do endpoint /orders/{id}:
request, response, erros, exemplos curl e JSON.
Baseie-se no código atual, não em suposições.
```

**Uso real:** manter docs sincronizadas com implementação.

### Exemplo 4 — Automação com workflows

```text
Crie workflow para:
1) ler issue,
2) propor plano,
3) alterar código,
4) rodar testes,
5) gerar resumo para PR.
```

**Uso real:** padronizar entrega e reduzir variação entre devs.

### Exemplo 5 — Revisão com regras internas

```text
Revise este diff com as regras do time:
- sem SQL inline
- logs estruturados
- tratamento explícito de erro
- sem segredo hardcoded
```

**Uso real:** qualidade consistente antes da revisão humana.

---

## 4. Boas Práticas

### 4.1 Defina contexto forte no início

- Informe stack/versão e convenções do projeto.
- Diga onde pode e não pode mexer.
- Declare critérios de aceite objetivos.

### 4.2 Use instruções versionadas no repositório

- Centralize padrões em arquivos de instrução (ex.: `AGENTS.md`, regras do projeto, comandos customizados).
- Mantenha histórico de mudanças dessas regras.

### 4.3 Trabalhe por ciclos curtos

- Planejar → editar → testar → resumir.
- Peça diffs pequenos e iterativos.

### 4.4 Peça rastreabilidade

- Sempre exigir “o que mudou” + “por que mudou” + “riscos”.
- Em mudanças críticas, pedir plano de rollback.

### 4.5 Segurança por padrão

- Nunca expor tokens, segredos, dados sensíveis em prompts.
- Validar comandos de shell antes de executar.

---

## 5. Limitações e Armadilhas

### Limitações técnicas

- Pode interpretar errado contexto implícito ou regra não documentada.
- Mudanças amplas sem escopo claro podem gerar regressões.
- Dependência da qualidade dos prompts e das regras do projeto.

### Erros comuns

1. **Prompt genérico demais** → saída vaga.
2. **Sem critérios de aceite** → difícil validar resultado.
3. **Sem boundaries de arquivo** → alterações fora de escopo.
4. **Confiar sem validar** → risco de bug/sintaxe/arquitetura.
5. **Misturar objetivos** (feature + refactor + migração) no mesmo pedido.

### Como evitar

- Especificar escopo de arquivos e testes obrigatórios.
- Exigir execução de validações (lint/test/build).
- Quebrar tarefas complexas em etapas explícitas.

---

## 6. Quando Usar e Quando Evitar

### Quando usar

- Criação de boilerplate com padrão já definido.
- Refactors estruturais com cobertura de teste.
- Documentação técnica a partir do código.
- Investigação de bugs com hipótese e plano de verificação.

### Quando evitar (ou usar com alta cautela)

- Hotfix crítico em produção sem revisão humana.
- Mudanças de segurança/compliance sem validação especializada.
- Decisões arquiteturais sem contexto de negócio.

---

## 7. Alternativas Modernas

## 7.1 Comparativo rápido (conceitual)

| Plataforma | Ponto forte | Limitação comum |
|---|---|---|
| Claude Code + Desktop | Forte em raciocínio e organização de contexto | Exige boas regras e prompts para consistência |
| Windsurf (IDE nativa) | Fluxo integrado de coding/agent no editor | Pode variar em governança conforme setup |
| Cursor | UX de edição assistida muito direta | Requer disciplina para evitar mudanças amplas |
| GitHub Copilot (Chat/Agent) | Ecossistema GitHub e integração ampla | Qualidade depende bastante do contexto fornecido |

## 7.2 Mapeamento de conceitos (Windsurf ↔ Claude Code)

- **Rules (Windsurf)** ↔ **Instruções do projeto** (`AGENTS.md`, guias de contribuição, padrões internos).
- **Workflows** ↔ **Comandos e rotinas versionadas** (fluxos reproduzíveis no repositório).
- **Skills** ↔ **Especializações reutilizáveis** (playbooks por domínio: backend, frontend, docs, testes).
- **Agents** ↔ **Subagentes especializados** para tarefas específicas (exploração, review, testes, docs).

---

## 8. Resumo Final

### Principais aprendizados

- Claude Code funciona melhor com **escopo explícito + critérios de aceite + regras versionadas**.
- O aplicativo desktop agrega valor como hub de contexto e integração local.
- Agentes, skills, commands, workflows e rules são o núcleo para escalar uso em equipe com previsibilidade.

### Guia rápido de consulta

1. Defina regras do projeto (estilo, arquitetura, segurança).
2. Crie comandos/workflows reutilizáveis para tarefas recorrentes.
3. Use agentes especializados por tipo de trabalho.
4. Sempre valide com testes/lint/build.
5. Registre resumo técnico das mudanças para auditoria.

---

## Apêndice A — Como configurar agentes, skills, commands, workflows e rules

> Estrutura conceitual para um ambiente local de engenharia assistida (adaptável ao seu setup).

### A.1 Rules (regras do projeto)

Crie um arquivo de regras no repositório (ex.: `AGENTS.md`) com:

- padrões de arquitetura,
- convenções de código,
- checklist de segurança,
- critérios mínimos de teste,
- políticas de commit/PR.

Exemplo de regra:

```markdown
## Regras obrigatórias
- Não alterar contratos públicos sem plano de migração.
- Toda mudança de negócio exige teste automatizado.
- Logs devem ser estruturados (JSON) sem dados sensíveis.
```

### A.2 Commands (comandos reutilizáveis)

Padronize comandos para tarefas recorrentes (ex.: gerar guia, revisar diff, abrir PR). Benefícios:

- consistência,
- menor esforço cognitivo,
- onboarding mais rápido.

### A.3 Skills (especializações)

Organize skills por domínio:

- `backend` (API, banco, performance)
- `frontend` (UX, acessibilidade, estado)
- `test-engineer` (estratégia e cobertura)
- `docs-specialist` (documentação técnica)

Cada skill deve descrever:

- quando usar,
- entradas esperadas,
- saída padrão,
- validações obrigatórias.

### A.4 Agents (subagentes)

Use subagentes para dividir responsabilidades:

- **explore** para mapear codebase,
- **code-reviewer** para checagem de qualidade,
- **test-engineer** para desenho/execução de testes,
- **docs-specialist** para documentação final.

Padrão eficaz: um agente principal orquestra e agentes especialistas executam partes.

### A.5 Workflows (fluxos similares ao Windsurf)

Workflow recomendado para feature:

1. Ler contexto (issue + regras).
2. Gerar plano técnico curto.
3. Implementar em etapas pequenas.
4. Executar testes/lint/build.
5. Produzir resumo para PR (o que/por quê/risco).

Checklist do workflow:

- [ ] Escopo definido
- [ ] Regras aplicadas
- [ ] Testes executados
- [ ] Riscos documentados
- [ ] Resumo final gerado

---

## Apêndice B — Prompt templates prontos

### Template de implementação

```text
Contexto: [stack + versão + restrições]
Tarefa: [feature/bug/refactor]
Escopo: [arquivos/pastas]
Regras: [arquitetura, segurança, estilo]
Validação: [testes, lint, build]
Entrega: [diff + resumo técnico + riscos]
```

### Template de revisão

```text
Revise este diff com foco em:
1) regressão funcional,
2) segurança,
3) performance,
4) aderência às regras do projeto.
Retorne: severidade, evidência, correção sugerida.
```

### Template de investigação de bug

```text
Sintoma: [erro observado]
Cenário: [como reproduzir]
Hipótese inicial: [suspeita]
Tarefa: localizar causa raiz e propor correção mínima segura.
Inclua testes de regressão.
```
