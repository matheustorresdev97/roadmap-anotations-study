"Um agregado é um conjunto de objetos associados que tratamos como uma unidade para propósito de mudança de dados"
Os agregados são um padrão essencial no Domain-Driven Design (DDD) que organizam objetos relacionados em um cluster coeso, tratado como uma única unidade.
## Benefícios:

- **Segurança** - Por serem imutáveis, são thread-safe e previnem estados inconsistentes.
- **Expressividade** - Capturam conceitos do domínio de forma clara e coesa.
- **Simplicidade** - Facilitam o raciocínio sobre o código por não terem efeitos colaterais.

Use objetos de valor sempre que estiver modelando conceitos onde a identidade não importa, apenas os atributos e suas relações.

![[Captura de tela de 2025-04-03 10-11-31.png]]


Exemplo: Pedido (Order)
Um pedido pode ser modelado como um agregado:
```java
// Raiz do Agregado
public class Order {
    private OrderId id;
    private CustomerId customerId;
    private List<OrderItem> items;
    private ShippingAddress shippingAddress;
    private OrderStatus status;
    
    // Construtor e métodos de validação
    
    // Métodos que modificam o estado interno do agregado
    public void addItem(Product product, int quantity) {
        validateProductCanBeAdded(product, quantity);
        items.add(new OrderItem(product.getId(), quantity, product.getPrice()));
        recalculateTotal();
    }
    
    public void changeShippingAddress(ShippingAddress newAddress) {
        if (status != OrderStatus.CREATED) {
            throw new IllegalStateException("Cannot change address of processed order");
        }
        this.shippingAddress = newAddress;
    }
    
    // Entidades e objetos de valor dentro do agregado são 
    // acessados apenas através da raiz
}

// Entidade interna ao agregado - identidade local
public class OrderItem {
    private ProductId productId;
    private int quantity;
    private Money unitPrice;
    
    // Construtores e métodos
}

// Value Object dentro do agregado
public class ShippingAddress {
    // Implementação como Value Object
}
```

### Regras Práticas para Definir Agregados

1. **Modelar Agregados Pequenos** - Prefira agregados menores, focados em uma única responsabilidade.
2. **Referências entre Agregados** - Use referências por ID (não por objeto) entre diferentes agregados.
3. **Invariantes de Negócio** - Um agregado encapsula e protege invariantes de negócio relacionados.
4. **Transações** - Uma transação deve modificar apenas um agregado (regra de ouro).

### Benefícios

- **Consistência** - Garante que as regras de negócio sejam sempre respeitadas.
- **Encapsulamento** - Esconde detalhes internos de implementação.
- **Desempenho** - Facilita operações de persistência e concorrência.
- **Escalabilidade** - Permite distribuir agregados diferentes em múltiplos serviços/bancos.