# VS Code — Guia de Referência

Documentação de uso do Visual Studio Code focada em produtividade, configuração e reuso.

---

## Extensões

### Listar Extensões Instaladas

O VS Code oferece o CLI `code` para gerenciar extensões via terminal.

> **Atenção (Windows):** Ao executar dentro do terminal integrado do VS Code, use `code.cmd` em vez de `code` para evitar que uma nova janela seja aberta.

```powershell
# Listar todas as extensões instaladas
code.cmd --list-extensions

# Listar com versão
code.cmd --list-extensions --show-versions
```

**Via interface:**
`Ctrl+Shift+X` → digitar `@installed` na barra de busca

---

### Exportar Extensões com Links do Marketplace

Útil para compartilhar sua lista de extensões com outras pessoas ou documentar seu ambiente de desenvolvimento.

**Gera um arquivo `.txt` com um link por extensão:**

```powershell
code.cmd --list-extensions | ForEach-Object { 
    "https://marketplace.visualstudio.com/items?itemName=$_" 
} | Out-File extensions-links.txt
```

**Resultado esperado (`extensions-links.txt`):**
```
https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-typescript-next
https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode
https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme
...
```

Cada URL leva diretamente à página da extensão no Marketplace, onde a pessoa pode ler a descrição e instalar manualmente.

---

### Exportar Perfil Completo (Extensões + Configurações + Atalhos)

Quando o objetivo é replicar todo o ambiente de desenvolvimento (não apenas as extensões):

1. `Ctrl+Shift+P` → **"Export Profile"**
2. Selecione o que incluir: extensões, `settings.json`, keybindings, snippets
3. Exporte como:
   - **Arquivo `.code-profile`** → compartilhe diretamente
   - **GitHub Gist** → compartilhe o link gerado

**Para importar:**

1. `Ctrl+Shift+P` → **"Import Profile"**
2. Cole o link do Gist ou selecione o arquivo `.code-profile`

> O recurso de Perfis está disponível a partir do VS Code **v1.75**.

---

### Instalar uma Extensão pelo ID

Quando você conhece o ID da extensão (ex: `esbenp.prettier-vscode`):

```powershell
code --install-extension esbenp.prettier-vscode
```

O ID de cada extensão aparece na sua página no Marketplace e também na aba de detalhes dentro do VS Code.

---

## Referências

- [VS Code CLI Documentation](https://code.visualstudio.com/docs/editor/command-line)
- [VS Code Profiles](https://code.visualstudio.com/docs/editor/profiles)
- [VS Code Marketplace](https://marketplace.visualstudio.com/vscode)
