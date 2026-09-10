# Git Worktree com IntelliJ IDEA e Claude Code — Guia de Uso (Git 2.55 / IntelliJ 2026.x / Claude Code 2.1.x, Windows)

## 1. Introdução

Um worktree é um segundo (terceiro, quarto) diretório de trabalho ligado ao **mesmo** repositório. Cada worktree tem seus próprios arquivos e sua própria branch em checkout, mas todos compartilham o mesmo `.git` (histórico, remotes, stash, config, objetos).

Na prática, é o `git clone` sem clonar. Você deixa de precisar de `git stash` e de trocar de branch no meio de uma tarefa.

### Problema que resolve

- Trabalhar em duas branches ao mesmo tempo sem `stash` e sem perder o estado da IDE.
- Rodar um hotfix urgente sem interromper a feature em andamento.
- Rodar **várias sessões do Claude em paralelo** sem que uma sessão sobrescreva os arquivos da outra.
- Comparar comportamento entre branches com dois processos rodando lado a lado (duas portas, dois builds).

### Quando faz sentido aplicar

- Você usa Claude para tarefas longas e quer continuar codando manualmente enquanto ele trabalha.
- Build pesado (Maven/Gradle, `node_modules`) em que trocar de branch invalida o cache de compilação.
- Revisão de PR sem sujar a branch atual.

Quando **não** compensa, veja a seção 6.

---

## 2. Sintaxe Básica

### Comandos essenciais do git

```bash
# Criar worktree em branch nova
git worktree add C:/Projetos/worktrees/app-feature-x -b feature/x

# Criar worktree a partir de branch que já existe
git worktree add C:/Projetos/worktrees/app-hotfix hotfix/123

# Listar
git worktree list

# Remover (o diretório precisa estar limpo, senão use --force)
git worktree remove C:/Projetos/worktrees/app-feature-x

# Limpar registros órfãos (pastas apagadas na mão)
git worktree prune
```

Regra de ouro: **um branch só pode estar em checkout em um worktree por vez**. O git recusa e avisa qual worktree já está usando.

### Estrutura no disco

```
C:\Projetos\
├── app\                      <- checkout principal (contém o .git real)
│   └── .git\                    worktrees\  (metadados dos links)
└── worktrees\
    ├── app-feature-x\        <- .git é um ARQUIVO apontando para o principal
    └── app-hotfix\
```

---

## 3. Exemplos Simples e Práticos

### Exemplo 1 — Hotfix no meio de uma feature

```bash
# Você está em C:\Projetos\app com trabalho não commitado. Não mexa nele.
git worktree add C:/Projetos/worktrees/app-hotfix -b hotfix/PROJ-123 origin/main
```

Abra a pasta nova no IntelliJ, corrija, commite, faça o push e remova o worktree. O projeto principal continua exatamente como estava.

### Exemplo 2 — Criar pela UI do IntelliJ (fluxo com mouse)

1. `Alt+9` para abrir a Git tool window.
2. Aba **Worktrees**, botão **New** (ou menu **Git | New Worktree**).
3. Preencha **From branch**, **Project name** e **Location**.
4. O IntelliJ abre o worktree como **projeto separado**, em outra janela.

Para alternar: aba **Worktrees**, clique duplo no worktree desejado. Também dá para clicar com o botão direito em uma branch no **Log** ou no widget de branch e escolher **Open Worktree**.

Para remover: aba **Worktrees**, selecione, botão **Delete**. Não é possível excluir o worktree principal nem o que está aberto.

### Exemplo 3 — Claude Code criando o worktree sozinho

```bash
claude --worktree feature-auth
```

Cria `.claude/worktrees/feature-auth/` na raiz do repositório, em uma branch nova `worktree-feature-auth`, e inicia a sessão lá dentro. Rode o mesmo comando com outro nome em outro terminal para uma segunda sessão isolada.

Dentro de uma sessão você também pode simplesmente pedir "trabalhe em um worktree", e o Claude usa a ferramenta `EnterWorktree`.

### Exemplo 4 — Claude usando um worktree que VOCÊ criou

```bash
git worktree add C:/Projetos/worktrees/app-feature-x -b feature/x
cd C:/Projetos/worktrees/app-feature-x
claude
```

Esse é o fluxo que respeita a sua organização de diretórios. O Claude enxerga o worktree como o projeto, e o `.git` compartilhado continua funcionando normalmente (commit, push, log).

### Exemplo 5 — Levar o `.env` para dentro de cada worktree novo

Worktree é checkout limpo, então nada que está no `.gitignore` vem junto. Crie um `.worktreeinclude` na raiz do projeto (sintaxe de `.gitignore`):

```text
.env
.env.local
config/secrets.json
```

O Claude copia esses arquivos em todo worktree que ele criar.

---

## 4. Boas Práticas

### O diretório padrão, ponto a ponto

Essa é a pergunta central, e as três fontes não concordam entre si.

| Fonte | Padrão | Racional declarado |
|---|---|---|
| **Git** | Nada é imposto. Os exemplos oficiais usam irmão do projeto, `git worktree add ../project-feature-a` | O git só exige um diretório vazio ou inexistente |
| **IntelliJ IDEA** | Você escolhe o **Location** na caixa de diálogo. A documentação diz explicitamente para **evitar aninhar**, "não é recomendado criar um worktree dentro do diretório do projeto atual" | Se ficar dentro, a IDE trata como projeto multi-root e a integração quebra |
| **Claude Code** | `.claude/worktrees/<nome>/` **dentro** do repositório | Descoberta automática, retomada de sessão (`--resume`) e faxina automática dependem desse caminho conhecido |
| **Skill superpowers** | `.worktrees/` na raiz, obrigatoriamente no `.gitignore` | Mesmo raciocínio de descoberta, com pasta oculta |

### A sua proposta está correta

Criar `C:\Projetos\worktrees\` **um nível acima do projeto** é a opção mais segura e é exatamente o que o IntelliJ e os exemplos do git recomendam. Os ganhos são reais, não teóricos.

- **Nada varre o worktree por engano.** Lint, `mvn test`, `gradle build`, ESLint, indexação da IDE, `find`, scanners de segurança e Docker build context enxergam apenas a árvore do projeto. Colocar em `.gitignore` resolve para o git, mas **não** resolve para essas ferramentas, que na maioria ignoram o `.gitignore`.
- **Sem projeto multi-root no IntelliJ.** Com a pasta dentro do projeto, a IDE tenta anexar o worktree ao mesmo projeto e a integração Git visual fica confusa.
- **Sem risco de commit acidental** se alguém esquecer a linha do `.gitignore`.
- **Sem custo em disco extra.** Aninhado ou irmão, o `.git` é o mesmo. A localização não muda nada para o git.

O único preço é o Claude Code. Fora de `.claude/worktrees/`, ele pede aprovação ao entrar com `EnterWorktree`, e a retomada de sessão (`--resume`) tem regras mais estritas (precisa ser lançada a partir do checkout principal em alguns casos). Nada disso impede o uso, só adiciona um clique.

### Recomendação prática (modelo híbrido)

Use os dois caminhos, separando por dono do worktree.

1. **Worktree que você vai abrir no IntelliJ e trabalhar com a mão** vai para `C:\Projetos\worktrees\<repo>-<assunto>`. Crie pela UI da IDE ou pelo `git worktree add`.
2. **Worktree efêmero de sessão do Claude** deixe no padrão dele, `.claude/worktrees/`. É descartável, dura horas, e o Claude limpa sozinho ao sair.
3. Coloque no `.gitignore` do projeto de qualquer forma:

```gitignore
.claude/worktrees/
.worktrees/
```

Se você quiser **um único padrão** e mandar o Claude obedecer o seu diretório, configure um hook `WorktreeCreate` no `settings.json`, que substitui a lógica padrão de criação e permite apontar para fora do repositório. Vale o esforço só se worktree virar rotina diária no time.

### Convenção de nomes

Use `<repo>-<tipo>-<id>`, por exemplo `app-hotfix-PROJ-123`. No Windows, o caminho completo entra no limite de 260 caracteres com facilidade em projeto Java com pacotes profundos, então nome curto ajuda.

### Configuração útil do git

```bash
git config --global worktree.useRelativePaths true
```

Grava os links entre worktree e repositório com caminho relativo (Git 2.48+). Assim, mover a pasta `C:\Projetos\` inteira não quebra os worktrees.

### Ajustes específicos do IntelliJ

- **Nunca versione `.idea/workspace.xml`.** Se ele estiver commitado, o worktree herda o mesmo `ProjectId` e a IDE embaralha os dois projetos. Apague o arquivo no worktree ou remova a linha `ProjectId`, e adicione ao `.gitignore`.
- Cada worktree é um projeto separado, então a IDE **reindexa** e faz download de dependências de novo. Espere de 1 a 3 minutos no primeiro `open` de um projeto Java grande.
- O suporte nativo cobre projeto com **um único repositório**. Monorepo com vários repositórios git ainda não é suportado nativamente, use a linha de comando nesse caso.
- Se a sua versão da IDE for antiga e não tiver a aba **Worktrees**, existem os plugins `Git Worktree` e `Another Git Worktree` no Marketplace. Confira antes se a aba já existe, o suporte nativo tornou o plugin desnecessário.

### Isolamento além dos arquivos

Worktree isola arquivos, **não** isola banco, porta, variável de ambiente nem container. Duas sessões rodando migration no mesmo banco local vão brigar. Para tarefas que tocam infraestrutura, use schema separado, `.env` com porta diferente ou container por worktree.

---

## 5. Limitações e Armadilhas

| Armadilha | O que acontece | Como evitar |
|---|---|---|
| Mesma branch em dois worktrees | Git recusa o checkout | Uma branch por worktree, é por design |
| Pasta apagada no Explorer | Registro órfão em `.git/worktrees` | `git worktree prune` |
| `node_modules`, `target`, `build` | Não são compartilhados, cada worktree recompila do zero | Aceite o custo, ou use cache global (pnpm store, repositório local do Maven) |
| `.env` e arquivos ignorados | Não aparecem no worktree novo | `.worktreeinclude` ou cópia manual |
| Worktree dentro do projeto | Build, lint e IDE varrem a pasta; IntelliJ vira multi-root | Use diretório irmão |
| `.idea/workspace.xml` versionado | Projetos com o mesmo `ProjectId`, IDE confusa | Remover do versionamento |
| Caminho longo no Windows | Erro de build ou de checkout acima de 260 caracteres | Nomes curtos, ou `git config --system core.longpaths true` |
| Git LFS com `git lfs install --local` | O Claude cria o worktree com arquivos ponteiro em vez do conteúdo real | Rode `git lfs pull` dentro do worktree |
| Junction/symlink dentro do worktree no Windows | Remoção apaga só o link, não o destino (comportamento correto, mas surpreende) | Confira o que sobrou antes de recriar |
| Sessão `-p` (não interativa) do Claude | Não faz limpeza ao sair e deixa lock no worktree | `git worktree unlock` e depois `git worktree remove` |
| Muitas sessões paralelas | Custo cognitivo alto, revisão vira gargalo | Duas ou três sessões é o teto sustentável |

---

## 6. Quando Usar e Quando Evitar

### Use worktree quando

- Precisa de duas branches vivas ao mesmo tempo (feature + hotfix).
- Vai rodar o Claude em tarefa longa e quer continuar trabalhando.
- O build é caro e trocar de branch invalida o cache de compilação.
- Quer revisar um PR rodando de verdade, sem tocar no seu ambiente atual.

### Evite worktree quando

- A troca de branch é rápida e o projeto é pequeno, `git switch` resolve.
- A tarefa exige serviço externo com estado único (um banco local só, uma porta fixa) e você não vai isolar.
- O repositório usa submódulos pesados, o suporte de worktree a submódulo ainda é irregular.
- Você faria apenas um checkout descartável de leitura, `git stash` ou `git show branch:arquivo` são mais baratos.

### Worktree contra as alternativas

| Situação | Melhor ferramenta |
|---|---|
| Interrupção de 5 minutos | `git stash` |
| Duas branches por horas ou dias | **worktree** |
| Ambientes totalmente independentes (outro remote, outro estado do `.git`) | `git clone` separado |
| Ver um arquivo de outra branch | `git show branch:caminho/arquivo` |

---

## 7. Alternativas Modernas

- **Sessões paralelas do Claude Desktop.** No app desktop, cada sessão nova já ganha o próprio worktree automaticamente, sem comando nenhum. É o caminho de menor atrito se o seu uso do Claude for pelo desktop.
- **Subagentes com `isolation: worktree`.** No frontmatter de um agente em `.claude/agents/`, essa linha faz cada execução rodar em worktree temporário próprio, com limpeza automática. Útil para refatoração mecânica em muitos arquivos.
- **`git worktree` + Dev Containers.** Um container por worktree resolve o isolamento de banco e porta que o worktree sozinho não resolve.
- **Plugins do Marketplace JetBrains.** `Git Worktree` e `Another Git Worktree` oferecem tool window dedicada. Hoje são redundantes com o suporte nativo, mas podem trazer ações extras.
- **Codespaces / ambientes remotos.** Isolamento total, custo maior, faz sentido quando a máquina local não aguenta dois builds.

---

## 8. Resumo Final

### Respostas diretas às suas perguntas

1. **Qual o diretório padrão?** Não existe um padrão único. O git não impõe nada, o IntelliJ deixa você escolher e pede para não aninhar, o Claude usa `.claude/worktrees/` dentro do repo.
2. **O que o Claude recomenda?** `.claude/worktrees/<nome>/` na raiz do projeto, criado com `claude --worktree <nome>`, adicionado ao `.gitignore`.
3. **O que o IntelliJ recomenda?** Qualquer local **fora** do diretório do projeto atual. A documentação é explícita contra o aninhamento.
4. **O que o git recomenda?** Os exemplos oficiais usam diretório irmão, `../project-feature-a`. Nenhuma imposição.
5. **A sua ideia de `C:\Projetos\worktrees\` está certa?** Sim. É a opção alinhada ao IntelliJ e ao git, e resolve de verdade o risco de lint, build e indexação varrerem cópias do projeto, coisa que o `.gitignore` sozinho não resolve.

### Guia rápido

```bash
# criar (padrão recomendado, diretório irmão)
git worktree add C:/Projetos/worktrees/app-feature-x -b feature/x

# criar via Claude (efêmero, dentro do repo)
claude --worktree feature-x

# listar / remover / limpar
git worktree list
git worktree remove C:/Projetos/worktrees/app-feature-x
git worktree prune

# caminhos relativos, sobrevive a mover a pasta
git config --global worktree.useRelativePaths true
```

No IntelliJ, com o mouse: `Alt+9` > aba **Worktrees** > **New** para criar, clique duplo para alternar, **Delete** para remover. Aponte o campo **Location** para `C:\Projetos\worktrees\`.

Não esqueça do `.gitignore`:

```gitignore
.claude/worktrees/
.worktrees/
.idea/workspace.xml
```

---

## Fontes

- [Use Git worktrees, documentação IntelliJ IDEA](https://www.jetbrains.com/help/idea/use-git-worktrees.html)
- [Run parallel sessions with worktrees, documentação Claude Code](https://code.claude.com/docs/en/worktrees)
- [git-worktree, documentação oficial do Git](https://git-scm.com/docs/git-worktree)
- [Git Worktree, plugin JetBrains Marketplace](https://plugins.jetbrains.com/plugin/23813-git-worktree/versions)
