# Estudo sobre Estratégias de Cache em Aplicações Spring Boot

Este documento apresenta uma análise técnica sobre as soluções de cache em memória e distribuído utilizando o ecossistema Spring, com foco em performance, escalabilidade e facilidade de manutenção.

---

## 1. Abstração de Cache do Spring
O Spring Framework fornece uma poderosa abstração para cache que permite adicionar suporte a cache de forma declarativa via anotações, sem acoplar o código a uma implementação específica.

### Configuração Inicial
Para ativar o suporte a cache, adicione a anotação na classe principal:
```kotlin
@EnableCaching
@SpringBootApplication
class Application
```

---

## 2. Provedores de Cache Local (In-Memory)

Ideal para aplicações de instância única ou dados que não precisam de sincronismo entre múltiplos nós.

### A. ConcurrentMap (Default)
*   **Funcionamento**: Utiliza um `ConcurrentHashMap` da própria JVM.
*   **Vantagens**: Sem dependências extras, extremamente leve.
*   **Desvantagens**: Sem suporte nativo a expiração (TTL).

### B. Caffeine (Recomendado para Performance Local)
A biblioteca Caffeine é amplamente considerada a solução em memória mais eficiente para Java/Kotlin.
*   **Recursos**: TTL, TTI, Limite de tamanho, Refrescamento Automático.
*   **Exemplo de Configuração (`application.yml`)**:
```yaml
spring:
  cache:
    caffeine:
      spec: expireAfterWrite=60m,maximumSize=500
```

---

## 3. Cache Distribuído (Escalabilidade Horizontal)

Quando temos múltiplas instâncias da aplicação rodando simultaneamente (Ex: Kubernetes), um cache local pode gerar inconsistências (o nó A tem um dado cacheado e o nó B tem outro). Para isso, utilizamos **Cache Distribuído**.

### A. Redis (O Padrão de Mercado)
O Redis é um armazenamento de estrutura de dados chave-valor em memória, persistente e ultra-rápido.
*   **Por que usar?**: Permite que todas as instâncias da sua API compartilhem o mesmo cache.
*   **Vantagens**: Suporta estruturas complexas (listas, sets), persistência opcional e alta disponibilidade (Cluster/Sentinel).

### B. Hazelcast
Uma grade de dados em memória distribuída.
*   **Diferencial**: O Hazelcast pode rodar "embutido" na aplicação, onde os próprios nós das instâncias conversam entre si para sincronizar os dados, sem precisar de um servidor centralizado como o Redis.

---

## 4. Exemplos de Uso Distribuído (Redis)

### Configuração de Dependência (`pom.xml`)
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

### Exemplo: Cache de Sessão de Usuário ou Tokens
Ao utilizar Redis, o Spring gerencia automaticamente a serialização dos objetos.
```kotlin
@Cacheable(value = ["sessoesAtivas"], key = "#token")
fun validarToken(token: String): SessaoDto {
    // Busca informações complexas de permissões no banco
    return authRepository.findByToken(token)
}
```

### Exemplo: Cache de Resultados de Relatórios Pesados
Cenário onde o relatório leva 30 segundos para ser gerado e várias pessoas consultam o mesmo dado.
```kotlin
@Cacheable(value = ["relatoriosMensais"], key = "#anoMes")
fun gerarRelatorioConsumo(anoMes: String): RelatorioPdfDto {
    // Lógica pesada de agregação de dados
    return processadorRelatorios.executar(anoMes)
}
```

---

## 5. Comparativo Técnico: Local vs. Distribuído

| Característica | Local (Caffeine) | Distribuído (Redis) |
| :--- | :--- | :--- |
| **Velocidade** | Ultra-rápida (nanossegundos) | Rápida (milissegundos - rede) |
| **Sincronismo** | Não (cada nó tem o seu) | Sim (global) |
| **Persistência** | Não (perde no restart) | Sim (configurável) |
| **Complexidade** | Baixa | Média (exige servidor externo) |

---

## 6. Anotações de Uso Prático

### `@Cacheable`
Indica que o resultado do método deve ser cacheado. 
```kotlin
@Cacheable(value = ["taxas"], key = "#moeda")
fun getTaxa(moeda: String): BigDecimal = ...
```

### `@CacheEvict`
Remove dados do cache para evitar dados obsoletos.
```kotlin
@CacheEvict(value = ["taxas"], allEntries = true)
fun atualizarTabela() { ... }
```

---

## 7. Boas Práticas
1.  **Serialização**: Para cache distribuído (Redis), suas classes DTO **devem** implementar `Serializable`.
2.  **Monitoramento**: Utilize ferramentas como o `Redis Insight` para visualizar o que está ocupando espaço no cache distribuído.
3.  **Fallback**: O cache deve ser uma melhoria, não uma dependência. Se o Redis cair, a aplicação deve continuar funcionando (consultando o banco de dados diretamente).
