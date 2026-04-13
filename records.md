# Estudo Completo: Records na Linguagem Java

Este documento é um guia prático e de referência para aprender, aplicar e reutilizar **Java Records** no dia a dia.

---

## 1) O que são Records?

`record` é um tipo especial de classe introduzido como preview no Java 14 e estabilizado no **Java 16**. Ele foi criado para modelar **dados imutáveis** de forma concisa, eliminando boilerplate de classes com muitos campos e pouco comportamento.

### Problema que o Record resolve

Em classes tradicionais, era comum escrever manualmente:
- construtor
- getters
- `equals()`
- `hashCode()`
- `toString()`

Com `record`, o compilador gera isso automaticamente com base nos componentes declarados.

---

## 2) Exemplo mais simples

```java
public record UserDto(Long id, String nome, String email) {
}
```

Ao declarar esse record, o Java já gera:
- construtor canônico: `new UserDto(id, nome, email)`
- acessores: `id()`, `nome()`, `email()`
- `equals()` e `hashCode()` por valor
- `toString()` legível

### Uso rápido

```java
UserDto user = new UserDto(1L, "Ana", "ana@email.com");

System.out.println(user.nome());
System.out.println(user);
```

---

## 3) Como o Record é estruturado

Um record tem:
- **nome**
- **componentes** (no cabeçalho)
- **corpo opcional** para regras adicionais

```java
public record Produto(String sku, String descricao, double preco) {
}
```

> Os componentes viram campos `private final` internamente.

---

## 4) Construtores em Records

### 4.1 Construtor canônico explícito

Você pode declarar explicitamente o construtor completo quando precisar de validações detalhadas:

```java
public record Cliente(String nome, String documento) {
    public Cliente(String nome, String documento) {
        if (nome == null || nome.isBlank()) {
            throw new IllegalArgumentException("Nome é obrigatório");
        }

        if (documento == null || documento.isBlank()) {
            throw new IllegalArgumentException("Documento é obrigatório");
        }

        this.nome = nome;
        this.documento = documento;
    }
}
```

### 4.2 Compact constructor (mais comum)

No compact constructor, você valida os parâmetros sem repetir assinatura e sem fazer atribuição manual (`this.campo = campo`):

```java
public record Cliente(String nome, String documento) {
    public Cliente {
        if (nome == null || nome.isBlank()) {
            throw new IllegalArgumentException("Nome é obrigatório");
        }

        if (documento == null || documento.isBlank()) {
            throw new IllegalArgumentException("Documento é obrigatório");
        }
    }
}
```

---

## 5) Comportamento adicional no Record

Record não é “só DTO”. Você pode ter:
- métodos de negócio simples
- métodos estáticos
- validações
- interfaces implementadas

```java
public record Money(java.math.BigDecimal valor, String moeda) {
    public Money {
        if (valor == null || valor.signum() < 0) {
            throw new IllegalArgumentException("Valor não pode ser negativo");
        }
        if (moeda == null || moeda.isBlank()) {
            throw new IllegalArgumentException("Moeda é obrigatória");
        }
    }

    public boolean isZero() {
        return valor.signum() == 0;
    }

    public static Money zero(String moeda) {
        return new Money(java.math.BigDecimal.ZERO, moeda);
    }
}
```

---

## 6) Limitações importantes

Use record sabendo das regras:

1. **Não pode estender outra classe**
   - Todo record já estende implicitamente `java.lang.Record`.

2. **É final**
   - Não pode ser herdado.

3. **Campos de instância extras não são permitidos**
   - Só os componentes declarados no cabeçalho.
   - Campos `static` são permitidos.

4. **Imutabilidade de referência, não necessariamente profunda**
   - Se um componente for mutável (ex.: `List`), o conteúdo pode mudar se não for protegido.

### Exemplo de cuidado com coleção mutável

```java
import java.util.List;

public record PedidoDto(Long id, List<String> itens) {
    public PedidoDto {
        itens = List.copyOf(itens); // cópia imutável defensiva
    }
}
```

---

## 7) Classe tradicional vs Record

### Classe tradicional

```java
public class Pessoa {
    private final String nome;
    private final int idade;

    public Pessoa(String nome, int idade) {
        this.nome = nome;
        this.idade = idade;
    }

    public String getNome() {
        return nome;
    }

    public int getIdade() {
        return idade;
    }

    @Override
    public boolean equals(Object o) {
        // implementação completa omitida
        return true;
    }

    @Override
    public int hashCode() {
        // implementação completa omitida
        return 1;
    }

    @Override
    public String toString() {
        return "Pessoa{nome='" + nome + "', idade=" + idade + "}";
    }
}
```

### Mesmo caso com Record

```java
public record Pessoa(String nome, int idade) {
}
```

### Resultado prático

- Menos código repetitivo
- Menor chance de erro em `equals/hashCode`
- Leitura mais rápida para o time

---

## 8) Record vs Lombok (`@Data` e `@Value`)

| Critério | Record | Lombok `@Data` | Lombok `@Value` |
| :--- | :--- | :--- | :--- |
| Nativo do Java | Sim | Não | Não |
| Boilerplate | Mínimo | Baixo | Baixo |
| Imutável por padrão | Sim | Não | Sim |
| Sem dependência extra | Sim | Não | Não |
| Sem processamento de anotação | Sim | Não | Não |

### Regra prática de escolha

- **Use Record**: quando modelar dados imutáveis e simples.
- **Use classe + Lombok**: quando precisar de mutabilidade controlada, herança específica, ou modelagem fora do perfil de record.

---

## 9) Exemplos simples de uso real

### 9.1 DTO de resposta de API

```java
public record UsuarioResponse(Long id, String nome, String email) {
}
```

### 9.2 DTO de entrada com validação no compact constructor

```java
public record CriarUsuarioRequest(String nome, String email) {
    public CriarUsuarioRequest {
        if (nome == null || nome.isBlank()) {
            throw new IllegalArgumentException("Nome é obrigatório");
        }
        if (email == null || !email.contains("@")) {
            throw new IllegalArgumentException("Email inválido");
        }
    }
}
```

### 9.3 Igualdade por valor (ótimo para coleção)

```java
import java.util.HashSet;
import java.util.Set;

public class Main {
    public static void main(String[] args) {
        Set<Ponto> pontos = new HashSet<>();
        pontos.add(new Ponto(10, 20));
        pontos.add(new Ponto(10, 20));

        // imprime 1, pois equals/hashCode são por valor
        System.out.println(pontos.size());
    }

    public record Ponto(int x, int y) {}
}
```

### 9.4 Serialização JSON (Jackson)

Com Java moderno + Jackson atualizado, records funcionam muito bem para request/response sem boilerplate.

```java
public record EnderecoResponse(String rua, String cidade, String cep) {
}
```

---

## 10) Quando usar e quando evitar

### Quando usar

- DTOs de API
- Objetos de valor (Value Objects)
- Retornos de consulta/projeção
- Eventos de domínio imutáveis

### Quando evitar

- Entidades JPA complexas com ciclo de vida e proxies
- Objetos com mutabilidade necessária
- Modelos que exigem herança de classe
- Casos com estado interno derivado e mutável

---

## 11) Boas práticas

1. **Mantenha o record pequeno e coeso**
2. **Valide invariantes no compact constructor**
3. **Faça cópia defensiva para coleções**
4. **Evite lógica de negócio complexa no record**
5. **Use record para modelar dado, não fluxo de aplicação**

---

## 12) Checklist rápido de decisão

Antes de criar uma classe, pergunte:

- [ ] Esse objeto representa dados imutáveis?
- [ ] Não preciso de herança de classe?
- [ ] Quero igualdade por valor automaticamente?
- [ ] O objetivo principal é transporte/representação de dados?

Se a maioria for “sim”, **record é forte candidato**.

---

## 13) Contexto para uso com IA (prompts e padronização)

Para acelerar produção com IA de forma consistente, use um prompt-base como referência:

```text
Crie um Java record para representar [entidade/DTO], com campos [x, y, z],
incluindo validações no compact constructor e exemplos de uso.
Evite boilerplate desnecessário e adote boas práticas de imutabilidade.
```

### Regras úteis para IA gerar records melhores

- Pedir validações explícitas
- Pedir exemplos de uso real
- Pedir comparação com classe tradicional quando houver dúvida
- Pedir cópia defensiva para listas/mapas

---

## 14) Como fica em Kotlin?

No Kotlin, o equivalente conceitual de `record` é a **data class**.

### Exemplo simples em Kotlin

```kotlin
data class UserDto(
    val id: Long,
    val nome: String,
    val email: String
)
```

Assim como em `record`, o Kotlin gera automaticamente:
- `equals()`
- `hashCode()`
- `toString()`
- cópia com `copy(...)`
- destruturação com `componentN()`

### Validação prática em Kotlin

```kotlin
data class CriarUsuarioRequest(
    val nome: String,
    val email: String
) {
    init {
        require(nome.isNotBlank()) { "Nome é obrigatório" }
        require(email.contains("@")) { "Email inválido" }
    }
}
```

### Comparação rápida: Java Record vs Kotlin Data Class

| Critério | Java `record` | Kotlin `data class` |
| :--- | :--- | :--- |
| Imutabilidade padrão | Sim (componentes finais) | Sim quando usa `val` |
| Geração de `equals/hashCode/toString` | Sim | Sim |
| Cópia de objeto | Não nativo (manual) | Sim (`copy`) |
| Desestruturação | Não nativo | Sim (`componentN`) |

### Quando usar em projetos mistos Java + Kotlin

- APIs em Java: prefira `record` para DTOs imutáveis.
- APIs em Kotlin: prefira `data class` para o mesmo propósito.
- Em ambos os casos, mantenha o foco em objetos pequenos, claros e sem mutabilidade acidental.

---

## 15) Resumo final

Records são uma evolução importante do Java para modelagem de dados imutáveis com menos código, mais clareza e menor chance de erro em implementações repetitivas.

No uso diário, eles funcionam muito bem para DTOs, respostas de API e objetos de valor. A adoção tende a melhorar legibilidade, manutenção e produtividade do time.

---

## Referências recomendadas

- Documentação oficial Java: JEP 395 (Records)
- Java Language Specification (records)
- Documentação Jackson para suporte a records
