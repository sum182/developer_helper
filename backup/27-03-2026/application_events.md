# Manual: Como Usar Spring Application Events

Guia rápido e prático para implementar eventos em novos serviços.

## 1. Criar o Evento (A Mensagem)
Crie uma classe que estenda `ApplicationEvent`. Ela deve carregar os dados que os outros componentes vão precisar.

```java
public class MyEvent extends ApplicationEvent {
  private final String data;

  public MyEvent(Object source, String data) {
    super(source);
    this.data = data;
  }
  // getter
}
```

## 2. Publicar o Evento (O Disparo)
No seu Service principal, use o `ApplicationEventPublisher`.

```java
@Service
public class MyService {
  private final ApplicationEventPublisher publisher;

  public void doSomething() {
    // ... lógica de negócio ...
    publisher.publishEvent(new MyEvent(this, "algum dado"));
  }
}
```

## 3. Escutar o Evento (A Reação)
Crie um Listener para reagir ao evento. Use `@TransactionalEventListener` se estiver trabalhando com banco de dados.

```java
@Component
public class MyListener {

  // Executa ANTES do commit. Ideal para salvar auditoria ou Outbox.
  @TransactionalEventListener(phase = TransactionPhase.BEFORE_COMMIT)
  public void onBeforeCommit(MyEvent event) {
    System.out.println("Salvando no banco antes de finalizar a transação: " + event.getData());
  }

  // Executa DEPOIS do commit. Ideal para enviar emails ou RabbitMQ.
  @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
  public void onAfterCommit(MyEvent event) {
    System.out.println("Ação externa após sucesso no banco: " + event.getData());
  }
}
```

## Dicas Rápidas:
- **Desacoplamento**: O `MyService` não sabe que o `MyListener` existe.
- **Transação**: Se o `MyService` falhar, o Listener `BEFORE_COMMIT` também não salva nada.
- **Assincronismo**: Por padrão, os eventos são síncronos. Para rodar em background, use `@Async` no Listener.
