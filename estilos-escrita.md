# Estilo de Escrita da IA (remover a cara de texto gerado)

Este documento registra o estudo e a implementação de regras de escrita para o Claude
Code, com o objetivo de eliminar os vícios de linguagem que denunciam texto gerado por
IA. Inclui a pesquisa feita, as alternativas avaliadas, o resultado aplicado e as
limitações conhecidas.

Data do estudo: 04/09/2026

---

## 1. Introdução

### Problema que resolve

A IA escreve com um conjunto de tiques reconhecíveis que atravessam todo texto que ela
produz, incluindo mensagem de commit, descrição de PR, card e documentação. Os três mais
incômodos no dia a dia são:

- Travessão (`—`) usado no meio da explicação, em vez de vírgula, ponto ou parênteses.
- Anglicismo técnico desnecessário, do tipo `dedup`, `backfill`, `leverage` e `surface`.
- Palavra de enfeite que não informa nada, do tipo `robusto`, `seamless` e
  `transformador`.

O efeito prático é ruim em dois lugares. No texto que vai para outra pessoa (card,
comunicado, PR), o leitor percebe que foi gerado e o texto perde credibilidade. No texto
técnico, o jargão em inglês esconde o que a coisa faz de fato.

### Quando faz sentido aplicar

Sempre que a IA escreve texto que você vai ler ou repassar. Não faz sentido aplicar a
conteúdo citado de arquivo, log ou saída de comando, que precisa ser reproduzido como
está na origem.

---

## 2. Onde a regra pode morar (alternativas avaliadas)

Quatro opções foram consideradas, com o resultado da avaliação.

### 2.1 Arquivo de regras globais (escolhida)

Local: `~/.claude/CLAUDE.md`

- Vale para toda sessão do Claude Code, em qualquer projeto.
- Entra como instrução, não como contexto de fundo, o que é mais forte.
- É um arquivo único, versionável e fácil de revisar.
- Não altera comportamento de ferramenta nenhuma, só a linguagem.

### 2.2 Memória do assistente

Local: `~/.claude/projects/<projeto>/memory/`

Persiste entre conversas, mas entra como contexto de fundo e não como instrução. Serve de
reforço, não de substituto do arquivo de regras.

### 2.3 Output style do Claude Code

Comando: `/output-style:new`

Descartada. O output style **substitui** o system prompt de engenharia de software do
Claude Code. Trocar as instruções que fazem a ferramenta trabalhar bem com código por um
texto de estilo é custo alto demais para resolver pontuação e vocabulário.

### 2.4 Preferências pessoais da conta em claude.ai

Local: perfil em claude.ai, campo "What personal preferences should Claude consider in
responses?"

É o lugar recomendado pela maioria dos artigos, mas vale para o chat do claude.ai e
**não** para as sessões do Claude Code. Quem usa os dois precisa configurar nos dois
lugares. O arquivo de regras cobre o Claude Code, as preferências do perfil cobrem o
chat.

---

## 3. O que a pesquisa mostrou

O material mais completo encontrado é o de Will Francis. O padrão se repete nas outras
fontes e tem três partes.

### 3.1 Lista explícita de palavras banidas

Pedido genérico do tipo "escreva natural" não funciona. É preciso nomear os termos. A
lista dele, em inglês: delve, dive into, navigate (figurado), underscore, bolster,
foster, harness, leverage, unpack, shed light on, pave the way, pivotal, groundbreaking,
cutting-edge, transformative, game-changing, robust, comprehensive, seamless, intricate,
nuanced, holistic, testament, landscape, realm.

### 3.2 Construções de frase banidas

A parte que os artigos focados só em pontuação esquecem, e que responde por boa parte da
cara de IA.

- "No mundo de hoje, cada vez mais acelerado..."
- "É importante notar que...", "Vale ressaltar que..."
- "Não é só X, é Y" e "Isso não é sobre X. É sobre Y."
- Abertura vazia do tipo "Vamos explorar" e "Vamos destrinchar"

### 3.3 Regra de pontuação com substituto declarado

Ele usa "no máximo um travessão por resposta". Os artigos dedicados ao travessão
recomendam listar as substituições permitidas em vez de só proibir, porque assim o modelo
tem para onde ir. Vírgula para interrupção curta, ponto para separar ideias, parênteses
para informação complementar.

Ele também bane o formato de lista `**Termo em negrito**: explicação`, que é outro tique
reconhecível.

### 3.4 Limitação relatada pelo autor

Funciona perceptivelmente melhor, principalmente pela restrição de vocabulário, mas não é
solução completa. Ainda escapa coisa e ainda é preciso revisar.

### 3.5 Consenso entre as fontes

Regra vaga não pega, regra concreta pega. "Escreva bem" não produz efeito, "no máximo um
travessão por resposta" produz.

---

## 4. Aproveitamento de um prompt externo de revisão

Um prompt de revisão de texto já em uso (originalmente num GPT customizado) foi analisado
para extrair regras. O resultado da triagem.

### 4.1 O que valia extrair

- **Entregar o texto pronto, sem comentário em volta.** O trecho mais útil. Evita o vício
  de embrulhar a mensagem de commit em "Aqui está a mensagem" antes e "Ajuste como
  preferir" depois, obrigando o leitor a caçar onde começa o que ele vai copiar.
- **Preservar o significado ao revisar.** Melhorar a forma sem inventar conteúdo, sem
  acrescentar informação que não foi dada e sem cortar o que o autor quis dizer.
- **Ajustar o registro conforme o público.** Card técnico e mensagem para área de negócio
  pedem vocabulário diferente.
- **Tom profissional e neutro**, sem excesso de formalidade.

### 4.2 O que já estava coberto

"Usar terminologia técnica apropriada quando o contexto for de TI" é o mesmo argumento da
regra que mantém os termos em inglês. O prompt externo reforçou manter essa parte.

### 4.3 Destino final do prompt

As quatro regras extraídas foram consideradas excesso para a regra global, que precisa
ser curta. O prompt inteiro virou a skill `revisor-textos`, acionada sob demanda, o que
preserva o comportamento completo sem inflar o arquivo de regras.

### 4.4 Detalhe encontrado

O próprio prompt de revisão continha um travessão ("comentários, explicações ou
justificativas — entregue somente o texto final"). Prompt que gera texto e contém o vício
que se quer evitar acaba propagando o vício para o resultado. Na versão que virou skill,
o travessão foi removido.

---

## 5. Resultado aplicado

### 5.1 Arquivos alterados

| Arquivo | O que recebeu |
| --- | --- |
| `~/.claude/CLAUDE.md` | Seção de estilo, item na lista "Aplicar sempre", título novo |
| `<repo dev-ai>/claude-code/CLAUDE.md` | Igual ao global (cópia versionada) |
| `<repo dev-ai>/kilo-code/rules/global_rules.md` | Só o título novo |

O título das três cópias passou de `# CLAUDE.md — Regras Globais` e `# global_rules.md`
para `# Regras Globais`. Dois motivos. O nome do arquivo no título é redundante, e o
título neutro permite comparar as cópias das duas ferramentas, que passam a diferir
apenas na frase de descrição.

### 5.2 Seção final, como ficou

```markdown
## Estilo de Escrita

Vale para todo texto que você escreve para mim, respostas no chat, mensagens de
commit, descrição de PR e de card, documentação e comentários de código. Não vale
para conteúdo citado de arquivo, log ou saída de comando, que deve ser reproduzido
exatamente como está na origem.

### Pontuação

- Travessão (`—`) e meia-risca (`–`) são **proibidos**, sem exceção. Nem em título,
  nem em lista, nem em código de exemplo escrito por você.
- Dois-pontos (`:`) só antes de lista. No meio de frase corrida é proibido.
- No lugar do travessão e dos dois-pontos (`:`), usar vírgula para interrupção
  curta, ponto para separar ideias completas e parênteses para informação
  complementar.
- Se a frase só funcionar com travessão ou com dois-pontos no meio, reescreva a
  frase.

### Palavras banidas

Substituir sempre:

- `dedup`, `deduplicar` → remover duplicados
- `backfill` → preencher dados retroativos
- `leverage`, `alavancar` → usar, aproveitar
- `surface` (como verbo) → mostrar, exibir
- `delve`, `deep dive`, `mergulhar fundo` → analisar
- `onboarding` → integração (de pessoa ou de parceiro)
- `robusto`, `seamless`, `holístico`, `sinergia`, `comprehensive`, `transformador`,
  `game-changing`, `cutting-edge`, `state-of-the-art` → dizer concretamente o que a
  coisa faz
- `pivotal`, `crucial`, `fundamental` usados como enfeite → cortar a palavra

### Termos técnicos em inglês

Termo em inglês que já é o nome da coisa no dia a dia do time continua em inglês.
Alguns exemplos são deploy, commit, pull request, endpoint, cache, log, build e
release, além de nomes de ferramentas, bibliotecas e frameworks. Não são todos, o
critério é como o time fala. Não traduzir esses termos nem inventar tradução para
eles.
```

### 5.3 Como validar que passou a valer

O arquivo de regras é lido no início de cada sessão. Depois de editar, é preciso reabrir
a sessão para o conteúdo novo entrar em contexto. Duas formas de conferir.

Pelo arquivo em disco:

```bash
head -14 ~/.claude/CLAUDE.md
```

Pela resposta da IA, pedindo um texto de teste sobre tema que convide ao vício (rotina de
importação, deploy, cache) e conferindo quatro pontos. Nenhum travessão, nenhum
dois-pontos no meio de frase, os termos banidos substituídos e os termos em inglês
mantidos sem tradução forçada.

---

## 6. Decisões tomadas e descartadas

Registro do que ficou fora e por quê, para não refazer a discussão depois.

| Item | Decisão | Motivo |
| --- | --- | --- |
| Proibir travessão totalmente | Mantido | Mais fácil de cumprir e de fiscalizar que "no máximo um por resposta" |
| Proibir dois-pontos no meio da frase | Mantido | Preferência pessoal, permitido só antes de lista |
| Banir o formato `**Termo**: explicação` | Descartado | Ajuda a leitura em resposta longa e técnica |
| Seção de tom (frases curtas, sem preâmbulo) | Descartada | Considerada excesso de regra |
| Lista de construções banidas | Descartada | Idem |
| Seção de registro e público | Descartada, virou skill | Idem |
| Seção de texto para colar | Descartada, virou skill | Idem |
| Termo `insights` na lista de banidos | Descartado | Não incomodava |
| Seção de termos em inglês | Mantida, encurtada | Sem ela, a IA erra para o outro lado e traduz demais |
| Lista fechada de termos em inglês | Trocada por critério | Lista de 17 itens parece exaustiva e gera dúvida sobre o que não está nela |

O ponto mais importante da tabela é a penúltima linha. A regra que bane anglicismo,
sozinha, empurra a IA a traduzir o que não deve, produzindo "solicitação de incorporação"
em vez de pull request. A seção de termos em inglês existe para travar essa direção.

---

## 7. Limitações conhecidas

- **Não é filtro, é instrução.** Reduz muito, não zera. Quando escapar, corrigir na hora
  e acrescentar a linha correspondente na regra.
- **Não alcança texto de terceiros.** Skill, plugin e mensagem do próprio sistema usam
  travessão e jargão em inglês, e a regra não afeta isso.
- **Não alcança conteúdo citado.** Código, saída de comando e trecho de arquivo aparecem
  como estão na origem, por decisão explícita da regra.
- **Contradição interna do arquivo.** As regras antigas de Backend e Frontend, escritas
  antes deste estudo, contêm sete travessões. Não afeta o comportamento, mas fica
  estranho para quem abre o arquivo.
- **Duas cópias para manter.** Claude Code e Kilo Code leem arquivos diferentes com o
  mesmo conteúdo. Editar um e esquecer o outro é o erro mais provável.

---

## 8. Como manter

A seção cresce por demanda. O ciclo é simples.

1. Um termo ou vício escapa e incomoda.
2. Acrescentar a linha correspondente na seção, no formato `termo → substituto`.
3. Replicar no arquivo da outra ferramenta, se estiver mantendo as duas cópias.
4. Reabrir a sessão para a regra nova entrar em contexto.

---

## 9. Resumo final

### Principais aprendizados

- Regra concreta funciona, regra vaga não. Nomear o termo e o substituto é o que produz
  efeito.
- Proibir sem oferecer alternativa gera texto pior. Toda proibição de pontuação precisa
  declarar o que usar no lugar.
- Regra de estilo pede a regra oposta como contrapeso. Banir anglicismo sem proteger os
  termos consagrados faz a IA traduzir o que não devia.
- O arquivo de regras precisa obedecer a si mesmo. Documento que proíbe travessão e usa
  travessão perde autoridade com quem lê.
- Menos regra, mais efeito. A versão final ficou com três blocos, depois de descartar
  cinco seções que pareciam úteis no rascunho.
- Regra global precisa ser curta, comportamento longo vira skill. Foi o destino do prompt
  de revisão.
- Alterar o arquivo não basta, é preciso reabrir a sessão para a regra valer.

### Referências

- [How to Stop Claude Writing Like an AI](https://willfrancis.com/how-to-stop-claude-writing-like-an-ai/)
- [Claude Em-Dash Problem: Why Claude Uses Them & How to Fix](https://www.context-link.ai/blog/claude-em-dash-remover)
- [How to Stop Claude From Using Em Dashes](https://www.getsyspro.com/newsroom/how-to-stop-claude-from-using-em-dashes/)
- [Claude Code output styles (documentação Anthropic)](https://docs.anthropic.com/en/docs/claude-code/output-styles)
- [Modifying system prompts (documentação Anthropic)](https://docs.anthropic.com/en/docs/claude-code/sdk/modifying-system-prompts)
- [I Trained Claude Code How to Write Like Me](https://levelupwithai.substack.com/p/i-trained-claude-code-how-to-write)
- [Fix Claude's Em Dash Problem With One Prompt Line](https://aiwave.dev/blog/2026-05-06-short-your-claude-output-has-an-em-dash-problem-heres-the-one-prompt-line-that-fixes-it)
