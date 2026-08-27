# Spring Cloud — Guia de Uso (Java 17+ / Spring Boot 3)

> Guia prático e didático sobre Spring Cloud no ecossistema Spring Boot 3.x, cobrindo Config Server, Service Discovery (Eureka), API Gateway, OpenFeign e Resilience4j.

---

## 1. Introdução

### Contexto

**Spring Cloud** é um conjunto de ferramentas que estende o Spring Boot para resolver problemas recorrentes em **sistemas distribuídos** e **arquiteturas baseadas em microsserviços**: configuração centralizada, descoberta de serviços, roteamento, resiliência, balanceamento de carga, rastreamento distribuído e mais.

Ele não é um framework único — é um **guarda-chuva** de projetos integrados, cada um focado em um problema:

| Projeto | Problema que resolve |
|---|---|
| Spring Cloud Config | Configurações externas e centralizadas |
| Spring Cloud Netflix Eureka | Registro e descoberta de serviços |
| Spring Cloud Gateway | API Gateway reativo (roteamento, filtros, segurança de borda) |
| Spring Cloud OpenFeign | Cliente HTTP declarativo entre microsserviços |
| Spring Cloud LoadBalancer | Balanceamento de carga client-side |
| Spring Cloud Circuit Breaker (Resilience4j) | Tolerância a falhas, retry, fallback, bulkhead |
| Spring Cloud Sleuth / Micrometer Tracing | Tracing distribuído |
| Spring Cloud Stream | Mensageria desacoplada (Kafka, RabbitMQ) |

### Problema que resolve

Em uma arquitetura monolítica, configuração, comunicação interna e tolerância a falhas são triviais. Já em microsserviços surgem problemas como:

- Onde cada serviço encontra os outros (IPs/portas mudam)?
- Como atualizar uma configuração sem rebuildar e redeployar dezenas de serviços?
- Como impedir que a falha de um serviço derrube toda a cadeia?
- Como expor uma única porta externa quando há dezenas de serviços internos?
- Como rastrear uma requisição que atravessa 5 serviços?

Spring Cloud oferece **respostas idiomáticas e integradas** para cada um desses pontos.

### Quando faz sentido aplicar

- Sistema dividido em **3+ serviços** que se comunicam.
- Necessidade de **deploy independente** por serviço.
- Equipes separadas que precisam de **isolamento de domínio**.
- Cenários com **escalabilidade horizontal heterogênea** (alguns serviços precisam de mais réplicas).

**Não aplicar** em monólitos ou em projetos pequenos: o custo operacional (mais infra, mais latência de rede, observabilidade complexa) supera o ganho.

### Versionamento

Spring Cloud usa **calendário** (ex: `2023.0.x`, `2024.0.x`), não SemVer. Cada release "train" é compatível com uma faixa do Spring Boot:

| Spring Boot | Spring Cloud |
|---|---|
| 3.4.x | 2024.0.x (Moorgate) |
| 3.3.x | 2023.0.x (Leyton) |
| 3.2.x | 2023.0.x |
| 3.0.x – 3.1.x | 2022.0.x (Kilburn) |

Sempre verifique a [matriz oficial de compatibilidade](https://spring.io/projects/spring-cloud) antes de subir versões.

---

## 2. Sintaxe Básica

### Configuração do BOM (Bill of Materials)

A forma idiomática de gerenciar versões. Importe o BOM no `dependencyManagement` (Maven) ou via `dependency-management plugin` (Gradle):

```xml
<properties>
    <java.version>17</java.version>
    <spring-cloud.version>2024.0.0</spring-cloud.version>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

Depois disso, basta declarar `spring-cloud-starter-*` sem versão.

### `bootstrap.yml` vs `application.yml`

Em versões antigas (Spring Cloud 2020.x e anteriores), o `bootstrap.yml` era carregado **antes** do `application.yml` para configurar Config Server. A partir do **Spring Cloud 2020.0+**, ele foi descontinuado por padrão — agora usa-se `spring.config.import`:

```yaml
# application.yml
spring:
  application:
    name: order-service
  config:
    import: "optional:configserver:http://localhost:8888"
```

Só reative o bootstrap (com `spring-cloud-starter-bootstrap`) se houver requisito legado.

---

## 3. Exemplos Simples e Práticos

### 3.1. Service Discovery com Eureka

**Servidor (Eureka Server):**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

```yaml
# application.yml do Eureka Server
server:
  port: 8761
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

**Cliente (qualquer microsserviço):**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```yaml
spring:
  application:
    name: order-service
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

A partir daí, o `order-service` é registrado e descoberto por nome lógico (`order-service`), não por IP.

### 3.2. Config Server centralizado

**Servidor:**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication { /* ... */ }
```

```yaml
server:
  port: 8888
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/empresa/configs-repo
          default-label: main
```

**Cliente:**

```yaml
spring:
  application:
    name: order-service
  config:
    import: "optional:configserver:http://localhost:8888"
```

O Config Server lerá `order-service.yml` e `order-service-<profile>.yml` do repositório Git.

### 3.3. Spring Cloud Gateway com roteamento dinâmico

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

```yaml
spring:
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true   # roteia automaticamente para serviços registrados no Eureka
          lower-case-service-id: true
      routes:
        - id: order-route
          uri: lb://order-service          # lb = load-balanced via discovery
          predicates:
            - Path=/api/orders/**
          filters:
            - StripPrefix=1
            - AddRequestHeader=X-Origem, gateway
```

Agora `GET /api/orders/123` chega no `order-service` como `/orders/123`.

### 3.4. OpenFeign — cliente HTTP declarativo

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableFeignClients
public class OrderServiceApplication { /* ... */ }
```

```java
@FeignClient(name = "customer-service")
public interface CustomerClient {

    @GetMapping("/customers/{id}")
    CustomerDTO buscarPorId(@PathVariable("id") Long id);
}
```

Uso direto via injeção:

```java
@Service
public class OrderService {

    private final CustomerClient customerClient;

    public OrderService(CustomerClient customerClient) {
        this.customerClient = customerClient;
    }

    public OrderDTO criar(OrderRequest request) {
        CustomerDTO cliente = customerClient.buscarPorId(request.customerId());
        // ... lógica de criação
    }
}
```

O Feign integra automaticamente com Eureka + LoadBalancer + Resilience4j.

### 3.5. Resilience4j — Circuit Breaker e Retry

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
</dependency>
```

```yaml
resilience4j:
  circuitbreaker:
    instances:
      customerService:
        sliding-window-size: 10
        failure-rate-threshold: 50
        wait-duration-in-open-state: 10s
  retry:
    instances:
      customerService:
        max-attempts: 3
        wait-duration: 500ms
```

```java
@Service
public class OrderService {

    private final CustomerClient customerClient;

    @CircuitBreaker(name = "customerService", fallbackMethod = "clienteFallback")
    @Retry(name = "customerService")
    public CustomerDTO buscarCliente(Long id) {
        return customerClient.buscarPorId(id);
    }

    private CustomerDTO clienteFallback(Long id, Throwable ex) {
        log.warn("Falha ao consultar cliente {}, retornando dados padrão", id);
        return new CustomerDTO(id, "Indisponível", null);
    }
}
```

---

## 4. Boas Práticas

### Arquitetura

- **Um Eureka Server por ambiente** (dev, hml, prd) — nunca compartilhar entre ambientes.
- **Config Server por ambiente** com repositórios Git versionados; trate configuração como código.
- **Gateway como única porta de entrada externa** — todos os serviços internos ficam atrás dele.
- **Não exponha Eureka, Config Server ou Actuator endpoints publicamente.** Use rede privada ou VPN.

### Configuração

- Centralize defaults no Config Server; mantenha apenas o **mínimo essencial** em `application.yml` local (basicamente `spring.application.name` e `spring.config.import`).
- Use **perfis** (`order-service-prd.yml`) para variações por ambiente.
- Para **segredos** (senhas, chaves), prefira **Vault** ou **AWS Secrets Manager**, não Git em texto puro. O Spring Cloud Config suporta `{cipher}...` para criptografia.

### Resiliência

- **Sempre defina fallback** quando usar `@CircuitBreaker`. Um circuit aberto sem fallback explode em runtime.
- Combine `@Retry` + `@CircuitBreaker` com cautela: o retry só faz sentido para falhas transientes (timeout, 5xx esporádico). Não tente retry em 4xx.
- **Timeout explícito** em todo cliente Feign (a default é "infinito" — péssimo):
  ```yaml
  spring:
    cloud:
      openfeign:
        client:
          config:
            default:
              connect-timeout: 2000
              read-timeout: 5000
  ```

### Observabilidade

- Use **Micrometer Tracing** (sucessor do Sleuth a partir de Spring Boot 3) com Zipkin/Tempo/Jaeger.
- Propague o `traceId` automaticamente via OpenFeign + WebClient.
- Exponha `/actuator/health`, `/actuator/info`, `/actuator/metrics` em todos os serviços.

### Performance

- Em **Spring Cloud Gateway**, evite filtros bloqueantes — ele é reativo (WebFlux). Misturar código bloqueante quebra o modelo.
- Configure **LoadBalancer cache** (`spring.cloud.loadbalancer.cache.ttl`) para reduzir chamadas ao Eureka.
- Em alto throughput, prefira **gRPC + Spring Cloud LoadBalancer** ou **Spring Cloud Stream + Kafka** a chamadas HTTP síncronas em cascata.

### Nomenclatura

- `spring.application.name` em **kebab-case** (`order-service`, `customer-service`) — é o identificador no Eureka.
- Repositórios de configuração organizados por serviço: `order-service.yml`, `order-service-prd.yml`, `application.yml` (defaults globais).

---

## 5. Limitações e Armadilhas

### Armadilhas Comuns

| Armadilha | Por que dói | Como evitar |
|---|---|---|
| Esperar 30s para um serviço aparecer no Eureka | Eureka usa *self-preservation* e renovação de leases; defaults são lentos | Em dev, reduza `eureka.instance.lease-renewal-interval-in-seconds` e `lease-expiration-duration-in-seconds` |
| `@FeignClient` sem fallback | Falha do downstream propaga 500 para o usuário final | Combine com `@CircuitBreaker` e fallback explícito |
| Misturar versões de Spring Boot e Spring Cloud incompatíveis | Erros obscuros no startup (`NoSuchMethodError`, `BeanCreationException`) | Sempre use o BOM oficial e respeite a matriz de compatibilidade |
| Subir Config Server sem fallback | Se Config Server cai, todos os serviços novos não sobem | Use `optional:` no import e mantenha defaults sensatos em `application.yml` |
| `lb://` sem registro no Discovery | `503 Service Unavailable` no Gateway | Verifique se o serviço está registrado e UP no Eureka |
| `@RibbonClient` em código novo | Ribbon foi **descontinuado** | Use Spring Cloud LoadBalancer (vem por padrão) |
| Hystrix em projeto novo | Hystrix foi **descontinuado** desde 2018 | Use Resilience4j |
| `bootstrap.yml` em Spring Cloud 2020+ | Ignorado silenciosamente | Use `spring.config.import` |

### Restrições Técnicas

- **Spring Cloud Gateway exige WebFlux** — não funciona em projeto Spring MVC tradicional.
- **OpenFeign não é reativo por padrão** — para reativo, use `WebClient` ou `Reactive Feign` (projeto comunitário).
- **Eureka não é forte em multi-região/multi-cluster** — para isso, prefira Consul, Kubernetes Service Discovery ou Nacos.
- **Config Server não tem UI** — gestão é via Git ou Vault.

### Erros Comuns

1. **`Connection refused: localhost/127.0.0.1:8761`** → Eureka Server não está rodando ou URL errada.
2. **`No instances available for service-name`** → Serviço não registrado, lease expirou, ou nome errado.
3. **`Load balancer does not contain an instance for service`** → Mesmo problema, geralmente após restart.
4. **`Request method 'POST' not supported`** ao usar Gateway → `StripPrefix` configurado incorretamente.
5. **Configuração não recarrega** → Adicionar `@RefreshScope` no bean + chamar `/actuator/refresh` (ou usar Spring Cloud Bus).

---

## 6. Quando Usar e Quando Evitar

### Use quando

- Arquitetura com **3 ou mais serviços** que se comunicam.
- Necessidade de **configuração centralizada** versionada.
- Múltiplas instâncias do mesmo serviço (balanceamento, alta disponibilidade).
- Equipes independentes com ciclos de deploy próprios.
- Necessidade explícita de **resiliência** (circuit breaker, retry, bulkhead).
- Ambiente **on-premises ou cloud-agnostic** onde Kubernetes não cobre tudo (ex: VMs, Docker Swarm).

### Evite quando

- **Monólito** ou aplicação com 1–2 serviços: overhead operacional não compensa.
- **Kubernetes maduro** já em uso: muitas funcionalidades (Service Discovery via DNS, Config via ConfigMap/Secret, traffic shaping via Istio/Linkerd) são fornecidas pela plataforma. Spring Cloud vira redundância.
- **Serverless puro** (AWS Lambda, Cloud Run): o modelo de instâncias longas que o Eureka pressupõe não combina.
- Time **sem experiência operacional** em sistemas distribuídos — debugar microsserviços é exponencialmente mais difícil.

### Critério rápido

> Se você tem dúvida se precisa, **provavelmente não precisa**. Comece monolítico, quebre quando a dor justificar.

---

## 7. Alternativas Modernas

### Por componente

| Spring Cloud | Alternativa moderna | Quando preferir |
|---|---|---|
| Eureka | **Kubernetes Service** + DNS | Já roda em K8s — discovery vem grátis |
| Eureka | **Consul** / **Nacos** | Multi-datacenter, multi-linguagem, KV store |
| Config Server | **K8s ConfigMap + Secret** | Stack puramente K8s |
| Config Server | **HashiCorp Vault** | Segredos críticos com auditoria |
| Spring Cloud Gateway | **Kong**, **Traefik**, **Envoy** | Gateway poliglota / sidecar mesh |
| Spring Cloud Gateway | **APISIX** | Plugins dinâmicos e UI rica |
| OpenFeign | **WebClient (reativo)** | Reatividade ponta a ponta |
| OpenFeign | **gRPC** + Spring Boot starter | Contratos fortes, baixa latência, streaming |
| Resilience4j | **Istio/Linkerd** (mesh) | Resiliência declarativa fora do código |
| Spring Cloud Sleuth | **Micrometer Tracing** (já é o substituto oficial) | Spring Boot 3+ |
| Spring Cloud Stream | **Spring for Apache Kafka** direto | Mais controle, sem abstração extra |

### Comparativo rápido: Spring Cloud vs Service Mesh (Istio)

| Aspecto | Spring Cloud | Service Mesh |
|---|---|---|
| Onde vive | Dentro da aplicação (libs) | Sidecar (fora da aplicação) |
| Linguagem | Java/Kotlin/Spring | Agnóstico |
| Resiliência | `@CircuitBreaker` no código | Política declarativa no mesh |
| Curva de aprendizado | Familiar para devs Spring | Exige conhecimento de infra/K8s |
| Upgrade | Rebuilda app | Atualiza sidecar |
| Observabilidade | Configurar em cada serviço | Automática para todo tráfego |

Em equipes Java-only que controlam ponta a ponta, **Spring Cloud é mais produtivo**. Em ambientes poliglotas em K8s, **mesh ganha**.

---

## 8. Resumo Final

### Principais aprendizados

1. **Spring Cloud é um conjunto de projetos**, não um framework único — escolha os componentes que você realmente precisa.
2. **Versão é por trem (calendar release)** — sempre use o BOM e respeite a matriz de compatibilidade com Spring Boot.
3. **`bootstrap.yml` morreu** em Spring Cloud 2020+; use `spring.config.import`.
4. **Hystrix e Ribbon estão descontinuados** — use Resilience4j e Spring Cloud LoadBalancer.
5. **Sempre tenha fallback** em chamadas externas: timeout, circuit breaker, retry só onde faz sentido.
6. **Não use em monólitos.** O custo operacional não compensa.
7. **Em ambientes Kubernetes maduros**, considere usar primitivas nativas (Service, ConfigMap) + service mesh em vez de Spring Cloud completo.

### Guia rápido de consulta

| Preciso de... | Use |
|---|---|
| Configuração centralizada | `spring-cloud-config-server` + `spring.config.import` |
| Registro/descoberta de serviços | `spring-cloud-starter-netflix-eureka-*` |
| API Gateway | `spring-cloud-starter-gateway` |
| Cliente HTTP declarativo | `spring-cloud-starter-openfeign` |
| Balanceamento client-side | `spring-cloud-loadbalancer` (transitivo) |
| Circuit breaker / retry / bulkhead | `spring-cloud-starter-circuitbreaker-resilience4j` |
| Tracing distribuído | `micrometer-tracing-bridge-otel` + exporter |
| Mensageria desacoplada | `spring-cloud-stream-binder-kafka` |

### Mini-receita para um novo serviço

```yaml
# application.yml mínimo de um microsserviço Spring Cloud típico
spring:
  application:
    name: order-service
  config:
    import: "optional:configserver:http://config-server:8888"

eureka:
  client:
    service-url:
      defaultZone: http://eureka:8761/eureka/

management:
  endpoints:
    web:
      exposure:
        include: health,info,refresh,metrics,prometheus
  tracing:
    sampling:
      probability: 1.0

resilience4j:
  circuitbreaker:
    instances:
      default:
        sliding-window-size: 20
        failure-rate-threshold: 50
        wait-duration-in-open-state: 15s

spring.cloud.openfeign.client.config.default:
  connect-timeout: 2000
  read-timeout: 5000
```

Com isso, o serviço já tem: configuração externa, descoberta, observabilidade, resiliência e timeout. Adicione domínio e regras de negócio por cima.

---

## Referências

- [Spring Cloud — site oficial](https://spring.io/projects/spring-cloud)
- [Matriz de compatibilidade Boot ↔ Cloud](https://spring.io/projects/spring-cloud#overview)
- [Resilience4j — documentação](https://resilience4j.readme.io/)
- [Micrometer Tracing](https://docs.micrometer.io/tracing/reference/)
- [Spring Cloud Gateway — referência](https://docs.spring.io/spring-cloud-gateway/reference/)
