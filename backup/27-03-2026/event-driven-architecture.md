# Manual: Event-Driven Architecture (EDA)

Este guia explica os conceitos fundamentais da Orientação a Eventos, essencial para entender como sistemas modernos e escaláveis se comunicam.

---

## 1. O que é Event-Driven Architecture (EDA)?

A **Event-Driven Architecture (EDA)** é um padrão onde o fluxo do software é acionado por fatos significativos chamados **Eventos**. 

Um evento é algo que **já aconteceu** no passado e não pode ser alterado.
*   *Exemplos*: "Empresa Cadastrada", "Pedido Pago", "Sensor Detectou Movimento".

---

## 2. Os Três Pilares da Comunicação

Para que a EDA funcione, precisamos de três personagens principais:

1.  **Produtor (Emitter)**: A aplicação que realiza uma ação e "anuncia" que ela ocorreu. Ela não sabe quem vai receber essa informação.
    *   *Exemplo*: O sistema de cadastro de empresas avisa: "Criei a empresa X".
2.  **Consumidor (Subscriber)**: Qualquer aplicação interessada naquele aviso. Ela reage ao evento executando sua própria lógica.
    *   *Exemplo*: O sistema de usuários recebe o aviso e cria um acesso para o administrador daquela nova empresa.
3.  **Broker (O Mensageiro)**: O software que transporta a mensagem. Ele garante que o aviso saia do produtor e chegue aos consumidores, mesmo que eles estejam temporariamente offline.
    *   *Exemplos*: **RabbitMQ**, Apache Kafka, AWS SNS/SQS.

---

## 3. Por que usar Eventos em vez de chamadas diretas (REST/HTTP)?

| Característica | Chamada Direta (Request-Response) | Orientação a Eventos (EDA) |
| :--- | :--- | :--- |
| **Dependência** | Se o sistema destino estiver fora, a ação principal falha. | Se o destino estiver fora, o Broker guarda a mensagem até ele voltar. |
| **Performance** | O usuário espera todas as etapas terminarem para ter o "OK". | O usuário recebe o "OK" assim que a ação principal ocorre. |
| **Flexibilidade** | Para adicionar um novo passo, você precisa alterar o código original. | Basta criar um novo "ouvinte" para o evento, sem tocar no código original. |

---

## 4. Resumo para Implementação

Ao desenhar um sistema orientado a eventos, lembre-se:
*   **Eventos são fatos passados**: Sempre use nomes no passado (`Created`, `Updated`, `Failed`).
*   **Desacoplamento é a meta**: O sistema que envia nunca deve depender da saúde ou da velocidade do sistema que recebe.
*   **Broker é o seguro**: O RabbitMQ ou Kafka é quem garante que nenhum evento se perca no caminho.
