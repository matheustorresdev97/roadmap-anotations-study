## Entidade = IDENTIDADE

> "Uma entidade é algo único que é capaz de ser alterado de forma contínua durante um longo período de tempo."

> "Uma entidade é algo que possui uma continuidade em seu ciclo de vida e pode ser distinguida independente dos atributos que são importantes para a aplicação do usuário. Pode ser uma pessoa, cidade, carro, um ticket de loteria ou uma transação bancária"

### Diferenciando Entidades Anêmicas vs Ricas
Criando uma entidade anêmica
Uma entidade anêmica é essencialmente uma estrutura de dados sem comportamento significativo. Ela funciona como um contêiner passivo de dados com getters e setters, mas sem incorporar regras de negócio.

```java
public class Customer {
    private String id;
    private String name;
    private String address;
    private boolean active = false;

    public Customer(String id, String name, String address) {
        this.id = id;
        this.name = name;
        this.address = address;
    }

    // Getters e setters simples
    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public boolean isActive() {
        return active;
    }

    public void setActive(boolean active) {
        this.active = active;
    }
}
```

O problema com entidades anêmicas é que elas violam o princípio de encapsulamento. As regras de negócio acabam vivendo fora da entidade, geralmente em serviços, tornando a lógica de negócio fragmentada e difícil de manter.

### Entidade Rica

Uma entidade rica encapsula tanto dados quanto comportamento. Ela implementa regras de negócio e preserva invariantes de domínio.

```java
public class Customer {
    private final String id;
    private String name;
    private String address;
    private boolean active = false;

    public Customer(String id, String name, String address) {
        this.id = id;
        this.name = name;
        this.address = address;
        validate();
    }

    // Note a diferença semântica entre setName e changeName
    // setName sugere apenas definição de um valor
    // changeName implica uma ação de negócio com possíveis regras
    public void changeName(String name) {
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("Name cannot be empty");
        }
        this.name = name;
    }

    // O método activate incorpora regras de negócio
    public void activate() {
        if (this.address == null || this.address.trim().isEmpty()) {
            throw new IllegalArgumentException("Address is mandatory to activate a customer");
        }
        this.active = true;
    }

    public void deactivate() {
        this.active = false;
    }

    public void changeAddress(String address) {
        this.address = address;
        validate();
    }

    // Princípio da auto-validação
    private void validate() {
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("Name is required");
        }
        
        // Outras validações...
    }

    // Getters (note a ausência de setters genéricos)
    public String getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getAddress() {
        return address;
    }

    public boolean isActive() {
        return active;
    }

    // Métodos de negócio adicionais
    public boolean canPlaceOrders() {
        return active;
    }

    public void addCreditLimit(CreditLimit creditLimit) {
        // Lógica para adicionar limite de crédito
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Customer customer = (Customer) o;
        return Objects.equals(id, customer.id);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}
```


### Consistência e Auto-validação
 Consistência Constante em Primeiro Lugar

A consistência constante significa que a entidade nunca está em um estado inválido. Isso é garantido por:

1. **Validação no construtor** - Garantindo que a entidade nasce válida
2. **Métodos que encapsulam regras** - Em vez de setters que permitem qualquer valor
3. **Validação após mudanças** - Verificando se todas as regras ainda são atendidas

```Java
public class Order {
    private final String id;
    private final String customerId;
    private final List<OrderItem> items = new ArrayList<>();
    private OrderStatus status;

    public Order(String id, String customerId) {
        this.id = id;
        this.customerId = customerId;
        this.status = OrderStatus.DRAFT;
    }

    public void addItem(String productId, int quantity, double price) {
        if (status != OrderStatus.DRAFT) {
            throw new IllegalStateException("Cannot add items to order that is not in draft status");
        }
        
        if (quantity <= 0) {
            throw new IllegalArgumentException("Quantity must be positive");
        }
        
        // Verifica se o produto já existe no pedido
        for (OrderItem item : items) {
            if (item.getProductId().equals(productId)) {
                item.increaseQuantity(quantity);
                return;
            }
        }
        
        // Adiciona novo item
        items.add(new OrderItem(productId, quantity, price));
    }

    public void confirm() {
        if (items.isEmpty()) {
            throw new IllegalStateException("Order must have at least one item to be confirmed");
        }
        
        this.status = OrderStatus.CONFIRMED;
    }

    public double getTotal() {
        return items.stream()
                .mapToDouble(OrderItem::getSubtotal)
                .sum();
    }

    // Outros métodos de negócio...
}

class OrderItem {
    private final String productId;
    private int quantity;
    private final double price;

    public OrderItem(String productId, int quantity, double price) {
        this.productId = productId;
        this.quantity = quantity;
        this.price = price;
    }

    public void increaseQuantity(int additional) {
        if (additional <= 0) {
            throw new IllegalArgumentException("Additional quantity must be positive");
        }
        this.quantity += additional;
    }

    public double getSubtotal() {
        return quantity * price;
    }

    public String getProductId() {
        return productId;
    }

    public int getQuantity() {
        return quantity;
    }

    public double getPrice() {
        return price;
    }
}

enum OrderStatus {
    DRAFT, CONFIRMED, SHIPPED, DELIVERED, CANCELED
}
```

### Princípio da Auto-validação

O princípio da auto-validação determina que uma entidade deve sempre garantir sua própria consistência. Isso significa:

1. **Validar no construtor** - A entidade nasce válida ou lança exceção
2. **Validar após cada operação** - Ou durante a operação
3. **Encapsular regras de negócio** - Não permitir que regras sejam burladas

```java
public class Product {
    private final String id;
    private String name;
    private double price;
    private int stock;

    public Product(String id, String name, double price) {
        this.id = id;
        this.name = name;
        this.price = price;
        validate();
    }

    public void changeName(String name) {
        this.name = name;
        validate();
    }

    public void updatePrice(double price) {
        if (price < 0) {
            throw new IllegalArgumentException("Price cannot be negative");
        }
        this.price = price;
    }

    public void addToStock(int quantity) {
        if (quantity <= 0) {
            throw new IllegalArgumentException("Quantity must be positive");
        }
        this.stock += quantity;
    }

    public void removeFromStock(int quantity) {
        if (quantity <= 0) {
            throw new IllegalArgumentException("Quantity must be positive");
        }
        
        if (quantity > this.stock) {
            throw new IllegalStateException("Not enough items in stock");
        }
        
        this.stock -= quantity;
    }

    private void validate() {
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("Name cannot be empty");
        }
        
        if (price < 0) {
            throw new IllegalArgumentException("Price cannot be negative");
        }
    }

    // Getters
    public String getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public double getPrice() {
        return price;
    }

    public int getStock() {
        return stock;
    }

    public boolean isAvailable() {
        return stock > 0;
    }
}
```

### Entidade vs ORM: Separando Negócio de Persistência
Complexidade de Negócio vs Complexidade Acidental

Uma das questões mais importantes no DDD é a separação entre a complexidade inerente ao domínio (complexidade de negócio) e a complexidade relacionada à implementação técnica (complexidade acidental).

#### Abordagem: Duas Classes Distintas

Uma abordagem é ter duas classes separadas - uma para o domínio e outra para persistência.

**Estrutura de Pacotes:**

com.empresa.produto/
    ├── domain/                   # Complexidade de negócio
    │   └── entity/
    │       └── Customer.java     # Entidade rica com regras de negócio
    └── infrastructure/           # Complexidade acidental
        └── entity/
            └── CustomerModel.java # Classe ORM/persistência


Entidade de Domínio:
```java
// domain/entity/Customer.java
package com.empresa.produto.domain.entity;

public class Customer {
    private final String id;
    private String name;
    private String address;
    private boolean active = false;
    
    // Construtor, métodos de negócio, validações...
    // (Como no exemplo de entidade rica acima)
}
```

Modelo de Persistência:
```java
// infrastructure/entity/CustomerModel.java
package com.empresa.produto.infrastructure.entity;

import javax.persistence.*;

@Entity
@Table(name = "customers")
public class CustomerModel {
    @Id
    private String id;
    
    @Column(nullable = false)
    private String name;
    
    private String address;
    
    private boolean active;

    // Getters e setters para ORM
    
    // Métodos de conversão
    public static CustomerModel fromDomain(Customer customer) {
        CustomerModel model = new CustomerModel();
        model.setId(customer.getId());
        model.setName(customer.getName());
        model.setAddress(customer.getAddress());
        model.setActive(customer.isActive());
        return model;
    }
    
    public Customer toDomain() {
        Customer customer = new Customer(id, name, address);
        if (active) {
            customer.activate();
        }
        return customer;
    }
}
```

Repositório:

```java
// infrastructure/repository/CustomerRepositoryImpl.java
package com.empresa.produto.infrastructure.repository;

import com.empresa.produto.domain.entity.Customer;
import com.empresa.produto.domain.repository.CustomerRepository;
import com.empresa.produto.infrastructure.entity.CustomerModel;
import org.springframework.stereotype.Repository;

@Repository
public class CustomerRepositoryImpl implements CustomerRepository {
    
    private final CustomerJpaRepository jpaRepository;
    
    public CustomerRepositoryImpl(CustomerJpaRepository jpaRepository) {
        this.jpaRepository = jpaRepository;
    }
    
    @Override
    public void save(Customer customer) {
        CustomerModel model = CustomerModel.fromDomain(customer);
        jpaRepository.save(model);
    }
    
    @Override
    public Customer findById(String id) {
        return jpaRepository.findById(id)
            .map(CustomerModel::toDomain)
            .orElse(null);
    }
    
    // Outros métodos...
}

// CustomerJpaRepository é uma interface Spring Data JPA
interface CustomerJpaRepository extends JpaRepository<CustomerModel, String> {
    // Queries específicas...
}
```

Abordagem: Classe Única com Anotações JPA
Outra abordagem é usar uma única classe que serve tanto para o domínio quanto para persistência.

```java
package com.empresa.produto.domain.entity;

import javax.persistence.*;

@Entity
@Table(name = "customers")
public class Customer {
    @Id
    private String id;
    
    @Column(nullable = false)
    private String name;
    
    private String address;
    
    private boolean active = false;

    // Construtor protegido para JPA
    protected Customer() {}
    
    // Construtor principal para uso no domínio
    public Customer(String id, String name, String address) {
        this.id = id;
        this.name = name;
        this.address = address;
        validate();
    }
    
    // Métodos de negócio
    public void activate() {
        if (address == null || address.trim().isEmpty()) {
            throw new IllegalArgumentException("Address is mandatory to activate a customer");
        }
        this.active = true;
    }
    
    public void deactivate() {
        this.active = false;
    }
    
    public void changeName(String name) {
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("Name cannot be empty");
        }
        this.name = name;
    }
    
    private void validate() {
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("Name is required");
        }
    }
    
    // Getters
    public String getId() {
        return id;
    }
    
    public String getName() {
        return name;
    }
    
    public String getAddress() {
        return address;
    }
    
    public boolean isActive() {
        return active;
    }
}
```

### Prós e Contras

#### Duas Classes (Separação Completa)

**Prós:**

- Domínio puro, sem poluição por detalhes de infraestrutura
- Mais fácil de testar a lógica de domínio isoladamente
- Flexibilidade para mudar a tecnologia de persistência

**Contras:**

- Código duplicado (duas classes semelhantes)
- Necessidade de mapear entre as classes
- Overhead de manutenção

#### Classe Única

**Prós:**

- Menos código e mapeamentos
- Mais simples de implementar inicialmente

**Contras:**

- Domínio contaminado por detalhes de infraestrutura (anotações JPA)
- Possível necessidade de comprometer o design do domínio para atender requisitos da persistência
- Mais difícil de testar isoladamente

Exemplo Completo de Implementação: Entidade Rica de Pedido

```java
public class Order {
    private final OrderId id;
    private final CustomerId customerId;
    private List<OrderItem> items = new ArrayList<>();
    private OrderStatus status;
    private ShippingAddress shippingAddress;
    private PaymentInfo paymentInfo;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;

    // Construtor para criação de novo pedido
    public Order(OrderId id, CustomerId customerId) {
        this.id = id;
        this.customerId = customerId;
        this.status = OrderStatus.DRAFT;
        this.createdAt = LocalDateTime.now();
        this.updatedAt = LocalDateTime.now();
    }

    // Adicionar item ao pedido
    public void addItem(Product product, int quantity) {
        validateDraftStatus();
        
        if (quantity <= 0) {
            throw new IllegalArgumentException("Quantity must be greater than zero");
        }
        
        // Verifica se o produto já existe
        for (OrderItem item : items) {
            if (item.getProductId().equals(product.getId())) {
                item.increaseQuantity(quantity);
                this.updatedAt = LocalDateTime.now();
                return;
            }
        }
        
        // Adiciona novo item
        OrderItem newItem = new OrderItem(
            new OrderItemId(UUID.randomUUID().toString()),
            product.getId(),
            product.getName(),
            product.getPrice(),
            quantity
        );
        
        items.add(newItem);
        this.updatedAt = LocalDateTime.now();
    }

    // Remover item do pedido
    public void removeItem(OrderItemId orderItemId) {
        validateDraftStatus();
        
        boolean removed = items.removeIf(item -> item.getId().equals(orderItemId));
        
        if (!removed) {
            throw new IllegalArgumentException("Item not found in order");
        }
        
        this.updatedAt = LocalDateTime.now();
    }

    // Atualizar quantidade de um item
    public void updateItemQuantity(OrderItemId orderItemId, int newQuantity) {
        validateDraftStatus();
        
        if (newQuantity <= 0) {
            throw new IllegalArgumentException("Quantity must be greater than zero");
        }
        
        OrderItem item = findItem(orderItemId);
        item.updateQuantity(newQuantity);
        this.updatedAt = LocalDateTime.now();
    }

    // Definir endereço de entrega
    public void setShippingAddress(ShippingAddress shippingAddress) {
        validateDraftStatus();
        
        if (shippingAddress == null) {
            throw new IllegalArgumentException("Shipping address cannot be null");
        }
        
        this.shippingAddress = shippingAddress;
        this.updatedAt = LocalDateTime.now();
    }

    // Definir informações de pagamento
    public void setPaymentInfo(PaymentInfo paymentInfo) {
        validateDraftStatus();
        
        if (paymentInfo == null) {
            throw new IllegalArgumentException("Payment info cannot be null");
        }
        
        this.paymentInfo = paymentInfo;
        this.updatedAt = LocalDateTime.now();
    }

    // Confirmar o pedido
    public void confirm() {
        validateDraftStatus();
        
        if (items.isEmpty()) {
            throw new IllegalStateException("Cannot confirm an order without items");
        }
        
        if (shippingAddress == null) {
            throw new IllegalStateException("Cannot confirm an order without shipping address");
        }
        
        if (paymentInfo == null) {
            throw new IllegalStateException("Cannot confirm an order without payment information");
        }
        
        this.status = OrderStatus.CONFIRMED;
        this.updatedAt = LocalDateTime.now();
    }

    // Processar o pedido (após pagamento confirmado)
    public void process() {
        if (status != OrderStatus.CONFIRMED) {
            throw new IllegalStateException("Only confirmed orders can be processed");
        }
        
        this.status = OrderStatus.PROCESSING;
        this.updatedAt = LocalDateTime.now();
    }

    // Marcar como enviado
    public void ship(String trackingCode) {
        if (status != OrderStatus.PROCESSING) {
            throw new IllegalStateException("Only processing orders can be shipped");
        }
        
        if (trackingCode == null || trackingCode.trim().isEmpty()) {
            throw new IllegalArgumentException("Tracking code is required");
        }
        
        this.status = OrderStatus.SHIPPED;
        this.updatedAt = LocalDateTime.now();
    }

    // Marcar como entregue
    public void deliver() {
        if (status != OrderStatus.SHIPPED) {
            throw new IllegalStateException("Only shipped orders can be delivered");
        }
        
        this.status = OrderStatus.DELIVERED;
        this.updatedAt = LocalDateTime.now();
    }

    // Cancelar pedido
    public void cancel() {
        if (status == OrderStatus.DELIVERED) {
            throw new IllegalStateException("Delivered orders cannot be canceled");
        }
        
        this.status = OrderStatus.CANCELLED;
        this.updatedAt = LocalDateTime.now();
    }

    // Calcular valor total do pedido
    public Money calculateTotal() {
        return items.stream()
                .map(OrderItem::getSubtotal)
                .reduce(Money.ZERO, Money::add);
    }

    // Métodos auxiliares
    private void validateDraftStatus() {
        if (status != OrderStatus.DRAFT) {
            throw new IllegalStateException("Order is not in draft status");
        }
    }

    private OrderItem findItem(OrderItemId orderItemId) {
        return items.stream()
                .filter(item -> item.getId().equals(orderItemId))
                .findFirst()
                .orElseThrow(() -> new IllegalArgumentException("Item not found in order"));
    }

    // Getters
    public OrderId getId() {
        return id;
    }

    public CustomerId getCustomerId() {
        return customerId;
    }

    public OrderStatus getStatus() {
        return status;
    }

    public List<OrderItem> getItems() {
        return Collections.unmodifiableList(items);
    }

    public ShippingAddress getShippingAddress() {
        return shippingAddress;
    }

    public PaymentInfo getPaymentInfo() {
        return paymentInfo;
    }

    public LocalDateTime getCreatedAt() {
        return createdAt;
    }

    public LocalDateTime getUpdatedAt() {
        return updatedAt;
    }

    // Value objects e identificadores
    
    public static class OrderId {
        private final String value;
        
        public OrderId(String value) {
            if (value == null || value.trim().isEmpty()) {
                throw new IllegalArgumentException("Order ID cannot be empty");
            }
            this.value = value;
        }
        
        public String getValue() {
            return value;
        }
        
        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            OrderId orderId = (OrderId) o;
            return value.equals(orderId.value);
        }
        
        @Override
        public int hashCode() {
            return Objects.hash(value);
        }
    }
    
    public enum OrderStatus {
        DRAFT, CONFIRMED, PROCESSING, SHIPPED, DELIVERED, CANCELLED
    }
}

// Item do pedido - Entidade dentro do Agregado Order
class OrderItem {
    private final OrderItemId id;
    private final ProductId productId;
    private final String productName;
    private final Money unitPrice;
    private int quantity;

    public OrderItem(OrderItemId id, ProductId productId, String productName, Money unitPrice, int quantity) {
        this.id = id;
        this.productId = productId;
        this.productName = productName;
        this.unitPrice = unitPrice;
        setQuantity(quantity);
    }

    public void increaseQuantity(int additionalQuantity) {
        if (additionalQuantity <= 0) {
            throw new IllegalArgumentException("Additional quantity must be greater than zero");
        }
        this.quantity += additionalQuantity;
    }

    public void updateQuantity(int newQuantity) {
        setQuantity(newQuantity);
    }

    private void setQuantity(int quantity) {
        if (quantity <= 0) {
            throw new IllegalArgumentException("Quantity must be greater than zero");
        }
        this.quantity = quantity;
    }

    public Money getSubtotal() {
        return unitPrice.multiply(quantity);
    }

    // Getters
    public OrderItemId getId() {
        return id;
    }

    public ProductId getProductId() {
        return productId;
    }

    public String getProductName() {
        return productName;
    }

    public Money getUnitPrice() {
        return unitPrice;
    }

    public int getQuantity() {
        return quantity;
    }
}

// Identificador para item de pedido
class OrderItemId {
    private final String value;
    
    public OrderItemId(String value) {
        if (value == null || value.trim().isEmpty()) {
            throw new IllegalArgumentException("Order item ID cannot be empty");
        }
        this.value = value;
    }
    
    public String getValue() {
        return value;
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        OrderItemId that = (OrderItemId) o;
        return value.equals(that.value);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(value);
    }
}

// Value Object para representar dinheiro
class Money {
    private final BigDecimal amount;
    private final String currency;
    
    public static final Money ZERO = new Money(BigDecimal.ZERO, "USD");
    
    public Money(BigDecimal amount, String currency) {
        this.amount = amount.setScale(2, RoundingMode.HALF_EVEN);
        this.currency = currency;
    }
    
    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Cannot add money with different currencies");
        }
        return new Money(this.amount.add(other.amount), this.currency);
    }
    
    public Money subtract(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Cannot subtract money with different currencies");
        }
        return new Money(this.amount.subtract(other.amount), this.currency);
    }
    
    public Money multiply(int multiplier) {
        return new Money(this.amount.multiply(new BigDecimal(multiplier)), this.currency);
    }
    
    // Getters e métodos de comparação...
}
```

Uma entidade rica no DDD é muito mais do que um simples contêiner de dados. Ela:

1. **Encapsula regras de negócio** - O comportamento está dentro da entidade, não espalhado em serviços
2. **Garante sua própria consistência** - Através de validações e invariantes
3. **Expressa o modelo mental do domínio** - Usando uma linguagem próxima do negócio
4. **Protege seu estado interno** - Através de métodos que expressam intenções

A separação clara entre a complexidade de negócio (domínio) e a complexidade acidental (infraestrutura/persistência) é fundamental para manter o modelo de domínio limpo e expressivo.

Ao implementar entidades ricas, estamos colocando o domínio no centro do design, conforme preconiza o DDD, e criando um código que reflete de forma mais fiel o mundo real que estamos modelando.