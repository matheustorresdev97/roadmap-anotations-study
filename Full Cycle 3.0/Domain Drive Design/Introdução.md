É uma forma de desenvolver software com o foco no coração da aplicação - o que chamamos de domínio - tendo o objetivo de entender suas regras, processos e complexidades, separando-as assim de outros pontos complexos que normalmente são adicionados durante o processo de desenvolvimento.
Esta metodologia se concentra em:
- Criar um modelo rico que reflete o domínio do problema
- Estabelecer uma linguagem ubíqua entre desenvolvedores e especialistas de domínio
- Isolar a lógica de domínio de outras preocupações técnicas

### Softwares complexos
- DDD é/ deve ser aplicado para casos de projetos de softwares complexos
- Grandes projetos possuem muitas áreas, muitas regras de negócio, muitas pessoas com diferentes visões em diferentes contextos.
- Não há como não utilizar técnicas avançadas em projetos de alta complexidade
- Grande parte da complexidade desse tipo de software não vem da tecnologia, mas sim da comunicação, separação de contextos, entendimento do negócio por diversos ângulos
- Pessoas


### Como DDD pode ajudar ?
- Entender com profundidade o domínio e subdomínios da aplicação
- Ter uma linguagem universal (linguagem ubíqua entre todos os envolvidos)
- Criar o design estratégico utilizando Bounded Contexts
- Criar o design tático para conseguir mapear e agregar as entidades e objetos de valor da aplicação, bem como os eventos de domínio
- Clareza do que é complexidade de negócio e complexidade técnica

### Conceitos Fundamentais do DDD

### 1. Linguagem Ubíqua

A Linguagem Ubíqua é um vocabulário compartilhado entre desenvolvedores e especialistas do domínio. Ela é usada tanto na comunicação quanto no código.

**Exemplo:** Em um sistema bancário, evite termos genéricos como "processar()" ou "atualizar()"; use termos do domínio como "depositarValor()", "sacarValor()", "transferirSaldo()".

```java
// Em vez disso:
public void processar(Conta conta, double valor) {
    conta.setSaldo(conta.getSaldo() + valor);
}

// Use isso:
public void depositarValor(Conta conta, double valor) {
    conta.adicionarSaldo(valor);
}
```

### 2. Bounded Contexts (Contextos Delimitados)

Um Bounded Context é uma fronteira explícita dentro da qual um modelo de domínio específico se aplica. Diferentes contextos podem ter diferentes definições para os mesmos termos.

**Exemplo:** No contexto de "Vendas", um "Cliente" pode representar uma pessoa interessada em comprar produtos. No contexto de "Suporte", um "Cliente" pode ser alguém que já comprou um produto e precisa de assistência.

```java
// Bounded Context: Vendas
public class Cliente {
    private String nome;
    private String email;
    private List<ProdutoInteresse> produtosDeInteresse;
    private Carrinho carrinhoAtual;
    
    public void adicionarAoCarrinho(Produto produto) {
        // Lógica específica de vendas
    }
}

// Bounded Context: Suporte
public class Cliente {
    private String nome;
    private String email;
    private List<Compra> historicoCompras;
    private List<ChamadoSuporte> chamadosAbertos;
    
    public ChamadoSuporte abrirChamado(String descricaoProblema) {
        // Lógica específica de suporte
    }
}
```

### 3. Entities (Entidades)

Entidades são objetos com identidade única que persistem ao longo do tempo. Dois objetos com os mesmos atributos, mas identidades diferentes, são considerados objetos diferentes.

```java
public class Conta {
    private final String numeroConta; // Identidade da conta
    private double saldo;
    private Cliente titular;
    
    // Dois objetos Conta são iguais se tiverem o mesmo número de conta
    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Conta)) return false;
        Conta outraConta = (Conta) obj;
        return this.numeroConta.equals(outraConta.numeroConta);
    }
    
    @Override
    public int hashCode() {
        return numeroConta.hashCode();
    }
}
```

### 4. Value Objects (Objetos de Valor)

Value Objects são imutáveis e sem identidade. São definidos apenas por seus atributos e são considerados iguais se todos os atributos forem iguais.

```java
public final class Endereco {
    private final String rua;
    private final String numero;
    private final String cidade;
    private final String estado;
    private final String cep;
    
    public Endereco(String rua, String numero, String cidade, String estado, String cep) {
        this.rua = rua;
        this.numero = numero;
        this.cidade = cidade;
        this.estado = estado;
        this.cep = cep;
    }
    
    // Sem setters - imutável
    
    // Dois endereços são iguais se todos os atributos forem iguais
    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Endereco)) return false;
        Endereco outro = (Endereco) obj;
        return  Objects.equals(this.rua, outro.rua) &&
                Objects.equals(this.numero, outro.numero) &&
                Objects.equals(this.cidade, outro.cidade) &&
                Objects.equals(this.estado, outro.estado) &&
                Objects.equals(this.cep, outro.cep);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(rua, numero, cidade, estado, cep);
    }
}
```

### 5. Aggregates (Agregados)

Agregados são grupos de objetos relacionados que são tratados como uma unidade para mudanças de dados. Cada agregado tem uma raiz (Aggregate Root) que é a única entidade acessível externamente.

```java
// Aggregate Root
public class Pedido {
    private final String numeroPedido;
    private Cliente cliente;
    private List<ItemPedido> itens = new ArrayList<>();
    private StatusPedido status;
    
    public void adicionarItem(Produto produto, int quantidade) {
        ItemPedido item = new ItemPedido(produto, quantidade);
        itens.add(item);
    }
    
    public void removerItem(int indiceItem) {
        if (indiceItem >= 0 && indiceItem < itens.size()) {
            itens.remove(indiceItem);
        }
    }
    
    public void confirmar() {
        if (itens.isEmpty()) {
            throw new IllegalStateException("Pedido não pode ser confirmado sem itens");
        }
        this.status = StatusPedido.CONFIRMADO;
    }
    
    // Acesso controlado aos itens
    public List<ItemPedido> getItensView() {
        return Collections.unmodifiableList(itens);
    }
}

// Parte do Aggregate Pedido
public class ItemPedido {
    private Produto produto;
    private int quantidade;
    private double precoUnitario;
    
    // ItemPedido só existe no contexto de um Pedido
}
```

### 6. Domain Services (Serviços de Domínio)

Serviços de Domínio encapsulam operações que não pertencem naturalmente a nenhuma entidade ou objeto de valor específico.

```java
public class ServicoTransferencia {
    
    public void transferir(Conta origem, Conta destino, double valor) {
        if (valor <= 0) {
            throw new IllegalArgumentException("Valor deve ser positivo");
        }
        
        if (origem.getSaldo() < valor) {
            throw new SaldoInsuficienteException("Saldo insuficiente para transferência");
        }
        
        origem.debitar(valor);
        destino.creditar(valor);
    }
}
```

### 7. Repositories (Repositórios)

Repositórios abstraem a persistência dos agregados, fornecendo métodos para recuperar e armazenar objetos.
Exemplo:

```java
public interface RepositorioPedidos {
    Pedido buscarPorNumero(String numeroPedido);
    List<Pedido> buscarPorCliente(String idCliente);
    List<Pedido> buscarPorStatus(StatusPedido status);
    void salvar(Pedido pedido);
    void remover(Pedido pedido);
}

// Implementação usando JPA
public class RepositorioPedidosJPA implements RepositorioPedidos {
    @PersistenceContext
    private EntityManager entityManager;
    
    @Override
    public Pedido buscarPorNumero(String numeroPedido) {
        return entityManager.find(Pedido.class, numeroPedido);
    }
    
    @Override
    public List<Pedido> buscarPorCliente(String idCliente) {
        return entityManager.createQuery(
            "SELECT p FROM Pedido p WHERE p.cliente.id = :idCliente", 
            Pedido.class)
            .setParameter("idCliente", idCliente)
            .getResultList();
    }
    
    // Outros métodos...
}
```

### 8. Factories (Fábricas)

Fábricas encapsulam a criação de objetos de domínio complexos e agregados.

**Exemplo:** 

```java
public class FabricaPedido {
    
    public Pedido criarPedido(Cliente cliente, List<ItemPedidoDTO> itensDTO) {
        Pedido pedido = new Pedido(gerarNumeroPedido(), cliente);
        
        for (ItemPedidoDTO itemDTO : itensDTO) {
            Produto produto = repositorioProdutos.buscarPorCodigo(itemDTO.getCodigoProduto());
            if (produto == null) {
                throw new ProdutoNaoEncontradoException(itemDTO.getCodigoProduto());
            }
            
            pedido.adicionarItem(produto, itemDTO.getQuantidade());
        }
        
        return pedido;
    }
    
    private String gerarNumeroPedido() {
        return "PED-" + UUID.randomUUID().toString().substring(0, 8);
    }
}
```

