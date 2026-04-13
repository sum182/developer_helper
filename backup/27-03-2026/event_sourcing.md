# Manual: O Padrão Event Sourcing

Este guia explica o padrão **Event Sourcing**, uma forma de persistência de dados focada na história completa dos fatos, e não apenas no estado final.

---

## 1. O que é Event Sourcing?

Tradicionalmente, os sistemas salvam apenas o "Estado Atual" de um dado. No **Event Sourcing**, nós salvamos a sequência de todos os **Eventos Mutáveis** que ocorreram.

O estado atual do sistema é obtido através do **reprocessamento (replay)** desses eventos.

---

## 2. O Exemplo Clássico: Saldo Bancário

### Abordagem de Estado Atual (CRUD)
Se você faz um depósito, o sistema executa: `UPDATE saldo = saldo + 100`.
- **Problema**: Se o valor final estiver errado, você não sabe qual transação causou o erro.

### Abordagem Event Sourcing
O sistema grava cada fato individualmente:
1.  `ContaAbertaEvent` (Data: 01/01)
2.  `DepositoRealizadoEvent` (Valor: 500)
3.  `SaqueRealizadoEvent` (Valor: 200)
4.  `RendimentoAplicadoEvent` (Valor: 10)

**Resultado**: Para saber o saldo hoje, o sistema soma os eventos: `0 + 500 - 200 + 10 = 310`.

---

## 3. Vantagens do Event Sourcing

1.  **Auditoria Nativa**: O banco de dados de eventos é um log de auditoria perfeito por natureza. Você sabe QUEM, QUANDO e O QUE mudou.
2.  **Viagem no Tempo**: Você pode reconstruir o estado de um objeto em qualquer ponto do passado. *"Qual era o saldo deste cliente em 15 de fevereiro?"*.
3.  **Depuração (Debug)**: Se houver um bug no código que calcula o saldo, os dados (eventos) ainda estão corretos. Você corrige o código e reprocessa os eventos para ter o saldo certo.

---

## 4. Quando NÃO usar?

Event Sourcing traz complexidade. Não deve ser usado se:
- O sistema é um CRUD simples.
- Você precisa de performance extrema em leituras complexas (para isso usa-se o padrão **CQRS** em conjunto).
- Você não tem uma equipe familiarizada com processamento assíncrono.

---

## 5. Resumo Tático
- **Estado**: É passageiro e pode ser reconstruído.
- **Evento**: É a fonte da verdade e é imutável.
- **Projeção**: É o nome dado ao processo de transformar eventos em um saldo ou perfil legível.

---

## 6. Nós deletamos dados no Event Sourcing?

A resposta curta é **NÃO**.

No Event Sourcing, os dados são **Imutáveis**. Um fato que aconteceu no passado não pode ser "apagado" da história.

### Como lidar com erros ou exclusões?

1. **Eventos de Compensação**: Se você inseriu um valor errado, você não apaga o evento. Você cria um **novo evento** corrigindo o anterior.
   - *Exemplo*: Se você depositou R$ 100 por erro, você cria um evento `EstornoDeDeposito` de R$ 100. A história mostra o erro e a correção.

2. **Evento de Deleção Logica**: Se um usuário quer "deletar" uma conta, nós gravamos o evento `ContaEncerradaEvent`.
   - Os eventos anteriores continuam lá, mas a **Projeção** (o estado atual) dirá que a conta está inativa e não deve ser exibida.

### Por que não deletar?
Se você deletar um evento da história, você quebra a integridade do sistema. Você perde a capacidade de reconstruir o estado passado e perde a trilha de auditoria, que é a principal vantagem deste padrão.
