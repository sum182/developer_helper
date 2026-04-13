# Manual: Spring Batch (Processamento de Dados em Massa)

Este guia explica o **Spring Batch**, um framework robusto para processamento de grandes volumes de dados, essencial para sistemas financeiros (processamento de arquivos de retorno, faturas, folhas de pagamento).

---

## 1. O que é Spring Batch?

É um framework projetado para processar milhões de registros de forma eficiente, confiável e com alta performance. Ele resolve problemas comuns de processamento em massa, como:
- **Reinicialização**: Se o processo cair no meio, ele sabe onde parou.
- **Transacionalidade**: Garante que os dados não fiquem inconsistentes.
- **Estatísticas**: Logs detalhados de quantos registros foram processados/falharam.

---

## 2. Arquitetura Básica (Os 3 Pilares)

O fluxo de um passo (**Step**) do Spring Batch é baseado no modelo **ETL** (Extract, Transform, Load):

1.  **ItemReader**: Responsável por ler os dados de uma fonte (Arquivo CSV, Banco de Dados, API).
2.  **ItemProcessor**: Responsável pela lógica de negócio (Validar dados, formatar strings, calcular taxas). *Opcional.*
3.  **ItemWriter**: Responsável por salvar os dados transformados em um destino (Outro banco, enviar p/ RabbitMQ, gerar arquivo).

---

## 3. Conceitos Fundamentais para a Entrevista

- **Job**: É o processo completo (ex: "Processar Faturas de Fevereiro").
- **Step**: É uma fase dentro do Job. Um Job pode ter vários Steps (ex: Step 1: Limpar tabela; Step 2: Ler arquivo e Salvar).
- **Chunk (Pedaço)**: É a técnica de processar registros em blocos (ex: de 100 em 100). Isso evita estourar a memória e torna o banco mais rápido.
- **JobRepository**: É onde o Spring Batch guarda o histórico de execução (se o Job teve sucesso, falha ou está pendente).

---

## 4. Exemplo Prático em Java

### Configuração do Step:
```java
@Bean
public Step myStep(ItemReader<User> reader, ItemProcessor<User, User> processor, ItemWriter<User> writer) {
    return stepBuilderFactory.get("myStep")
        .<User, User>chunk(100) // Processa em blocos de 100
        .reader(reader)
        .processor(processor)
        .writer(writer)
        .build();
}
```

### Exemplo de Processor (Lógica de Negócio):
```java
public class UserProcessor implements ItemProcessor<User, User> {
    @Override
    public User process(User user) {
        user.setName(user.getName().toUpperCase()); // Exemplo simples de transformação
        return user;
    }
}
```

---

## 5. Quando usar Spring Batch vs @Scheduled simples?

| Característica | @Scheduled Simples | Spring Batch |
| :--- | :--- | :--- |
| **Volume** | Poucos registros. | Milhões de registros. |
| **Recuperação** | Se falhar, começa do zero. | Se falhar, continua exatamente de onde parou. |
| **Controle** | Difícil de monitorar progresso. | Dashboard nativo e logs de execução. |
| **Chunking** | Tudo ou nada. | Commit a cada bloco processado. |

**Conclusão**: Use Spring Batch sempre que o tempo de processamento for longo ou o volume de dados for grande o suficiente para não caber na memória de uma vez.
