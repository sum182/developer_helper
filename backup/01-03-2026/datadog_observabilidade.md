# Manual: Observabilidade Sênior com Datadog

Este guia explica como utilizar o **Datadog** para monitorar sistemas distribuídos, focando em garantir que cada transação financeira seja rastreável de ponta a ponta.

---

## 1. O que é Observabilidade?
Não é apenas saber "se o servidor está vivo" (Monitoramento), mas sim entender **"por que o sistema está lento ou falhando"**. O Datadog baseia-se em três pilares: **Logs, Metrics e Traces**.

---

## 2. APM e Distributed Tracing (O mais importante)

Em um sistema de faturas, uma única ação passa por vários serviços.
- **TracerId (ou SpanId)**: O Datadog gera um ID único que viaja com a mensagem.
- **Visibilidade**: Se um e-mail não chegou, você olha o ID no Datadog e ele te mostra:
  1. Onde começou (Cron).
  2. Quanto tempo levou no Batch.
  3. Se houve erro no processador de PDF.
  4. Exatamente em qual linha de código a falha ocorreu.

---

## 3. Logs Estruturados
Em vez de logs simples em texto, enviamos logs em **JSON** para o Datadog.
- **Vantagem**: Você pode filtrar no painel do Datadog por campos como `customer_id`, `invoice_value` ou `error_type`.
- **Alertas**: Você pode criar um monitor que avisa no Slack: *"Atenção: A taxa de erro na geração de PDFs subiu 5% nos últimos 10 minutos!"*.

---

## 4. Dashboards de Negócio (Real-time)
O Datadog não serve apenas para erros técnicos. Você pode montar gráficos que mostram a saúde do negócio:
- Quantas faturas foram geradas hoje?
- Qual o valor total processado nos últimos 60 minutos?
- Qual a latência média (tempo de resposta) entre a geração do lote e o envio do e-mail?

---

## 5. Por que monitorar TODOS os serviços?

Em arquiteturas orientadas a eventos (EDA), o erro pode estar escondido no "silêncio".
- **Monitoramento de Filas**: Se o serviço de PDF está OK, mas a fila de faturas está crescendo sem parar, o Datadog te avisa que você precisa subir mais instâncias (Auto-scaling).
- **Rastreio de Banco de Dados**: Ele mostra quais queries SQL ou NoSQL estão lentas e travando o processamento em massa (Spring Batch).

---

## 6. O que é o Trace ID (ou Correlation ID)?

O **Trace ID** é o "RG" de uma requisição. Ele é o coração do monitoramento em sistemas distribuídos.

### Como ele funciona?
1.  **Nascimento**: Quando o Job inicial gera uma fatura, o sistema cria um ID único (ex: `uuid-12345`).
2.  **Propagação**: Esse ID viaja **dentro** da mensagem do RabbitMQ e nos headers das chamadas HTTP.
3.  **Registro**: Cada microserviço (Batch, PDF, Email) inclui esse `trace_id` em todos os seus logs.

### Por que ele é vital em faturamento?
Imagine que o cliente reclama: *"Não recebi minha fatura"*.
- **Sem Trace ID**: Você teria que procurar nos logs de 3 sistemas diferentes por horários aproximados. Uma agulha no palheiro.
- **Com Trace ID**: Você digita `uuid-12345` no Datadog e ele te mostra a "árvore" completa:
    *   10:00:00 - Gerado pelo Batch (Sucesso)
    *   10:00:05 - Processado pelo Serviço PDF (Sucesso)
    *   10:00:10 - Tentativa de envio de E-mail (**FALHA: SMTP Offline**)

### Conclusão:
O Trace ID transforma logs isolados em uma **história contínua**. Ele permite que você encontre a causa raiz de um erro em segundos, mesmo em um mar de 5 milhões de registros.

---

## 7. Dica para a Entrevista
Se te perguntarem sobre monitoramento, responda:
> *"Minha estratégia de observabilidade utiliza o **APM do Datadog** com **Distributed Tracing**. Isso garante que cada evento tenha um rastro único, permitindo identificar gargalos de performance e falhas silenciosas entre microserviços de forma visual e em tempo real."*
