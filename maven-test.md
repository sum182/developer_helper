# Guia Rápido: Testes com Maven no IntelliJ

Este guia detalha como configurar e executar testes utilizando o Maven, otimizando o ciclo de feedback durante o desenvolvimento.

## ⚙️ Configurações de Execução (Run Configurations)

Para configurar no IntelliJ, vá em **Run > Edit Configurations**, clique no **+**, selecione **Maven** e preencha o campo **Run** conforme os cenários abaixo:

### ⚡ Cenário 1: Teste Rápido (Desenvolvimento)
Utilize este comando para validar suas alterações de código rapidamente sem esperar pelo processo de build completo.

- **Comando:** `clean test`
- **O que faz:** Apaga arquivos compilados antigos, recompila o código e executa todos os testes unitários.
- **Quando usar:** Toda vez que terminar uma funcionalidade ou correção e quiser validar se não quebrou nada.

### 🛡️ Cenário 2: Teste Completo (Pré-Commit / Build)
Utilize este comando para garantir que o projeto está íntegro e pronto para ser enviado ao repositório.

- **Comando:** `clean install`
- **O que faz:** Limpa, compila, executa testes e gera o arquivo `.jar` final, instalando-o no repositório local.
- **Quando usar:** Antes de realizar um commit ou push para a branch principal.

---

## 🎯 Execuções Específicas (Linha de Comando)

Você também pode rodar comandos diretamente no terminal do VS Code ou IntelliJ para focar em partes específicas:

### Rodar apenas uma Classe de Teste
```bash
mvn test -Dtest=NomeDaSuaClasseTest
```

### Rodar apenas um Método específico
```bash
mvn test -Dtest=NomeDaSuaClasseTest#nomeDoMetodoDeTeste
```

### Pular os testes (Apenas em emergência!)
Se precisar apenas buildar o projeto sem rodar os testes:
```bash
mvn clean install -DskipTests
```

---

## 💡 Dicas de Performance

1. **Evite `install` no dia a dia**: O comando `install` é pesado pois copia arquivos para o disco. Prefira `test` para feedback visual imediato.
2. **Use `-T` para builds paralelos**: Se seu computador tiver muitos núcleos, use `mvn clean test -T 1C` (1 thread por núcleo) para acelerar a compilação.
