Aplicar testes em componentes táticos do DDD usando Java e TypeScript, focando em estratégias de teste para garantir que seus padrões táticos (Value Objects, Entidades, Agregados, etc.) funcionem corretamente.

### Testando Value Objects

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class EmailTest {

    @Test
    public void shouldCreateValidEmail() {
        // Arrange & Act
        Email email = new Email("user@example.com");
        
        // Assert
        assertEquals("user@example.com", email.getValue());
    }
    
    @Test
    public void shouldThrowExceptionForInvalidEmail() {
        // Arrange & Act & Assert
        assertThrows(IllegalArgumentException.class, () -> {
            new Email("invalid-email");
        });
    }
    
    @Test
    public void twoEmailsWithSameValueShouldBeEqual() {
        // Arrange
        Email email1 = new Email("user@example.com");
        Email email2 = new Email("user@example.com");
        
        // Act & Assert
        assertEquals(email1, email2);
        assertEquals(email1.hashCode(), email2.hashCode());
    }
    
    @Test
    public void emailShouldBeImmutable() {
        // Arrange
        Email email = new Email("user@example.com");
        
        // Act
        Email newEmail = email.withDomain("new-domain.com");
        
        // Assert
        assertEquals("user@example.com", email.getValue());
        assertEquals("user@new-domain.com", newEmail.getValue());
        assertNotSame(email, newEmail);
    }
}

// Classe Value Object que está sendo testada
class Email {
    private final String value;
    
    public Email(String value) {
        if (value == null || !value.matches("^[\\w-\\.]+@([\\w-]+\\.)+[\\w-]{2,4}$")) {
            throw new IllegalArgumentException("Invalid email format");
        }
        this.value = value;
    }
    
    public String getValue() {
        return value;
    }
    
    public Email withDomain(String domain) {
        String user = value.split("@")[0];
        return new Email(user + "@" + domain);
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Email email = (Email) o;
        return value.equals(email.value);
    }
    
    @Override
    public int hashCode() {
        return value.hashCode();
    }
}
```

```typescript
import { expect } from 'chai';

// Value Object
class Money {
  private readonly amount: number;
  private readonly currency: string;

  constructor(amount: number, currency: string) {
    if (amount < 0) {
      throw new Error('Amount cannot be negative');
    }
    
    if (!currency || currency.trim().length !== 3) {
      throw new Error('Currency must be a 3-letter ISO code');
    }
    
    this.amount = amount;
    this.currency = currency.toUpperCase();
  }

  public getAmount(): number {
    return this.amount;
  }

  public getCurrency(): string {
    return this.currency;
  }

  public add(money: Money): Money {
    if (this.currency !== money.currency) {
      throw new Error('Cannot add money with different currencies');
    }
    
    return new Money(this.amount + money.amount, this.currency);
  }

  public equals(other: Money): boolean {
    if (!(other instanceof Money)) return false;
    return this.amount === other.amount && this.currency === other.currency;
  }
}

// Testes
describe('Money Value Object', () => {
  it('should create a valid money object', () => {
    const money = new Money(100, 'USD');
    expect(money.getAmount()).to.equal(100);
    expect(money.getCurrency()).to.equal('USD');
  });

  it('should reject negative amounts', () => {
    expect(() => new Money(-50, 'USD')).to.throw('Amount cannot be negative');
  });

  it('should validate currency format', () => {
    expect(() => new Money(100, 'US')).to.throw('Currency must be a 3-letter ISO code');
    expect(() => new Money(100, 'USDD')).to.throw('Currency must be a 3-letter ISO code');
  });

  it('should add money of same currency', () => {
    const money1 = new Money(100, 'USD');
    const money2 = new Money(50, 'USD');
    const result = money1.add(money2);
    
    expect(result.getAmount()).to.equal(150);
    expect(result.getCurrency()).to.equal('USD');
    // Original objects should be unchanged (immutability)
    expect(money1.getAmount()).to.equal(100);
    expect(money2.getAmount()).to.equal(50);
  });

  it('should reject adding different currencies', () => {
    const money1 = new Money(100, 'USD');
    const money2 = new Money(50, 'EUR');
    
    expect(() => money1.add(money2)).to.throw('Cannot add money with different currencies');
  });

  it('should compare money objects correctly', () => {
    const money1 = new Money(100, 'USD');
    const money2 = new Money(100, 'USD');
    const money3 = new Money(200, 'USD');
    const money4 = new Money(100, 'EUR');
    
    expect(money1.equals(money2)).to.be.true;
    expect(money1.equals(money3)).to.be.false;
    expect(money1.equals(money4)).to.be.false;
  });
});
```

### Testando Agregados
```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

import java.util.ArrayList;
import java.util.List;
import java.util.UUID;

public class OrderAggregateTest {

    @Test
    public void shouldCreateOrderWithValidData() {
        // Arrange
        CustomerId customerId = new CustomerId(UUID.randomUUID());
        
        // Act
        Order order = new Order(customerId);
        
        // Assert
        assertEquals(customerId, order.getCustomerId());
        assertEquals(OrderStatus.CREATED, order.getStatus());
        assertTrue(order.getItems().isEmpty());
    }
    
    @Test
    public void shouldAddItemToOrder() {
        // Arrange
        Order order = new Order(new CustomerId(UUID.randomUUID()));
        ProductId productId = new ProductId(UUID.randomUUID());
        Money price = new Money(100, "USD");
        
        // Act
        order.addItem(productId, 2, price);
        
        // Assert
        assertEquals(1, order.getItems().size());
        assertEquals(productId, order.getItems().get(0).getProductId());
        assertEquals(2, order.getItems().get(0).getQuantity());
        assertEquals(new Money(200, "USD"), order.getTotalAmount());
    }
    
    @Test
    public void shouldNotAddItemToSubmittedOrder() {
        // Arrange
        Order order = new Order(new CustomerId(UUID.randomUUID()));
        order.submit();
        
        ProductId productId = new ProductId(UUID.randomUUID());
        Money price = new Money(100, "USD");
        
        // Act & Assert
        assertThrows(IllegalStateException.class, () -> {
            order.addItem(productId, 2, price);
        });
    }
    
    @Test
    public void shouldCalculateTotalCorrectly() {
        // Arrange
        Order order = new Order(new CustomerId(UUID.randomUUID()));
        Money price1 = new Money(100, "USD");
        Money price2 = new Money(50, "USD");
        
        // Act
        order.addItem(new ProductId(UUID.randomUUID()), 2, price1);
        order.addItem(new ProductId(UUID.randomUUID()), 1, price2);
        
        // Assert
        assertEquals(new Money(250, "USD"), order.getTotalAmount());
    }
    
    @Test
    public void shouldApplyDiscount() {
        // Arrange
        Order order = new Order(new CustomerId(UUID.randomUUID()));
        order.addItem(new ProductId(UUID.randomUUID()), 2, new Money(100, "USD"));
        
        // Act
        order.applyDiscount(10); // 10% discount
        
        // Assert
        assertEquals(new Money(180, "USD"), order.getTotalAmount());
    }
}

// Classes necessárias para os testes

class OrderId {
    private final UUID id;
    
    public OrderId() {
        this.id = UUID.randomUUID();
    }
    
    public UUID getValue() {
        return id;
    }
}

class CustomerId {
    private final UUID id;
    
    public CustomerId(UUID id) {
        this.id = id;
    }
    
    public UUID getValue() {
        return id;
    }
}

class ProductId {
    private final UUID id;
    
    public ProductId(UUID id) {
        this.id = id;
    }
    
    public UUID getValue() {
        return id;
    }
}

class Money {
    private final double amount;
    private final String currency;
    
    public Money(double amount, String currency) {
        if (amount < 0) {
            throw new IllegalArgumentException("Amount cannot be negative");
        }
        this.amount = amount;
        this.currency = currency;
    }
    
    public double getAmount() {
        return amount;
    }
    
    public String getCurrency() {
        return currency;
    }
    
    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Cannot add different currencies");
        }
        return new Money(this.amount + other.amount, this.currency);
    }
    
    public Money multiply(int quantity) {
        return new Money(this.amount * quantity, this.currency);
    }
    
    public Money applyDiscount(double percentageDiscount) {
        double discountAmount = this.amount * (percentageDiscount / 100.0);
        return new Money(this.amount - discountAmount, this.currency);
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Money money = (Money) o;
        return Double.compare(money.amount, amount) == 0 && currency.equals(money.currency);
    }
}

enum OrderStatus {
    CREATED, SUBMITTED, PAID, SHIPPED, DELIVERED, CANCELLED
}

class OrderItem {
    private final ProductId productId;
    private final int quantity;
    private final Money unitPrice;
    
    public OrderItem(ProductId productId, int quantity, Money unitPrice) {
        if (quantity <= 0) {
            throw new IllegalArgumentException("Quantity must be positive");
        }
        this.productId = productId;
        this.quantity = quantity;
        this.unitPrice = unitPrice;
    }
    
    public ProductId getProductId() {
        return productId;
    }
    
    public int getQuantity() {
        return quantity;
    }
    
    public Money getUnitPrice() {
        return unitPrice;
    }
    
    public Money getSubtotal() {
        return unitPrice.multiply(quantity);
    }
}

// Aggregate Root
class Order {
    private final OrderId id;
    private final CustomerId customerId;
    private final List<OrderItem> items;
    private OrderStatus status;
    private Money totalAmount;
    
    public Order(CustomerId customerId) {
        this.id = new OrderId();
        this.customerId = customerId;
        this.items = new ArrayList<>();
        this.status = OrderStatus.CREATED;
        this.totalAmount = new Money(0, "USD");
    }
    
    public OrderId getId() {
        return id;
    }
    
    public CustomerId getCustomerId() {
        return customerId;
    }
    
    public List<OrderItem> getItems() {
        return new ArrayList<>(items); // Retorna cópia defensiva
    }
    
    public OrderStatus getStatus() {
        return status;
    }
    
    public Money getTotalAmount() {
        return totalAmount;
    }
    
    public void addItem(ProductId productId, int quantity, Money unitPrice) {
        if (status != OrderStatus.CREATED) {
            throw new IllegalStateException("Cannot add items to order that is not in CREATED state");
        }
        
        OrderItem item = new OrderItem(productId, quantity, unitPrice);
        items.add(item);
        recalculateTotal();
    }
    
    public void submit() {
        if (items.isEmpty()) {
            throw new IllegalStateException("Cannot submit empty order");
        }
        
        if (status != OrderStatus.CREATED) {
            throw new IllegalStateException("Only orders in CREATED state can be submitted");
        }
        
        status = OrderStatus.SUBMITTED;
    }
    
    public void applyDiscount(double percentageDiscount) {
        if (percentageDiscount < 0 || percentageDiscount > 100) {
            throw new IllegalArgumentException("Discount must be between 0 and 100");
        }
        
        totalAmount = totalAmount.applyDiscount(percentageDiscount);
    }
    
    private void recalculateTotal() {
        Money total = new Money(0, "USD");
        for (OrderItem item : items) {
            total = total.add(item.getSubtotal());
        }
        totalAmount = total;
    }
}
```

```typescript
import { expect } from 'chai';
import { v4 as uuidv4 } from 'uuid';

// Value Objects
class ProductId {
  constructor(private readonly id: string) {
    if (!id) throw new Error('Product ID cannot be empty');
  }

  equals(other: ProductId): boolean {
    return this.id === other.id;
  }

  toString(): string {
    return this.id;
  }
}

class CustomerId {
  constructor(private readonly id: string) {
    if (!id) throw new Error('Customer ID cannot be empty');
  }

  equals(other: CustomerId): boolean {
    return this.id === other.id;
  }

  toString(): string {
    return this.id;
  }
}

class Money {
  constructor(private readonly amount: number, private readonly currency: string) {
    if (amount < 0) throw new Error('Amount cannot be negative');
    if (!currency || currency.length !== 3) throw new Error('Currency must be a 3-letter code');
  }

  getAmount(): number {
    return this.amount;
  }

  getCurrency(): string {
    return this.currency;
  }

  add(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new Error('Cannot add different currencies');
    }
    return new Money(this.amount + other.amount, this.currency);
  }

  multiply(multiplier: number): Money {
    return new Money(this.amount * multiplier, this.currency);
  }

  equals(other: Money): boolean {
    return this.amount === other.amount && this.currency === other.currency;
  }
}

// Entidades
enum OrderStatus {
  Created = 'CREATED',
  Submitted = 'SUBMITTED',
  Paid = 'PAID',
  Shipped = 'SHIPPED',
  Delivered = 'DELIVERED',
  Cancelled = 'CANCELLED'
}

class OrderItem {
  constructor(
    private readonly productId: ProductId,
    private readonly quantity: number,
    private readonly unitPrice: Money
  ) {
    if (quantity <= 0) throw new Error('Quantity must be positive');
  }

  getProductId(): ProductId {
    return this.productId;
  }

  getQuantity(): number {
    return this.quantity;
  }

  getUnitPrice(): Money {
    return this.unitPrice;
  }

  getSubtotal(): Money {
    return this.unitPrice.multiply(this.quantity);
  }
}

// Aggregate Root
class Order {
  private readonly id: string;
  private readonly customerId: CustomerId;
  private readonly items: OrderItem[] = [];
  private status: OrderStatus = OrderStatus.Created;
  private totalAmount: Money = new Money(0, 'USD');

  constructor(customerId: CustomerId) {
    this.id = uuidv4();
    this.customerId = customerId;
  }

  getId(): string {
    return this.id;
  }

  getCustomerId(): CustomerId {
    return this.customerId;
  }

  getItems(): ReadonlyArray<OrderItem> {
    return [...this.items]; // Defensive copy
  }

  getStatus(): OrderStatus {
    return this.status;
  }

  getTotalAmount(): Money {
    return this.totalAmount;
  }

  addItem(productId: ProductId, quantity: number, unitPrice: Money): void {
    if (this.status !== OrderStatus.Created) {
      throw new Error('Cannot add items to order that is not in CREATED state');
    }

    const item = new OrderItem(productId, quantity, unitPrice);
    this.items.push(item);
    this.recalculateTotal();
  }

  submit(): void {
    if (this.items.length === 0) {
      throw new Error('Cannot submit empty order');
    }

    if (this.status !== OrderStatus.Created) {
      throw new Error('Only orders in CREATED state can be submitted');
    }

    this.status = OrderStatus.Submitted;
  }

  cancel(): void {
    if (this.status === OrderStatus.Delivered) {
      throw new Error('Cannot cancel delivered order');
    }

    this.status = OrderStatus.Cancelled;
  }

  private recalculateTotal(): void {
    this.totalAmount = this.items.reduce(
      (total, item) => total.add(item.getSubtotal()),
      new Money(0, 'USD')
    );
  }
}

// Testes
describe('Order Aggregate', () => {
  it('should create an order with valid data', () => {
    // Arrange
    const customerId = new CustomerId(uuidv4());
    
    // Act
    const order = new Order(customerId);
    
    // Assert
    expect(order.getCustomerId()).to.deep.equal(customerId);
    expect(order.getStatus()).to.equal(OrderStatus.Created);
    expect(order.getItems()).to.be.empty;
  });

  it('should add item to order', () => {
    // Arrange
    const order = new Order(new CustomerId(uuidv4()));
    const productId = new ProductId(uuidv4());
    const price = new Money(100, 'USD');
    
    // Act
    order.addItem(productId, 2, price);
    
    // Assert
    expect(order.getItems().length).to.equal(1);
    expect(order.getItems()[0].getProductId()).to.deep.equal(productId);
    expect(order.getItems()[0].getQuantity()).to.equal(2);
    expect(order.getTotalAmount().getAmount()).to.equal(200);
  });

  it('should not add item to submitted order', () => {
    // Arrange
    const order = new Order(new CustomerId(uuidv4()));
    order.addItem(new ProductId(uuidv4()), 1, new Money(100, 'USD'));
    order.submit();
    
    // Act & Assert
    expect(() => {
      order.addItem(new ProductId(uuidv4()), 2, new Money(50, 'USD'));
    }).to.throw('Cannot add items to order that is not in CREATED state');
  });

  it('should calculate total correctly', () => {
    // Arrange
    const order = new Order(new CustomerId(uuidv4()));
    
    // Act
    order.addItem(new ProductId(uuidv4()), 2, new Money(100, 'USD'));
    order.addItem(new ProductId(uuidv4()), 1, new Money(50, 'USD'));
    
    // Assert
    expect(order.getTotalAmount().getAmount()).to.equal(250);
  });

  it('should not submit empty order', () => {
    // Arrange
    const order = new Order(new CustomerId(uuidv4()));
    
    // Act & Assert
    expect(() => {
      order.submit();
    }).to.throw('Cannot submit empty order');
  });

  it('should cancel order in valid state', () => {
    // Arrange
    const order = new Order(new CustomerId(uuidv4()));
    order.addItem(new ProductId(uuidv4()), 1, new Money(100, 'USD'));
    order.submit();
    
    // Act
    order.cancel();
    
    // Assert
    expect(order.getStatus()).to.equal(OrderStatus.Cancelled);
  });
});
```

### Testando Repositories
```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

import java.util.Arrays;
import java.util.List;
import java.util.Optional;
import java.util.UUID;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

// Interface do repositório
interface OrderRepository {
    void save(Order order);
    Optional<Order> findById(OrderId orderId);
    List<Order> findByCustomerId(CustomerId customerId);
}

// Test para serviço que usa o repositório
public class OrderServiceTest {

    private OrderService orderService;
    
    @Mock
    private OrderRepository orderRepository;
    
    @BeforeEach
    public void setUp() {
        MockitoAnnotations.openMocks(this);
        orderService = new OrderService(orderRepository);
    }
    
    @Test
    public void shouldCreateOrderSuccessfully() {
        // Arrange
        CustomerId customerId = new CustomerId(UUID.randomUUID());
        ProductId productId = new ProductId(UUID.randomUUID());
        
        // Act
        Order order = orderService.createOrder(customerId, productId, 2, new Money(100, "USD"));
        
        // Assert
        verify(orderRepository, times(1)).save(order);
        assertEquals(customerId, order.getCustomerId());
        assertEquals(1, order.getItems().size());
        assertEquals(OrderStatus.CREATED, order.getStatus());
    }
    
    @Test
    public void shouldSubmitExistingOrder() {
        // Arrange
        OrderId orderId = new OrderId();
        Order existingOrder = new Order(new CustomerId(UUID.randomUUID()));
        existingOrder.addItem(new ProductId(UUID.randomUUID()), 1, new Money(100, "USD"));
        
        when(orderRepository.findById(orderId)).thenReturn(Optional.of(existingOrder));
        
        // Act
        orderService.submitOrder(orderId);
        
        // Assert
        assertEquals(OrderStatus.SUBMITTED, existingOrder.getStatus());
        verify(orderRepository, times(1)).save(existingOrder);
    }
    
    @Test
    public void shouldThrowExceptionWhenSubmittingNonExistentOrder() {
        // Arrange
        OrderId orderId = new OrderId();
        when(orderRepository.findById(orderId)).thenReturn(Optional.empty());
        
        // Act & Assert
        assertThrows(IllegalArgumentException.class, () -> {
            orderService.submitOrder(orderId);
        });
        
        verify(orderRepository, never()).save(any(Order.class));
    }
    
    @Test
    public void shouldFindOrdersByCustomer() {
        // Arrange
        CustomerId customerId = new CustomerId(UUID.randomUUID());
        Order order1 = new Order(customerId);
        Order order2 = new Order(customerId);
        
        when(orderRepository.findByCustomerId(customerId)).thenReturn(Arrays.asList(order1, order2));
        
        // Act
        List<Order> result = orderService.getOrdersByCustomer(customerId);
        
        // Assert
        assertEquals(2, result.size());
        verify(orderRepository, times(1)).findByCustomerId(customerId);
    }
}

// Serviço que será testado
class OrderService {
    private final OrderRepository orderRepository;
    
    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }
    
    public Order createOrder(CustomerId customerId, ProductId productId, int quantity, Money unitPrice) {
        Order order = new Order(customerId);
        order.addItem(productId, quantity, unitPrice);
        orderRepository.save(order);
        return order;
    }
    
    public void submitOrder(OrderId orderId) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new IllegalArgumentException("Order not found"));
            
        order.submit();
        orderRepository.save(order);
    }
    
    public List<Order> getOrdersByCustomer(CustomerId customerId) {
        return orderRepository.findByCustomerId(customerId);
    }
}
```

```typescript
import { expect } from 'chai';
import * as sinon from 'sinon';
import { v4 as uuidv4 } from 'uuid';

// Usando as mesmas classes de Value Objects e Order do exemplo anterior

// Interface do Repository
interface ProductRepository {
  save(product: Product): Promise<void>;
  findById(id: ProductId): Promise<Product | null>;
  findByCategory(category: ProductCategory): Promise<Product[]>;
  remove(id: ProductId): Promise<boolean>;
}

// Implementação de Repository
class InMemoryProductRepository implements ProductRepository {
  private products: Map<string, Product> = new Map();

  async save(product: Product): Promise<void> {
    this.products.set(product.getId().toString(), product);
  }

  async findById(id: ProductId): Promise<Product | null> {
    const product = this.products.get(id.toString());
    return product || null;
  }

  async findByCategory(category: ProductCategory): Promise<Product[]> {
    return Array.from(this.products.values())
      .filter(product => product.getCategory().equals(category));
  }

  async remove(id: ProductId): Promise<boolean> {
    return this.products.delete(id.toString());
  }
}

// Classes necessárias
class ProductId {
  constructor(private readonly id: string) {
    if (!id) throw new Error('Product ID cannot be empty');
  }

  equals(other: ProductId): boolean {
    return this.id === other.id;
  }

  toString(): string {
    return this.id;
  }
}

class ProductCategory {
  constructor(private readonly name: string) {
    if (!name) throw new Error('Category name cannot be empty');
  }

  getName(): string {
    return this.name;
  }

  equals(other: ProductCategory): boolean {
    return this.name === other.name;
  }
}

class Money {
  constructor(private readonly amount: number, private readonly currency: string) {
    if (amount < 0) throw new Error('Amount cannot be negative');
  }

  getAmount(): number {
    return this.amount;
  }

  getCurrency(): string {
    return this.currency;
  }

  equals(other: Money): boolean {
    return this.amount === other.amount && this.currency === other.currency;
  }
}

class Product {
  constructor(
    private readonly id: ProductId,
    private name: string,
    private price: Money,
    private category: ProductCategory,
    private active: boolean = true
  ) {
    if (!name || name.trim() === '') throw new Error('Product name cannot be empty');
  }

  getId(): ProductId {
    return this.id;
  }

  getName(): string {
    return this.name;
  }

  getPrice(): Money {
    return this.price;
  }

  get
  ```
