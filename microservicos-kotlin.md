# Microserviços — Guia de Uso (Kotlin 2.x + Spring Boot 3.x + Java 21)

Guia prático para construir, comunicar e operar um microserviço em Kotlin com Spring Boot. Foco no que muda de verdade no dia a dia, não em teoria de arquitetura.

> Este guia trata do serviço em si. O ferramental de plataforma que costuma acompanhar (configuração centralizada, service discovery, gateway, OpenFeign) está em `spring-cloud.md`, no mesmo diretório. A seção 9 lista todos os guias relacionados.

---

## 1. Introdução

### Contexto

Microserviço é um serviço pequeno, com banco próprio, deploy próprio e um dono claro. Ele expõe uma fronteira (HTTP, fila, evento) e esconde tudo o resto.

Kotlin entra bem nesse cenário por três motivos concretos.

- Null safety no compilador, o que corta uma classe inteira de `NullPointerException` em payload de integração.
- `data class` e `sealed class`, que deixam DTO e resultado de operação curtos e explícitos.
- Coroutines, que dão concorrência barata quando o serviço passa a maior parte do tempo esperando I/O de outro serviço.

### Problema que resolve

Um monolito grande tem três dores que o microserviço ataca.

- Deploy acoplado, onde uma mudança pequena obriga a subir tudo.
- Escala acoplada, onde um endpoint pesado obriga a escalar a aplicação inteira.
- Time acoplado, onde várias equipes disputam o mesmo código e a mesma janela de release.

### Quando faz sentido aplicar

Faz sentido quando pelo menos duas destas condições são verdadeiras.

- Times diferentes precisam subir código em ritmos diferentes.
- Partes do sistema têm perfil de carga muito diferente (OCR de fatura versus cadastro de cliente).
- O domínio já tem fronteira clara e estável, com linguagem própria e dono definido.
- Falha em uma parte não pode derrubar as outras.

Não faz sentido quando o domínio ainda está sendo descoberto, quando há um time só, ou quando não existe infraestrutura de deploy, log centralizado e trace distribuído. Sem isso, você troca um problema de código por um problema de operação.

---

## 2. Sintaxe Básica

### Dependências mínimas (Gradle Kotlin DSL)

```kotlin
plugins {
    kotlin("jvm") version "2.1.0"
    kotlin("plugin.spring") version "2.1.0"
    kotlin("plugin.jpa") version "2.1.0"
    id("org.springframework.boot") version "3.4.1"
    id("io.spring.dependency-management") version "1.1.7"
}

kotlin {
    jvmToolchain(21)
    compilerOptions {
        freeCompilerArgs.addAll("-Xjsr305=strict")
    }
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-validation")
    implementation("org.springframework.boot:spring-boot-starter-actuator")
    implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}
```

Três pontos importantes.

- `kotlin("plugin.spring")` abre as classes automaticamente. Sem ele, `@Service` e `@Configuration` quebram, porque classe em Kotlin é `final` por padrão e o Spring precisa criar proxy.
- `kotlin("plugin.jpa")` gera o construtor sem argumentos que o Hibernate exige nas entidades.
- `-Xjsr305=strict` faz o Kotlin respeitar as anotações de nulidade do Spring, o que evita passar `null` onde a API Java não aceita.

### Exemplo mínimo funcional

```kotlin
@SpringBootApplication
class ClienteServiceApplication

fun main(args: Array<String>) {
    runApplication<ClienteServiceApplication>(*args)
}

data class ClienteResponse(
    val id: Long,
    val nome: String,
    val documento: String,
)

@RestController
@RequestMapping("/api/v1/clientes")
class ClienteController(private val clienteService: ClienteService) {

    @GetMapping("/{id}")
    fun buscarPorId(@PathVariable id: Long): ClienteResponse =
        clienteService.buscarPorId(id)
}
```

Repare que a injeção é por construtor primário, sem `@Autowired` e sem campo mutável. Esse é o padrão em Kotlin e deve ser o único usado.

---

## 3. Exemplos Simples e Práticos

### 3.1. DTO de entrada com validação

```kotlin
data class CriarClienteRequest(
    @field:NotBlank(message = "Nome é obrigatório")
    val nome: String,

    @field:CPF(message = "CPF inválido")
    val documento: String,

    @field:Email(message = "E-mail inválido")
    val email: String,
)
```

O prefixo `@field:` é obrigatório. Sem ele a anotação vai para o parâmetro do construtor e o Bean Validation simplesmente ignora a regra, sem erro nenhum. É a armadilha número um de quem vem de Java.

Para validação que envolve mais de um campo (data final maior que data inicial, campo obrigatório só quando o tipo é X), use `@AssertTrue` e `@AssertFalse`. O detalhamento está em `asserts.md`, no mesmo diretório.

### 3.2. Resultado de operação com `sealed class`

Em vez de devolver `null` ou lançar exceção para tudo, modele o resultado.

```kotlin
sealed class ResultadoCobranca {
    data class Aprovada(val transacaoId: String) : ResultadoCobranca()
    data class Recusada(val motivo: String) : ResultadoCobranca()
    data object ProvedorIndisponivel : ResultadoCobranca()
}

fun tratar(resultado: ResultadoCobranca) = when (resultado) {
    is ResultadoCobranca.Aprovada -> log.info("Cobrança aprovada. Transação {}", resultado.transacaoId)
    is ResultadoCobranca.Recusada -> log.warn("Cobrança recusada. Motivo {}", resultado.motivo)
    ResultadoCobranca.ProvedorIndisponivel -> log.error("Provedor de cobrança indisponível")
}
```

O `when` sobre `sealed` é exaustivo. Se alguém adicionar um novo caso, o compilador quebra o build em todos os pontos que precisam tratar. Isso é o oposto de um `if/else` que silenciosamente cai no ramo errado.

### 3.3. Chamada HTTP para outro serviço com `RestClient`

```kotlin
@Configuration
class HttpClientConfig {

    @Bean
    fun contratoRestClient(
        builder: RestClient.Builder,
        @Value("\${servicos.contrato.url}") baseUrl: String,
    ): RestClient = builder
        .baseUrl(baseUrl)
        .requestFactory(
            SimpleClientHttpRequestFactory().apply {
                setConnectTimeout(Duration.ofSeconds(2))
                setReadTimeout(Duration.ofSeconds(5))
            }
        )
        .build()
}

@Component
class ContratoClient(private val contratoRestClient: RestClient) {

    fun buscarPorCliente(clienteId: Long): List<ContratoResponse> =
        contratoRestClient.get()
            .uri("/api/v1/contratos?clienteId={id}", clienteId)
            .retrieve()
            .body(object : ParameterizedTypeReference<List<ContratoResponse>>() {})
            ?: emptyList()
}
```

Timeout explícito não é detalhe. Cliente HTTP sem timeout é a causa mais comum de efeito cascata, onde um serviço lento trava o pool de threads de quem chama e derruba a cadeia inteira.

Interceptors, tratamento de erro e comparação com `WebClient` estão em `rest-client.md`. Serviço legado que ainda usa `RestTemplate` tem o guia próprio em `rest-template.md`. Se o ambiente usa Eureka e balanceamento por nome de serviço, o `OpenFeign` cobre o mesmo papel de forma declarativa, conforme `spring-cloud.md`. Os três no mesmo diretório.

### 3.4. Resiliência com Resilience4j

```kotlin
@Component
class ContratoClient(private val contratoRestClient: RestClient) {

    @CircuitBreaker(name = "contrato-service", fallbackMethod = "buscarPorClienteFallback")
    @Retry(name = "contrato-service")
    fun buscarPorCliente(clienteId: Long): List<ContratoResponse> =
        contratoRestClient.get()
            .uri("/api/v1/contratos?clienteId={id}", clienteId)
            .retrieve()
            .body(object : ParameterizedTypeReference<List<ContratoResponse>>() {})
            ?: emptyList()

    private fun buscarPorClienteFallback(clienteId: Long, ex: Exception): List<ContratoResponse> {
        log.warn("Falha ao consultar contratos do cliente {}. Retornando lista vazia", clienteId, ex)
        return emptyList()
    }
}
```

```yaml
resilience4j:
  circuitbreaker:
    instances:
      contrato-service:
        slidingWindowSize: 20
        failureRateThreshold: 50
        waitDurationInOpenState: 30s
  retry:
    instances:
      contrato-service:
        maxAttempts: 3
        waitDuration: 200ms
        enableExponentialBackoff: true
```

Duas regras aqui.

- O método de fallback precisa ter a mesma assinatura mais um parâmetro de exceção, senão o Resilience4j não encontra e lança erro em runtime.
- Só use retry em operação idempotente. Retry em `POST` de cobrança gera cobrança duplicada.

O Resilience4j também aparece integrado ao restante do ecossistema distribuído, junto com Config Server, Eureka e Gateway, em `spring-cloud.md`, no mesmo diretório.

### 3.5. Consumo de evento assíncrono

```kotlin
@Component
class PropostaAprovadaListener(private val comissaoService: ComissaoService) {

    @RabbitListener(queues = ["\${filas.proposta-aprovada}"])
    fun consumir(evento: PropostaAprovadaEvento) {
        log.info("Recebido evento de proposta aprovada. Proposta {}", evento.propostaId)

        if (comissaoService.jaProcessada(evento.eventoId)) {
            log.info("Evento {} já processado. Ignorando", evento.eventoId)
            return
        }

        comissaoService.calcular(evento)
    }
}
```

A checagem de `jaProcessada` é obrigatória, não opcional. Broker entrega pelo menos uma vez, então o mesmo evento vai chegar duas vezes em algum momento. Quem garante o "exatamente uma vez" é o consumidor, através de uma chave de idempotência gravada em banco.

Três guias complementam esta parte, todos no mesmo diretório.

- `event-driven-architecture.md` traz os conceitos de evento, produtor, consumidor e broker, além da diferença entre evento e comando.
- `rabbitmq_resiliencia.md` detalha o lado do produtor, com publisher confirms, Outbox Pattern e DLQ, que é exatamente o que garante que a mensagem não se perca entre o commit do banco e o envio.
- `application_events.md` trata dos eventos internos do Spring, que rodam dentro do mesmo processo. Não confunda os dois, porque `ApplicationEvent` desacopla componentes dentro do serviço, enquanto o evento de broker desacopla serviços diferentes.

### 3.6. Tratamento de erro centralizado

```kotlin
data class ErroResponse(
    val codigo: String,
    val mensagem: String,
    val campos: Map<String, String> = emptyMap(),
)

@RestControllerAdvice
class TratadorGlobalDeErros {

    @ExceptionHandler(RecursoNaoEncontradoException::class)
    fun tratarNaoEncontrado(ex: RecursoNaoEncontradoException) =
        ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(ErroResponse("RECURSO_NAO_ENCONTRADO", ex.message.orEmpty()))

    @ExceptionHandler(MethodArgumentNotValidException::class)
    fun tratarValidacao(ex: MethodArgumentNotValidException): ResponseEntity<ErroResponse> {
        val campos = ex.bindingResult.fieldErrors
            .associate { it.field to (it.defaultMessage ?: "Valor inválido") }

        return ResponseEntity.badRequest()
            .body(ErroResponse("DADOS_INVALIDOS", "Dados de entrada inválidos", campos))
    }
}
```

Contrato de erro estável é parte da API pública do serviço. Quem consome precisa conseguir tratar por código, não por texto da mensagem.

---

## 4. Boas Práticas

### Fronteira do serviço

- Um banco por serviço. Nada de dois serviços escrevendo na mesma tabela, porque isso recria o acoplamento do monolito com a latência da rede junto.
- Versione a API no path (`/api/v1/...`). Mudança incompatível vira `v2`, com a `v1` viva até os consumidores migrarem.
- Publique evento com os dados que o consumidor precisa, e não só o `id`. Evento magro força todo consumidor a fazer uma chamada de volta, o que multiplica carga e cria dependência circular.

Banco próprio também significa escolher o banco certo para aquele domínio, já que a decisão deixa de ser global. Os critérios de relacional versus documento estão em `sql_vs_nosql.md`, no mesmo diretório.

### Código

- Injeção por construtor primário, sempre. Sem `lateinit var` em bean.
- Controller fino, apenas recebendo, validando e delegando. Regra de negócio mora no service.
- `val` por padrão. Use `var` só quando o estado realmente muda.
- Rotina de apoio complexa vai para `private fun` dentro do próprio service, mantendo o método público como orquestrador legível.
- Use `require` e `check` para pré-condição, com mensagem em português.

```kotlin
fun aprovar(proposta: Proposta) {
    require(proposta.status == StatusProposta.EM_ANALISE) {
        "Proposta ${proposta.id} não está em análise"
    }
    // fluxo principal continua linear
}
```

Sobre a organização interna, o padrão em camadas (`Controller`, `Service`, `Repository`) atende bem a maior parte dos serviços. Quando o núcleo de negócio é complexo e precisa ficar isolado de banco e de integração externa, vale considerar Ports and Adapters, detalhado em `hexagonal.md`, no mesmo diretório. Microserviço e arquitetura hexagonal não competem, porque um resolve a fronteira externa e o outro resolve a organização interna.

Para listagem com filtro dinâmico, paginação e ordenação, use `Specification` com `JpaSpecificationExecutor`, no padrão descrito em `spring-data.md`. Endpoint de listagem sem paginação é o caminho mais rápido para estourar a memória do pod.

Cache reduz chamada entre serviços e é uma das defesas mais baratas contra latência em cadeia. As estratégias de cache local e distribuído estão em `cache.md`. Ambos no mesmo diretório.

### Configuração

- Configuração tipada com `@ConfigurationProperties`, não com `@Value` espalhado.

```kotlin
@ConfigurationProperties(prefix = "servicos.contrato")
data class ContratoProperties(
    val url: String,
    val timeoutSegundos: Long = 5,
)
```

- Segredo vem de variável de ambiente ou de secret manager, nunca do `application.yml` versionado.

Quando o número de serviços cresce, manter configuração duplicada em cada repositório vira custo. O Spring Cloud Config centraliza isso em um repositório único, com refresh sem redeploy. Veja `spring-cloud.md`, no mesmo diretório.

### Observabilidade

Três coisas são obrigatórias em qualquer microserviço que vai para produção.

- Log estruturado em JSON, com `correlationId` propagado entre serviços via header.
- Trace distribuído (Micrometer Tracing com OpenTelemetry), para conseguir seguir uma requisição que atravessa quatro serviços.
- Health check separando `liveness` e `readiness`, porque o Kubernetes trata os dois de forma diferente.

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,prometheus
  endpoint:
    health:
      probes:
        enabled: true
      show-details: when_authorized
  tracing:
    sampling:
      probability: 1.0
```

Use log com placeholder, não com interpolação de string.

```kotlin
log.info("Contrato {} assinado pelo cliente {}", contratoId, clienteId)
```

Interpolação (`"Contrato $contratoId assinado"`) monta a string mesmo quando o nível está desligado e destrói a possibilidade de agrupar logs por mensagem no Datadog.

Como montar trace distribuído, dashboard e alerta em cima disso está em `datadog_observabilidade.md`, no mesmo diretório. Em microserviço, trace não é luxo, porque sem ele a investigação de um erro vira leitura manual de log em quatro serviços diferentes.

### Testes

- Muitos testes de unidade no service, com Mockito ou MockK, no padrão Arrange, Act, Assert.
- Alguns testes de integração com Testcontainers, subindo banco e broker reais.
- Contract testing entre serviços que se chamam, para pegar quebra de contrato antes do deploy.

```kotlin
@Test
fun `deve recusar aprovacao quando proposta nao esta em analise`() {
    // Arrange
    val proposta = Proposta(id = 1L, status = StatusProposta.APROVADA)
    whenever(propostaRepository.findById(1L)).thenReturn(Optional.of(proposta))

    // Act
    val excecao = assertThrows<IllegalArgumentException> { propostaService.aprovar(1L) }

    // Assert
    assertThat(excecao.message).contains("não está em análise")
}
```

Nome de teste em crase com frase descritiva é idiomático em Kotlin e vale muito mais que `testAprovar2`.

Os comandos de execução por cenário (teste rápido durante o desenvolvimento, build completo antes do commit, execução de uma classe isolada) estão em `maven-test.md`, no mesmo diretório.

---

## 5. Limitações e Armadilhas

### Armadilhas de Kotlin com Spring

| Armadilha | O que acontece | Como evitar |
|---|---|---|
| `@field:` esquecido no DTO | Validação nunca roda, e dado inválido entra | Sempre `@field:NotBlank`, `@field:Email` |
| Classe `final` em `@Service` | Proxy de `@Transactional` e `@Cacheable` não é criado | Usar `kotlin-spring` plugin |
| `data class` como entidade JPA | `equals`/`hashCode` usam todos os campos, incluindo o `id` gerado, e quebram `Set` e cache do Hibernate | Entidade é classe normal, com `equals` por `id` |
| Tipo não nulo em campo carregado lazy | `NullPointerException` disfarçada na inicialização do Hibernate | Campo de entidade opcional é nulável (`var contrato: Contrato? = null`) |
| `runBlocking` dentro de controller | Trava a thread e anula o ganho da coroutine | Ou o fluxo é suspenso de ponta a ponta, ou não usa coroutine |

A `data class` de Kotlin ocupa o mesmo lugar do `record` de Java, e as duas têm a mesma restrição de não servirem como entidade JPA. Se o serviço é em Java ou se o time transita entre as duas linguagens, o comparativo está em `records.md`, no mesmo diretório.

### Armadilhas de arquitetura

- **Monolito distribuído.** Serviços que só funcionam se todos estiverem no ar, na versão certa, na ordem certa. É pior que monolito, porque soma o acoplamento do monolito com a latência e a falha parcial da rede.
- **Transação distribuída.** Não existe `@Transactional` entre dois serviços. O que existe é saga, com passo compensatório explícito, e isso é caro de escrever e de testar.
- **Consistência eventual invisível.** O usuário salva e não vê o resultado na hora. Se o produto não aceita isso, a fronteira do serviço está no lugar errado.
- **Chamada síncrona em cadeia.** Quatro serviços em série com 200 ms cada resultam em 800 ms de latência e quatro pontos de falha multiplicados. Prefira evento assíncrono ou composição em paralelo.
- **Banco compartilhado.** Duas equipes escrevendo na mesma tabela significa que nenhuma das duas consegue mudar o schema sozinha.

### Custo operacional real

Antes de quebrar um serviço em dois, saiba que você vai precisar de log centralizado, trace distribuído, pipeline de deploy por serviço, versionamento de contrato, ambiente local que suba as dependências, e alguém de plantão que entenda o mapa. Esse custo é fixo por serviço, não por linha de código.

O item do ambiente local costuma ser o mais subestimado. Assim que o desenvolvedor precisa de três serviços no ar para testar uma tela, `docker compose` deixa de ser opcional. O passo a passo de containerizar a aplicação Spring Boot e subir junto com as dependências está em `docker-containerizacao-local.md`, no mesmo diretório.

---

## 6. Quando Usar e Quando Evitar

### Use microserviço quando

- Times independentes precisam de ciclo de release independente.
- O perfil de escala é muito diferente entre partes do sistema.
- A fronteira do domínio já é clara, estável e tem dono.
- Requisitos de isolamento de falha ou de compliance exigem separação.
- Partes do sistema pedem tecnologia diferente por motivo técnico real.

### Evite microserviço quando

- O domínio ainda está mudando de forma rápida, porque mover fronteira entre serviços é muito mais caro que mover pacote dentro do monolito.
- Há um time só, com menos de oito pessoas.
- Não existe automação de deploy nem observabilidade centralizada.
- A regra de negócio exige consistência forte entre as duas partes.
- A motivação é "porque é moderno". Isso não é critério técnico.

### Caminho recomendado

Comece com um monolito modular, com módulos bem separados e sem dependência cruzada. Quando um módulo mostrar necessidade real de deploy ou escala independente, extraia só ele. A fronteira já vai estar desenhada e o custo de extração será baixo.

Um caso específico merece atenção. Processamento em massa (arquivo de retorno, geração de fatura em lote, conciliação) costuma ser confundido com necessidade de microserviço, quando na verdade é necessidade de job. Antes de criar um serviço novo, avalie um `Job` ou `CronJob` com Spring Batch, que já resolve reinicialização, transação por chunk e estatística de processamento. O detalhamento está em `spring_batch.md`, no mesmo diretório.

---

## 7. Alternativas Modernas

### Runtime e framework

| Opção | Ponto forte | Ponto fraco |
|---|---|---|
| Spring Boot 3.x (MVC) | Ecossistema maior, time já conhece, integração pronta com quase tudo | Startup mais lento, footprint de memória maior |
| Spring Boot com WebFlux | Alta concorrência de I/O com poucas threads | Stack reativa contamina todo o código e dificulta debug |
| Ktor | Nativo Kotlin, leve, coroutines de ponta a ponta | Ecossistema menor, mais código manual de infraestrutura |
| Quarkus | Startup em milissegundos, imagem nativa madura | Menos material e menos gente com experiência |
| Micronaut | Injeção em tempo de compilação, sem reflection | Comunidade menor que a do Spring |

### Virtual Threads em vez de reativo

Com Java 21, ative virtual threads e ganhe boa parte do benefício de concorrência sem reescrever o código em estilo reativo.

```yaml
spring:
  threads:
    virtual:
      enabled: true
```

Para serviço que passa a maior parte do tempo esperando I/O (o caso mais comum), essa é hoje a primeira escolha. Só vá para WebFlux se medir e comprovar que não basta.

### GraalVM Native Image

Reduz startup para dezenas de milissegundos e corta memória de forma significativa, o que ajuda muito em serviço que escala para zero. O custo é build mais lento e limitação com reflection e proxy dinâmico, o que exige configuração extra.

### Comunicação

- **REST com OpenAPI** é o padrão para chamada síncrona entre serviços internos e para API pública.
- **gRPC** entrega contrato forte e payload binário menor, com ganho real em chamada interna de alto volume. Custa ferramental extra e menos legibilidade em debug.
- **Mensageria (RabbitMQ, Kafka, SQS)** é a escolha certa quando o fluxo tolera consistência eventual. Desacopla de verdade, porque o produtor não precisa saber quem consome.

Quando o histórico completo do que aconteceu é requisito de negócio (auditoria financeira, reconstrução de saldo, trilha regulatória), a persistência por eventos é uma opção. O padrão e o custo dele estão em `event_sourcing.md`. A base conceitual de comunicação por eventos está em `event-driven-architecture.md`. O ferramental de plataforma (Config Server, Eureka, Gateway, OpenFeign, Resilience4j) está em `spring-cloud.md`. Os três no mesmo diretório.

### Outros

- **Spring Modulith** organiza um monolito em módulos com fronteira verificada em teste, permitindo extrair serviços depois com custo baixo.
- **Testcontainers** para teste de integração com banco e broker reais, em vez de mock frágil ou banco em memória que se comporta diferente.
- **OpenTelemetry** como padrão de trace e métrica, evitando amarrar o serviço a um fornecedor de observabilidade.

---

## 8. Resumo Final

### Principais aprendizados

- Microserviço resolve problema organizacional e de escala, não problema de código feio. Código feio continua feio distribuído.
- A parte difícil não é dividir, é definir onde cortar. Fronteira errada custa muito mais caro que monolito.
- Kotlin ajuda com null safety, `data class` e `sealed class`, mas exige três cuidados com Spring, os plugins `kotlin-spring` e `kotlin-jpa`, o prefixo `@field:` nas validações, e entidade JPA que não é `data class`.
- Timeout, retry idempotente e circuit breaker não são refinamento, são requisito básico de qualquer chamada entre serviços.
- Consumidor de evento sempre precisa de idempotência, porque o broker entrega pelo menos uma vez.
- Virtual threads no Java 21 cobrem a maior parte dos casos que antes exigiam stack reativa.

### Guia rápido de consulta

| Preciso de | Use |
|---|---|
| Chamar outro serviço via HTTP | `RestClient` com timeout de conexão e de leitura |
| Proteger contra serviço lento ou fora | `@CircuitBreaker` mais `@Retry` do Resilience4j |
| Comunicação desacoplada | Evento em RabbitMQ ou Kafka, com consumidor idempotente |
| Contrato de erro da API | `@RestControllerAdvice` com DTO de erro versionado |
| Validar entrada | `@field:` no `data class` de request |
| Configuração tipada | `@ConfigurationProperties` em `data class` |
| Alta concorrência de I/O | Virtual threads (`spring.threads.virtual.enabled: true`) |
| Teste com banco real | Testcontainers |
| Rastrear requisição entre serviços | Micrometer Tracing com OpenTelemetry |
| Startup rápido e pouca memória | GraalVM Native Image ou Quarkus |

### Checklist antes de subir um microserviço novo

- [ ] Banco próprio, sem tabela compartilhada com outro serviço
- [ ] API versionada no path
- [ ] Timeout explícito em todo cliente HTTP
- [ ] Circuit breaker e fallback nas chamadas externas
- [ ] Idempotência em todo consumidor de evento
- [ ] Log estruturado em JSON com `correlationId`
- [ ] Trace distribuído habilitado
- [ ] `liveness` e `readiness` separados no Actuator
- [ ] Segredo fora do repositório
- [ ] Teste de integração com Testcontainers no pipeline
- [ ] Documentação OpenAPI publicada
- [ ] Alerta e dashboard criados antes do primeiro deploy

O padrão de publicação da documentação OpenAPI usado nos projetos Bolt, com o frontend renderizando o Swagger UI, está em `guia-implementacao-swagger-bolt.md`, no mesmo diretório.

---

## 9. Guias Relacionados

Todos os arquivos abaixo estão neste mesmo diretório.

### Comunicação entre serviços

| Arquivo | Quando consultar |
|---|---|
| `spring-cloud.md` | Config Server, Eureka, API Gateway, OpenFeign e Resilience4j. É o complemento mais direto deste guia |
| `rest-client.md` | Cliente HTTP síncrono do Spring 6, com builder, interceptor e tratamento de erro |
| `rest-template.md` | Manutenção de serviço legado que ainda usa `RestTemplate` |
| `event-driven-architecture.md` | Conceitos de evento, produtor, consumidor e broker |
| `rabbitmq_resiliencia.md` | Publisher confirms, Outbox Pattern, DLQ e garantia de entrega |
| `application_events.md` | Eventos internos do Spring, dentro do mesmo processo. Não confundir com evento de broker |
| `event_sourcing.md` | Persistir a sequência de fatos em vez do estado atual |

### Construção do serviço

| Arquivo | Quando consultar |
|---|---|
| `hexagonal.md` | Organização interna quando o núcleo de negócio precisa ficar isolado de banco e integração |
| `spring-data.md` | Filtro dinâmico, paginação e ordenação com `Specification` |
| `asserts.md` | Validação entre campos com `@AssertTrue` e `@AssertFalse` |
| `cache.md` | Cache local e distribuído para reduzir chamada entre serviços |
| `sql_vs_nosql.md` | Escolher o banco de cada serviço, já que cada um tem o seu |
| `spring_batch.md` | Processamento em massa que costuma ser job, não microserviço novo |
| `records.md` | Equivalente Java da `data class`, útil quando o time transita entre as duas linguagens |

### Operação e qualidade

| Arquivo | Quando consultar |
|---|---|
| `datadog_observabilidade.md` | Log, métrica e trace distribuído ponta a ponta |
| `docker-containerizacao-local.md` | Subir a aplicação e as dependências localmente com um comando |
| `maven-test.md` | Comandos de teste por cenário, do feedback rápido ao build completo |
| `guia-implementacao-swagger-bolt.md` | Padrão Bolt de publicação da documentação OpenAPI |
| `worktree.md` | Trabalhar em mais de uma branch ao mesmo tempo, útil quando se mexe em vários serviços na mesma tarefa |
