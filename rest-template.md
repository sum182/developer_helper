# Rest Template com Kotlin

## Introdução

`RestTemplate` é um cliente HTTP síncrono da stack Spring para consumo de APIs REST. Em projetos Kotlin com Spring Boot, ele ainda aparece com frequência em sistemas legados e integrações internas.

Ele resolve o problema de comunicação HTTP com APIs externas de forma simples, com suporte a:
- métodos HTTP (`GET`, `POST`, `PUT`, `DELETE`)
- serialização/desserialização automática com Jackson
- configuração de timeouts e interceptors
- tratamento de erros via `ResponseErrorHandler`

Quando faz sentido estudar/aplicar:
- manutenção de sistemas existentes que já usam `RestTemplate`
- integrações síncronas simples
- times que ainda não migraram para `WebClient`

---

## Sintaxe básica

Conceitos essenciais:
- criar um `RestTemplate` (geralmente via `@Bean`)
- chamar métodos como `getForObject`, `postForEntity` e `exchange`
- mapear resposta para data classes Kotlin

Exemplo mínimo funcional:

```kotlin
import org.springframework.boot.web.client.RestTemplateBuilder
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration
import org.springframework.web.client.RestTemplate
import java.time.Duration

@Configuration
class RestTemplateConfig {

    @Bean
    fun restTemplate(builder: RestTemplateBuilder): RestTemplate {
        return builder
            .setConnectTimeout(Duration.ofSeconds(3))
            .setReadTimeout(Duration.ofSeconds(5))
            .build()
    }
}
```

Uso simples:

```kotlin
import org.springframework.stereotype.Service
import org.springframework.web.client.getForObject

data class UserResponse(
    val id: Long,
    val name: String,
    val email: String
)

@Service
class UserApiClient(private val restTemplate: org.springframework.web.client.RestTemplate) {

    fun buscarUsuario(id: Long): UserResponse? {
        return restTemplate.getForObject("https://api.exemplo.com/users/$id")
    }
}
```

---

## Exemplos simples e práticos

### 1) GET com `getForEntity`

```kotlin
import org.springframework.http.ResponseEntity

fun buscarStatus(): ResponseEntity<String> {
    return restTemplate.getForEntity("https://api.exemplo.com/status", String::class.java)
}
```

### 2) POST com body

```kotlin
data class CreateOrderRequest(
    val productId: Long,
    val quantity: Int
)

data class CreateOrderResponse(
    val orderId: String,
    val status: String
)

fun criarPedido(): CreateOrderResponse? {
    val request = CreateOrderRequest(productId = 10, quantity = 2)

    return restTemplate.postForObject(
        "https://api.exemplo.com/orders",
        request,
        CreateOrderResponse::class.java
    )
}
```

### 3) Headers + método `exchange`

```kotlin
import org.springframework.http.HttpEntity
import org.springframework.http.HttpHeaders
import org.springframework.http.HttpMethod

fun buscarComHeader(): String? {
    val headers = HttpHeaders().apply {
        set("X-Correlation-Id", "abc-123")
    }

    val entity = HttpEntity<Void>(headers)

    val response = restTemplate.exchange(
        "https://api.exemplo.com/resource",
        HttpMethod.GET,
        entity,
        String::class.java
    )

    return response.body
}
```

### 4) PUT para atualização

```kotlin
data class UpdateProfileRequest(
    val name: String
)

fun atualizarPerfil(userId: Long) {
    val request = UpdateProfileRequest(name = "Novo Nome")
    restTemplate.put("https://api.exemplo.com/users/$userId", request)
}
```

### 5) DELETE

```kotlin
fun removerUsuario(userId: Long) {
    restTemplate.delete("https://api.exemplo.com/users/$userId")
}
```

---

## Boas práticas

- Centralize a criação do cliente em um único `@Bean`.
- Defina timeouts de conexão e leitura explicitamente.
- Evite espalhar URLs fixas no código; use `application.yml` e classes de configuração.
- Padronize tratamento de erro para respostas 4xx/5xx.
- Inclua logs estruturados com contexto (endpoint, método, duração, correlation id).
- Use DTOs específicos por operação (request/response) para manter clareza.
- Crie uma camada de client (`FooApiClient`) para não misturar integração externa com regra de negócio.

---

## Limitações e armadilhas

### Limitações técnicas

- `RestTemplate` é bloqueante (thread fica ocupada até resposta).
- Não é a opção mais moderna para alto volume de I/O concorrente.
- Evolução da stack Spring prioriza `WebClient` para novos cenários.

### Armadilhas comuns

- **Sem timeout**: risco de requisições presas.
- **Tratamento de erro genérico**: dificulta observabilidade e troubleshooting.
- **Uso direto em várias classes**: gera duplicação de lógica HTTP.
- **Mapeamento frágil de JSON**: pode quebrar com mudanças de contrato.

Como evitar:
- configurar timeout e retry (quando fizer sentido)
- encapsular chamadas em clients dedicados
- validar e versionar contratos de API
- monitorar latência, erro e taxa de timeout

---

## Quando usar e quando evitar

### Quando usar

- projeto legado já baseado em `RestTemplate`
- baixa concorrência de chamadas externas
- integração simples, síncrona e direta
- equipe já familiarizada com abordagem imperativa

### Quando evitar

- novas aplicações com alta necessidade de escala de I/O
- cenários reativos (Spring WebFlux)
- necessidades avançadas de composição assíncrona
- casos com alto throughput e baixa latência sob carga

---

## Alternativas modernas e atuais

### 1) `WebClient` (Spring)

- cliente HTTP recomendado para novos projetos Spring
- suporta modo reativo e também uso bloqueante controlado
- melhor integração com observabilidade moderna

### 2) OpenFeign (Spring Cloud)

- interface declarativa para APIs HTTP
- reduz boilerplate em integrações entre serviços
- boa escolha para ecossistemas de microserviços Spring Cloud

### Comparativo rápido

| Critério | RestTemplate | WebClient | OpenFeign |
|---|---|---|---|
| Modelo | Síncrono bloqueante | Reativo/não bloqueante | Declarativo (geralmente bloqueante) |
| Curva de aprendizado | Baixa | Média | Baixa/Média |
| Indicado para novos projetos | Não recomendado | Sim | Depende do ecossistema |
| Boilerplate | Médio | Médio | Baixo |

---

## Comparativo entre RestClient, WebClient e RestTemplate

Com Spring 6+, o `RestClient` surgiu como alternativa moderna no estilo imperativo, enquanto `WebClient` segue como principal escolha para cenários reativos e alta concorrência.

### Visão geral

- **RestTemplate**: API clássica, estável e muito usada em legado.
- **RestClient**: API mais nova, fluente e alinhada às versões atuais do Spring, mantendo modelo síncrono.
- **WebClient**: API reativa e não bloqueante, ideal para alto volume de chamadas externas.

### Tabela de decisão rápida

| Critério | RestTemplate | RestClient | WebClient |
|---|---|---|---|
| Estilo de API | Tradicional | Fluente moderna | Fluente reativa |
| Modelo de execução | Bloqueante | Bloqueante | Não bloqueante |
| Melhor uso | Legado e manutenção | Novos projetos síncronos | Cenários reativos e alto throughput |
| Curva de aprendizado | Baixa | Baixa | Média/Alta |
| Suporte futuro no ecossistema | Manutenção | Recomendado para imperativo moderno | Recomendado para reativo |

### Exemplo rápido de cada abordagem (Kotlin)

```kotlin
// RestTemplate
val usuarioRt = restTemplate.getForObject(
    "https://api.exemplo.com/users/{id}",
    UserResponse::class.java,
    1
)

// RestClient (Spring 6+)
val usuarioRc = restClient.get()
    .uri("https://api.exemplo.com/users/{id}", 1)
    .retrieve()
    .body(UserResponse::class.java)

// WebClient
val usuarioWc = webClient.get()
    .uri("https://api.exemplo.com/users/{id}", 1)
    .retrieve()
    .bodyToMono(UserResponse::class.java)
    .block() // uso bloqueante opcional; em fluxo reativo, evite block()
```

### Recomendação prática

- Se você já está em legado: mantenha `RestTemplate` com boas práticas.
- Se vai iniciar integração síncrona nova em Spring 6+: prefira `RestClient`.
- Se precisa escalar chamadas I/O com eficiência: use `WebClient`.

---

## Resumo final

`RestTemplate` continua relevante para manutenção de legado e integrações síncronas simples em Kotlin com Spring Boot. Para uso sustentável:
- configure timeouts e tratamento de erros
- encapsule chamadas em clients dedicados
- evite acoplamento de regras de negócio com HTTP

Para novos projetos, a decisão padrão tende a favorecer `WebClient` (ou `OpenFeign` em cenários declarativos), mantendo `RestTemplate` como opção de transição e compatibilidade.
