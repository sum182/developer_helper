# magic/ui — Guia de Uso (frontend)

## 1. Introdução
- **Contexto do tema:** Magic UI é uma coleção de componentes e efeitos visuais para React/Next.js, com foco em interfaces modernas, animações suaves e composição rápida de telas.
- **Problema que resolve:** acelera a construção de UI atrativa sem começar do zero em CSS/animações, reduzindo tempo de design para times que precisam entregar produto com aparência profissional.
- **Quando faz sentido aplicar:** em frontends de ERP, painéis internos e produtos SaaS quando você quer melhorar experiência visual (listas, filtros, modais, dashboards) sem sacrificar produtividade.

No contexto de programação agêntica com IA, Magic UI ajuda a:
- criar estados visuais claros para ações do agente ("processando", "sugerindo", "erro", "concluído");
- destacar feedback em tempo real (toasts, banners, cards de recomendação);
- aumentar confiança do usuário final com UX consistente em fluxos assistidos por IA.

## 2. Sintaxe Básica
- **Conceitos essenciais:**
  - componentes reutilizáveis (UI building blocks);
  - composição por props (`className`, variações, children);
  - animações/transições para feedback de estado;
  - integração com Tailwind (na maioria dos setups).

- **Exemplo mínimo funcional:** card de resumo para ERP com ação de IA.

```tsx
import { useState } from "react"

export function AiSuggestionCard() {
  const [loading, setLoading] = useState(false)

  async function gerarSugestao() {
    setLoading(true)
    try {
      // chamada para seu endpoint/agente
      await new Promise((r) => setTimeout(r, 1200))
    } finally {
      setLoading(false)
    }
  }

  return (
    <div className="rounded-xl border bg-white p-4 shadow-sm">
      <h3 className="text-sm font-semibold">Sugestão de IA para cadastro</h3>
      <p className="mt-1 text-sm text-zinc-600">
        Padronize descrição e categoria automaticamente.
      </p>

      <button
        onClick={gerarSugestao}
        disabled={loading}
        className="mt-3 rounded-md bg-zinc-900 px-3 py-2 text-sm text-white disabled:opacity-60"
      >
        {loading ? "Gerando..." : "Gerar sugestão"}
      </button>
    </div>
  )
}
```

## 3. Exemplos Simples e Práticos
### Exemplo 1 — Lista de pedidos com destaque inteligente
Use componentes visuais para destacar linhas com risco (atraso, valor alto, inconsistência detectada pela IA).

```tsx
<tr className={pedido.risco ? "bg-amber-50" : ""}>
  <td>{pedido.numero}</td>
  <td>{pedido.cliente}</td>
  <td>{pedido.status}</td>
</tr>
```

### Exemplo 2 — Modal de confirmação com contexto do agente
Antes de atualizar muitos registros, abra modal com resumo: "A IA identificou 37 itens para correção".

```tsx
<Dialog open={open} onOpenChange={setOpen}>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>Confirmar atualização em lote</DialogTitle>
    </DialogHeader>
    <p className="text-sm text-zinc-600">A IA sugeriu corrigir 37 descrições.</p>
    <Button onClick={confirmar}>Aplicar alterações</Button>
  </DialogContent>
</Dialog>
```

### Exemplo 3 — CRUD com assistente de preenchimento
No formulário de produto, botão "Completar com IA" para sugerir categoria, tags e descrição.

```tsx
<Button variant="outline" onClick={preencherComIA}>
  Completar com IA
</Button>
```

### Exemplo 4 — Empty state orientado por ação
Em listas vazias, use bloco visual com CTA claro: "Importar planilha" ou "Gerar primeiro registro com IA".

### Exemplo 5 — Toasts para feedback de ação agêntica
Após ação automatizada, mostrar resultado objetivo: quantidade processada, falhas e próximo passo.

## 4. Boas Práticas
- Defina **design tokens** (cores, espaçamento, tipografia) para manter consistência em todo ERP.
- Use animações com moderação: priorize feedback de estado, não efeito decorativo.
- Padronize componentes críticos: tabela, filtro, paginação, modal, drawer e toast.
- Em fluxos com IA, sempre mostre:
  - o que foi sugerido;
  - o que será aplicado;
  - opção de revisão humana.
- Garanta acessibilidade (foco em teclado, contraste, `aria-*`, rótulos claros).
- Para listas grandes, combine UI rica com performance: virtualização, debounce em busca e loading skeleton.

## 5. Limitações e Armadilhas
- **Risco de overdesign:** excesso de animação piora leitura em telas densas de ERP.
- **Dependência de estilo sem padrão:** sem guideline, cada tela fica com "cara" diferente.
- **Custo de performance:** efeitos pesados em tabelas com muitos itens podem degradar UX.
- **Acoplamento visual com lógica:** evitar componente bonito mas difícil de testar/manter.
- **Erro comum em IA + UI:** aplicar sugestões automaticamente sem confirmação em ações sensíveis (financeiro, fiscal, estoque).

Como evitar:
- adotar biblioteca base + convenções internas;
- medir performance (LCP/INP) em páginas críticas;
- exigir modo "pré-visualizar antes de aplicar" para automações.

## 6. Quando Usar e Quando Evitar
**Quando usar**
- necessidade de elevar qualidade visual rapidamente;
- produto com alto volume de CRUD que precisa UX mais clara;
- fluxos com IA exigindo comunicação de estado e confiança do usuário.

**Quando evitar**
- projeto extremamente restrito em bundle/performance sem otimização;
- equipe sem disciplina de design system (alto risco de inconsistência);
- telas puramente operacionais onde simplicidade total é prioridade e o ganho visual é irrelevante.

## 7. Alternativas Modernas
- **shadcn/ui:** excelente base para sistemas ERP/CRUD; muito flexível e fácil de adaptar.
- **MUI (Material UI):** ecossistema robusto, componentes prontos e maturidade empresarial.
- **Chakra UI:** API amigável, produtividade alta para apps internos.
- **Mantine:** boa cobertura de componentes e utilitários para dashboards.

### Comparativo rápido
- **Magic UI:** foco maior em estética e efeitos modernos.
- **shadcn/ui:** foco em composição e controle fino (muito usado em produtos SaaS).
- **MUI/Chakra/Mantine:** foco em produtividade com kit mais fechado.

## 8. Resumo Final
- Magic UI é útil para acelerar interfaces frontend com aparência moderna e melhor percepção de qualidade.
- Em ERP/CRUD/listas/modais, o ganho real vem de consistência visual + feedback claro de estado.
- Em experiências com IA/agentes, priorize transparência: sugestão, confirmação e rastreabilidade.
- Use animações como suporte à usabilidade, não como fim estético.
- Combine Magic UI com padrões de design system e métricas de performance para adoção segura em produção.
