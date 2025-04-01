### Arquitetura em Camadas no DDD
O DDD geralmente é implementado com uma arquitetura em camadas:

### 1. Camada de Aplicação

Interface entre os clientes externos e a lógica de domínio.

```java
public class PedidoApplicationService {
    private final RepositorioPedidos repositorioPedidos;
    private final RepositorioClientes repositorioClientes;
    private final FabricaPedido fabricaPedido;
    
    // Construtor com injeção de dependências
    
    @Transactional
    public String criarPedido(String idCliente, List<ItemPedidoDTO> itens) {
        Cliente cliente = repositorioClientes.buscarPorId(idCliente);
        if (cliente == null) {
            throw new ClienteNaoEncontradoException(idCliente);
        }
        
        Pedido pedido = fabricaPedido.criarPedido(cliente, itens);
        repositorioPedidos.salvar(pedido);
        
        return pedido.getNumeroPedido();
    }
    
    @Transactional
    public void confirmarPedido(String numeroPedido) {
        Pedido pedido = repositorioPedidos.buscarPorNumero(numeroPedido);
        if (pedido == null) {
            throw new PedidoNaoEncontradoException(numeroPedido);
        }
        
        pedido.confirmar();
        repositorioPedidos.salvar(pedido);
    }
}
```

### 2. Camada de Domínio

Contém as regras de negócio, entidades, value objects, services de domínio, etc.

// Esta é a camada central onde estão os exemplos de Entidades, Value Objects, 
// Agregados e Serviços de Domínio mostrados anteriormente

### 3. Camada de Infraestrutura

Implementa as interfaces definidas no domínio para interagir com bancos de dados, serviços externos, etc.

// Implementações de repositórios, serviços de envio de email, integrações externas, etc.
### 4. Camada de Interface (UI/API)

```java
@RestController
@RequestMapping("/pedidos")
public class PedidoController {
    private final PedidoApplicationService pedidoService;
    
    @PostMapping
    public ResponseEntity<String> criarPedido(@RequestBody CriarPedidoRequest request) {
        try {
            String numeroPedido = pedidoService.criarPedido(
                request.getIdCliente(), 
                request.getItens()
            );
            return ResponseEntity.ok(numeroPedido);
        } catch (Exception e) {
            return ResponseEntity.badRequest().body(e.getMessage());
        }
    }
    
    @PutMapping("/{numeroPedido}/confirmar")
    public ResponseEntity<Void> confirmarPedido(@PathVariable String numeroPedido) {
        try {
            pedidoService.confirmarPedido(numeroPedido);
            return ResponseEntity.ok().build();
        } catch (Exception e) {
            return ResponseEntity.badRequest().build();
        }
    }
}
```
