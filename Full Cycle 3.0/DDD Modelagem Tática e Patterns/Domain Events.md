"Use um evento de domínio para capturar uma ocorrência de algo que aconteceu no domínio"
"A essência de um evento de domínio é que você o usa para capturar coisas que podem desencadear uma mudança no estado do aplicativo que você está desenvolvendo. Esses objetos de evento são processados para causar alterações no sistema e armazenados para fornecer um AuditLog"

Todo evento deve ser representado em uma ação realizada no passado:

- UserCreated
- OrderPlaced
- EmailSent

Normalmente um Domain Event deve ser utilizado quando queremos notificar outros Bounded Contexts de uma mudança de estado.

### Componentes

- Event
- Handler: Executa o processamento quando um evento é chamado
- Event Dispatcher: Responsável por armazenar e executar os handlers de um evento quando ele for disparado.

### Dinâmica

- Criar um "Event Dispatcher"
- Criar um "Evento"
- Criar um "Handler" para o "Evento"
- Registrar o Evento, juntamente com o Handler no "Event Dispatcher"

Agora para disparar um evento, basta executar o método "notify" do "Event Dispatcher". Nesse momento todos os Handlers registrados no evento serão executados.


### Características importantes de Domain Events

- **Imutáveis**: Uma vez criados, Domain Events não devem ser modificados. Eles representam algo que já aconteceu.
- **Rico em contexto**: Devem conter todas as informações necessárias para seu processamento pelos handlers.
- **Identificáveis**: Normalmente possuem um identificador único e timestamp.
- **Históricos**: Podem ser armazenados permanentemente para reconstrução de estado ou auditoria.

### Benefícios do uso de Domain Events

- **Desacoplamento**: Reduz o acoplamento entre diferentes partes do sistema.
- **Extensibilidade**: Facilita adicionar novos comportamentos sem modificar código existente.
- **Rastreabilidade**: Facilita o entendimento de como o sistema chegou a determinado estado.
- **Auditoria**: Fornece um registro histórico de todas as mudanças significativas.

### Padrões relacionados

- **Event Sourcing**: Padrão onde o estado de um agregado é determinado pela sequência de eventos que o afetaram.
- **CQRS (Command Query Responsibility Segregation)**: Separação de modelos para leitura e escrita, frequentemente usado com eventos.
- **Saga Pattern**: Coordenação de transações distribuídas usando eventos.

Exemplos de Events

```java
public class OrderPlaced implements DomainEvent {
    private final UUID orderId;
    private final UUID customerId;
    private final List<OrderItem> items;
    private final BigDecimal totalAmount;
    private final LocalDateTime occurredOn;

    public OrderPlaced(UUID orderId, UUID customerId, List<OrderItem> items, BigDecimal totalAmount) {
        this.orderId = orderId;
        this.customerId = customerId;
        this.items = Collections.unmodifiableList(items);
        this.totalAmount = totalAmount;
        this.occurredOn = LocalDateTime.now();
    }

    // Getters (sem setters para garantir imutabilidade)
}
```

Exemplo de Handler

```java
public class InventoryHandler implements EventHandler<OrderPlaced> {
    private final InventoryService inventoryService;

    public InventoryHandler(InventoryService inventoryService) {
        this.inventoryService = inventoryService;
    }

    @Override
    public void handle(OrderPlaced event) {
        event.getItems().forEach(item -> 
            inventoryService.reduceStock(item.getProductId(), item.getQuantity())
        );
    }
}
```

