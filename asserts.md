
# Documentação: Uso de @AssertTrue e @AssertFalse no Spring Boot com Kotlin

## 1. Introdução

O Spring Boot, em conjunto com a **API de Validação Bean (JSR-380)**, oferece annotations declarativas para validar dados de entrada de forma simples e maintainable. Dentre elas, **@AssertTrue** e **@AssertFalse** se destacam como ferramentas poderosas para implementar **regras de validação customizadas e cross-field** (validação entre campos), encapsulando lógica complexa em métodos booleanos.

## 2. Pré-requisitos

### Dependências Necessárias
Adicione a dependência `spring-boot-starter-validation` ao seu `pom.xml` (para projetos Maven):
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

Para Gradle:
```groovy
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

## 3. Conceitos Fundamentais

### 3.1 O que são @AssertTrue e @AssertFalse?
Essas annotations marcam **métodos ou propriedades booleanas** que representam regras de validação. A validação é acionada automaticamente em dois cenários principais:
1. Quando um DTO é recebido em um endpoint com a annotation `@Valid`
2. Quando um objeto é validado manualmente via a interface `javax.validation.Validator`

### 3.2 Parâmetros Principais

| Parâmetro | Descrição |
|-----------|-----------|
| `message` | Mensagem de erro customizada (suporta internacionalização com chaves de properties, ex: `{validacao.termos}`) |
| `groups` | Define grupos de validação para regras condicionais |
| `payload` | Meta-dados adicionais (usado para classes customizadas de validação) |

## 4. Exemplos Práticos em Kotlin

### 4.1 Validação Básica (Termos de Serviço)
Valida se o usuário aceitou os termos de serviço:
```kotlin
import javax.validation.constraints.AssertTrue

data class UsuarioCreateDto(
    val nome: String,
    val email: String,
    @AssertTrue(message = "Você deve aceitar os termos de serviço para continuar")
    val aceitaTermos: Boolean
)
```

### 4.2 Validação Cross-Field (Senha e Confirmação)
Valida se a senha e a confirmação de senha coincidem:
```kotlin
import javax.validation.constraints.AssertTrue
import javax.validation.constraints.Size

data class UsuarioCreateDto(
    val nome: String,
    @Size(min = 8, message = "A senha deve ter pelo menos 8 caracteres")
    val senha: String,
    val confirmarSenha: String
) {
    @AssertTrue(message = "A senha e a confirmação de senha devem ser iguais")
    fun isSenhasCoincidem(): Boolean {
        return senha == confirmarSenha
    }
}
```

### 4.3 Validação de Data (Data de Fim > Data de Início)
Valida se a data de término de um evento é posterior à data de início:
```kotlin
import javax.validation.constraints.AssertTrue
import javax.validation.constraints.Future
import java.time.LocalDate

data class EventoCreateDto(
    val nome: String,
    @Future(message = "A data de início deve ser no futuro")
    val dataInicio: LocalDate,
    val dataFim: LocalDate
) {
    @AssertTrue(message = "A data de término deve ser posterior à data de início")
    fun isDataFimValida(): Boolean {
        return dataFim.isAfter(dataInicio)
    }
}
```

### 4.4 Integração com Endpoints
Use `@Valid` para acionar a validação automaticamente em controllers:
```kotlin
import org.springframework.validation.annotation.Validated
import org.springframework.web.bind.annotation.PostMapping
import org.springframework.web.bind.annotation.RequestBody
import org.springframework.web.bind.annotation.RestController

@RestController
@Validated
class UsuarioController {

    @PostMapping("/usuarios")
    fun criarUsuario(@Valid @RequestBody dto: UsuarioCreateDto) {
        // Lógica de criação
    }
}
```

## 5. Validação Manual
Para validar objetos fora de endpoints, use o `Validator`:
```kotlin
import javax.validation.Validation
import javax.validation.Validator

fun validarDto(dto: UsuarioCreateDto): List<String> {
    val validator: Validator = Validation.buildDefaultValidatorFactory().validator
    val violations = validator.validate(dto)
    return violations.map { it.message }
}
```

## 6. Casos de Uso Comuns

### 6.1 Aceitação de Termos
- Validar aceitação de termos de serviço ou políticas de privacidade

### 6.2 Combinação de Campos
- Validação de senha e confirmação de senha
- Validação de email e confirmação de email

### 6.3 Idade Mínima
- Verificar se um usuário tem pelo menos 18 anos (baseado na data de nascimento)

### 6.4 Datas Válidas
- Validar que a data de término é posterior à data de início
- Validar que uma data não é passada

### 6.5 Condições Complexas
- Validar que um campo é obrigatório apenas se outro campo tiver um valor específico
- Validar que um usuário tem permissão para realizar uma ação

## 7. Melhores Práticas

### 7.1 Mantenha a Lógica Simples
Métodos de validação devem ser **pequenos e focados em uma única regra**. Evite lógica complexa ou de negócio direto na validação.

### 7.2 Evite Side Effects
Não altere o estado do objeto durante a validação — a função deve ser **pura** (retornar sempre o mesmo resultado para os mesmos parâmetros).

### 7.3 Use Mensagens Claros
Mensagens de erro devem **explicar o problema e como resolvê-lo**. Evite termos técnicos desnecessários para o usuário final.

### 7.4 Teste a Validação
Escreva testes unitários para validar suas regras de validação. Exemplo com `spring-boot-starter-test`:
```kotlin
import org.junit.jupiter.api.Assertions
import org.junit.jupiter.api.Test
import javax.validation.Validation

class UsuarioCreateDtoTest {

    @Test
    fun `deve retornar erro quando senhas não coincidem`() {
        val dto = UsuarioCreateDto(
            nome = "João",
            senha = "12345678",
            confirmarSenha = "87654321"
        )

        val violations = Validation.buildDefaultValidatorFactory()
            .validator
            .validate(dto)

        Assertions.assertTrue(violations.isNotEmpty())
        Assertions.assertEquals(1, violations.size)
        Assertions.assertEquals("A senha e a confirmação de senha devem ser iguais", violations.first().message)
    }
}
```

### 7.5 Priorize Declaratividade
Use annotations nativas (ex: `@Min`, `@Email`, `@Future`) sempre que possível, reservando `@AssertTrue` e `@AssertFalse` para lógicas customizadas.

## 8. Erros Comuns e Soluções

### 8.1 Métodos Não Públicos
A validação só funciona com métodos `public` ou `protected`. Erro comum:
```kotlin
// Não funciona! Método é private
private fun isSenhasCoincidem(): Boolean {
    return senha == confirmarSenha
}
```

### 8.2 Propriedades Nulláveis
Verifique se o valor é `null` antes de acessar propriedades:
```kotlin
// Correto: Verifica dataNascimento != null
@AssertTrue(message = "O usuário deve ter pelo menos 18 anos")
fun isMaiorDeIdade(): Boolean {
    return dataNascimento?.let { LocalDate.now().minusYears(18).isAfter(it) } ?: false
}
```

### 8.3 Validação Não Acionada
Certifique-se de usar `@Valid` no controller e `@Validated` na classe do controller:
```kotlin
// Correto: Controller com @Validated
@RestController
@Validated
class UsuarioController {

    @PostMapping("/usuarios")
    fun criarUsuario(@Valid @RequestBody dto: UsuarioCreateDto) {
        // Lógica de criação
    }
}
```

### 8.4 Performance
Evite lógicas complexas na validação, como chamadas a APIs externas ou consultas ao banco de dados — essas operações devem ficar na camada de serviço.

## 9. Conclusão

@AssertTrue e @AssertFalse são ferramentas essenciais para implementar regras de validação customizadas em APIs Kotlin com Spring Boot. Elas encapsulam lógica complexa em métodos booleanos, mantendo o código limpo e declarativo, e garantem que os dados de entrada sejam válidos antes de serem processados.

## 10. Recursos Adicionais
- [Documentação Oficial do Spring Boot sobre Validação](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.validation)
- [JSR-380 (Bean Validation 2.0) Specification](https://jakarta.ee/specifications/bean-validation/2.0/)
- [Baeldung: Bean Validation with Spring Boot](https://www.baeldung.com/spring-boot-bean-validation)
