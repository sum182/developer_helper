# Manual: SQL vs NoSQL

Este guia explica as diferenças fundamentais entre bancos de dados Relacionais (SQL) e Não Relacionais (NoSQL), ajudando a escolher a tecnologia certa para cada problema.

---

## 1. Banco de Dados Relacional (SQL)

Bancos como **PostgreSQL, MySQL e SQL Server** focam em estrutura e consistência.

### Características:
- **Esquema Rígido**: Você precisa definir as tabelas e colunas antes de inserir dados.
- **Relacionamentos**: Excelente para lidar com dados complexos que se conectam (ex: Empresa tem muitos Usuários).
- **ACID**: Garante que as transações sejam 100% seguras (ou tudo salva, ou nada salva).

### Quando usar?
- Quando a consistência dos dados é crítica (Sistemas Financeiros).
- Quando os dados são altamente estruturados e não mudam de forma o tempo todo.
- Quando você precisa de relatórios complexos cruzando várias tabelas (JOINs).

---

## 2. Banco de Dados Não Relacional (NoSQL)

Bancos como **MongoDB, Redis e Cassandra** focam em flexibilidade e escala.

### Características:
- **Esquema Flexível**: Você pode salvar um documento hoje com 5 campos e amanhã com 10, sem alterar o banco.
- **Escalabilidade Horizontal**: Projetados para rodar em dezenas de servidores ao mesmo tempo com facilidade.
- **Tipos Variados**: Documento (JSON), Chave-Valor, Grafos ou Colunar.

### Quando usar?
- **Big Data**: Volumes imensos de dados que um banco SQL não aguentaria.
- **Dados Semiformatados**: Logs de sistema, carrinhos de compras, perfis de redes sociais.
- **Desenvolvimento Ágil**: Quando a estrutura do dado muda muito rápido e você não quer lidar com migrações de banco (Migrations) o tempo todo.

### Principais Vantagens do NoSQL:

1. **Escalabilidade Horizontal (Sharding)**:
   Ao contrário do SQL, onde você precisa de servidores cada vez maiores e mais caros (escala vertical), no NoSQL você pode adicionar centenas de servidores comuns para dividir o processamento. É ideal para crescer conforme a demanda financeira aumenta.

2. **Alta Disponibilidade e Replicação**:
   Os bancos NoSQL são desenhados para serem distribuídos em **Clusters** (vários servidores).
   - **Como funciona?** Os dados são automaticamente copiados (**replicados**) em diferentes servidores (nós).
   - **Resiliência**: Se o "Servidor A" queimar, o "Servidor B" (que tem uma cópia idêntica dos dados) assume as requisições instantaneamente. O usuário final nem percebe a falha. É o conceito de sistema **"Always On"**.

3. **Performance de Escrita (Baixa Latência)**:
   Como não há necessidade de verificar chaves estrangeiras ou fazer bloqueios de tabela (locks) complexos em cada transação, a gravação de dados é extremamente veloz.

4. **Esquema Flexível (Schemaless)**:
   Você não precisa parar o banco para adicionar novas informações (como um `ALTER TABLE`). O software evolui mais rápido, permitindo lançar novos produtos financeiros com agilidade.

5. **Otimização para Dados Semiformatados**:
   Perfeito para logs de auditoria, históricos de cliques, metadados de transações e telemetria, onde os dados podem mudar de formato frequentemente.

---

## 3. Tabela Comparativa (Resumo para o Teste)

| Característica | SQL (Relacional) | NoSQL (Não Relacional) |
| :--- | :--- | :--- |
| **Estrutura** | Tabelas com linhas e colunas fixas. | Documentos (JSON), Chave-Valor, etc. |
| **Escala** | Vertical (Aumenta o servidor). | Horizontal (Adiciona mais servidores). |
| **Consultas** | SQL (SELECT, JOIN, WHERE). | APIs focadas em coleções/documentos. |
| **Foco** | Integridade e Relacionamento. | Velocidade e Disponibilidade. |

---

## 4. Cenário Híbrido (O que as empresas usam hoje)

A maioria das grandes empresas usa os dois ao mesmo tempo:
- **SQL**: Para o cadastro principal do cliente e transações financeiras (onde não pode haver erro).
- **NoSQL**: Para o histórico de navegação, cache de performance (Redis) e logs de eventos (MongoDB).

**Conclusão**: Não existe um "melhor". Existe a ferramenta certa para cada tipo de dado!

---

## 5. Transações no NoSQL (O NoSQL controla transação?)

Antigamente, dizia-se que NoSQL não tinha transação. **Isso mudou.** Bancos modernos como o MongoDB (desde a v4.0) suportam transações multi-documento de forma similar ao SQL.

### Como funciona?
1. **Transação de Documento Único (Atômica)**: No NoSQL, qualquer operação em um único documento (ex: salvar um cliente com todos os seus endereços embutidos) é **Sempre Atômica**. Ou salva tudo, ou nada. Você não precisa de configuração extra para isso.
2. **Transações Multi-Documento**: Quando você precisa salvar dados em coleções diferentes (ex: Criar Cliente E Criar Registro Financeiro em outra coleção), o Spring Boot gerencia isso com o mesmo `@Transactional` que você já conhece.

### Exemplo em Java:

```java
@Service
public class FinanceService {

    @Transactional // O Spring inicia uma sessão de transação no MongoDB
    public void processNewClient(Client client, Account account) {
        clientRepo.save(client);
        accountRepo.save(account);
        
        if (erro) throw new RuntimeException(); // Ambos sofrem Rollback no NoSQL!
    }
}
```

### Configuração Necessária:
Para que o `@Transactional` funcione no NoSQL, você deve configurar um **Transaction Manager** na sua classe de configuração:

```java
@Configuration
public class MongoConfig {
    @Bean
    MongoTransactionManager transactionManager(MongoDatabaseFactory dbFactory) {
        return new MongoTransactionManager(dbFactory);
    }
}
```

**Resumo para o Teste**: Sim, o NoSQL moderno suporta transações ACID, mas a recomendação de design é tentar **embutir** os dados relacionados para que a transação ocorra naturalmente em um único documento, o que é muito mais performático.

---

## 5. Exemplo Prático: CRUD NoSQL (MongoDB) em Java

Em Spring Boot, usar um banco NoSQL como o MongoDB é quase idêntico ao JPA/SQL, mudando apenas as anotações.

### A. Dependência (`pom.xml`)
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

### B. Entidade (O "Documento" JSON)
No NoSQL, chamamos as classes de `@Document`.

```java
@Data
@Document(collection = "clients") // Em vez de @Table
public class Client {
    @Id
    private String id; // No MongoDB o ID padrão é uma String (ObjectId)
    
    private String name;
    private String email;
    private List<String> tags; // Listas são salvas diretamente no JSON!
}
```

### C. Repositório
Em vez de `JpaRepository`, usamos `MongoRepository`.

```java
@Repository
public interface ClientRepository extends MongoRepository<Client, String> {
    Optional<Client> findByEmail(String email);
}
```

### D. Service
A lógica permanece a mesma, o Spring Data abstrai tudo.

```java
@Service
@RequiredArgsConstructor
public class ClientService {
    private final ClientRepository repository;

    public Client create(Client client) {
        return repository.save(client);
    }

    public List<Client> findAll() {
        return repository.findAll();
    }
}
```

### O que mudou na prática?
1. **Sem Migrations**: Se você adicionar o campo `telefone` na classe `Client`, o MongoDB salva automaticamente sem você precisar rodar um script SQL `ALTER TABLE`.
2. **Sem JOINs**: Se um cliente tiver endereços, você pode salvar a lista de endereços **dentro** do documento do cliente, em vez de ter uma tabela separada.

---

## 6. Consultas Avançadas (Paginação e Filtros Dinâmicos)

Na área financeira, a velocidade e a precisão das consultas são vitais. Veja como o NoSQL lida com isso:

### A. Paginação e Ordenação
É idêntica ao SQL no Spring Boot, o que facilita a vida do desenvolvedor.

```java
public Page<Client> findAllPaged(int page, int size) {
    Pageable pageable = PageRequest.of(page, size, Sort.by("name").ascending());
    return repository.findAll(pageable);
}
```

### B. Filtros Dinâmicos (Substituto da Specification)
Como não há "Criteria API" padrão do JPA, usamos o **MongoTemplate** com **Criteria** do MongoDB.

```java
public List<Client> searchClients(String name, String email) {
    Query query = new Query();
    if (name != null) query.addCriteria(Criteria.where("name").regex(name, "i"));
    if (email != null) query.addCriteria(Criteria.where("email").is(email));
    
    return mongoTemplate.find(query, Client.class);
}
```

### C. Agregações (O "GROUP BY" do NoSQL)
Para relatórios financeiros (ex: total de gastos por categoria), usamos o **Aggregation Framework**.

```java
public List<TotalPorTag> aggregateByTag() {
    Aggregation agg = Aggregation.newAggregation(
        Aggregation.unwind("tags"),
        Aggregation.group("tags").count().as("total")
    );
    return mongoTemplate.aggregate(agg, Client.class, TotalPorTag.class).getMappedResults();
}
```

---

## 7. Relacionamentos no NoSQL (Cliente x Endereços)

Diferente do SQL, onde você **sempre** teria duas tabelas e um JOIN, no NoSQL você tem duas opções:

### Opção A: Embutir (Embedding) - Recomendado para 1-para-Muitos pequenos
Você salva os endereços **dentro** do documento do cliente. É o cenário mais comum para cadastros.

```java
@Document(collection = "clients")
public class Client {
    @Id private String id;
    private String name;
    
    // Lista de endereços embutida diretamente no JSON
    private List<Address> addresses;
}

// Classe simples de Endereço (não é uma entidade/tabela própria no NoSQL)
public class Address {
    private String street;
    private String city;
    private String zipCode;
}
```
- **Vantagem**: Com uma única consulta você tem todos os dados. Performance altíssima.
- **Desvantagem**: Se o cliente tiver 1 milhão de endereços, o documento ficaria pesado demais (o MongoDB tem limite de 16MB por documento).

### Opção B: Referência (Linking) - Parecido com SQL
Você salva os documentos em coleções separadas e usa o ID para conectá-los.

```java
@Document(collection = "addresses")
public class Address {
    @Id private String id;
    private String street;
    
    // Referência ao ID do cliente
    private String clientId;
}
```
- **Vantagem**: Melhor para dados que crescem indefinidamente.
- **Desvantagem**: Requer duas consultas ou o uso de `$lookup` (o "JOIN" do MongoDB), que é mais lento.

**Dica para a Entrevista**: No NoSQL, a regra de ouro é: **"O que você lê junto, deve ser salvo junto"**. Se sempre que você carrega o cliente você precisa dos endereços, use a **Opção A (Embedding)**.

---

## 8. JSON de Saída (Controller)

Quando o seu Controller Java devolve o objeto `Client`, o JSON já vem completo com os endereços aninhados, sem você precisar fazer nenhum `JOIN` manual no código.

### Exemplo de Resposta da API (GET /clients/1):

```json
{
  "id": "64f1a2b3c4d5e6f7",
  "name": "João da Silva",
  "email": "joao@email.com",
  "tags": ["Premium", "Vip"],
  "addresses": [
    {
      "street": "Avenida Paulista, 1000",
      "city": "São Paulo",
      "zipCode": "01310-100"
    },
    {
      "street": "Rua das Flores, 50",
      "city": "Campinas",
      "zipCode": "13000-000"
    }
  ]
}
```

### Por que isso é bom?
- **Redução de Latência**: O front-end faz apenas uma requisição e já recebe toda a árvore de dados.
- **Simplicidade**: O seu DTO em Java reflete exatamente essa estrutura, e o Spring Boot (via Jackson) converte a lista de objetos automaticamente para esse array no JSON.
