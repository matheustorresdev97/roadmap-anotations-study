"Um serviço de domínio é uma operação sem estado que cumpre uma tarefa específica do domínio. Muitas vezes, a melhor indicação de que você deve criar um Serviço no modelo de domínio é quando a operação que você precisa executar parece não se encaixar como um método em um Agregado (10) ou um Objeto de Valor (6)."
"Quando um processo ou transformação significativa no domínio não for uma responsabilidade natural de uma ENTIDADE ou OBJETO  DE VALOR, adicione uma operação ao modelo como uma interface autônoma declarada como um SERVIÇO. Defina a interface em baseada na linguagem do modelo de domínio e certifique-se de que o nome da operação faça parte do UBIQUITOUS LANGUAGE. Torne o SERVIÇO sem estado."

- Uma entidade pode realizar uma ação que vai afetar todas as entidades? 
- Como realizar uma operação em lote? 
- Como calcular algo cuja as informações constam em mais de uma entidade? 

CUIDADOS
- Quando houver muitos Domain Services em seu projeto, TALVEZ, isso pode indicar que seus agregados estão anêmicos
- Domain Services são Stateless

```typescript
// Entidades do domínio
class ContaBancaria {
  constructor(
    private readonly id: string,
    private readonly titular: string,
    private saldo: number
  ) {}

  public getId(): string {
    return this.id;
  }

  public getSaldo(): number {
    return this.saldo;
  }

  public debitar(valor: number): boolean {
    if (valor <= 0) {
      throw new Error("Valor para débito deve ser positivo");
    }
    
    if (this.saldo < valor) {
      return false;
    }
    
    this.saldo -= valor;
    return true;
  }

  public creditar(valor: number): void {
    if (valor <= 0) {
      throw new Error("Valor para crédito deve ser positivo");
    }
    
    this.saldo += valor;
  }
}

// Repositório para acesso às entidades
interface ContaBancariaRepository {
  findById(id: string): ContaBancaria | null;
  save(conta: ContaBancaria): void;
}

// Value Object
class TransferenciaInfo {
  constructor(
    public readonly contaOrigemId: string,
    public readonly contaDestinoId: string,
    public readonly valor: number,
    public readonly dataHora: Date = new Date()
  ) {
    if (valor <= 0) {
      throw new Error("Valor de transferência deve ser positivo");
    }
  }
}

// Domain Service
class TransferenciaFinanceiraService {
  constructor(private readonly contaRepository: ContaBancariaRepository) {}

  // Método principal do serviço de domínio
  public realizarTransferencia(info: TransferenciaInfo): boolean {
    // Validação básica
    if (info.contaOrigemId === info.contaDestinoId) {
      throw new Error("Contas de origem e destino não podem ser iguais");
    }

    // Recupera as entidades do repositório
    const contaOrigem = this.contaRepository.findById(info.contaOrigemId);
    const contaDestino = this.contaRepository.findById(info.contaDestinoId);

    if (!contaOrigem || !contaDestino) {
      throw new Error("Conta de origem ou destino não encontrada");
    }

    // Aplica regras de negócio
    const sucessoDebito = contaOrigem.debitar(info.valor);
    if (!sucessoDebito) {
      return false; // Não há saldo suficiente
    }

    // Completa a operação
    contaDestino.creditar(info.valor);

    // Persiste as mudanças
    this.contaRepository.save(contaOrigem);
    this.contaRepository.save(contaDestino);

    return true;
  }
}

// Exemplo de uso
const repo: ContaBancariaRepository = {/* implementação do repositório */} as any;
const transferenciaService = new TransferenciaFinanceiraService(repo);

const transferInfo = new TransferenciaInfo(
  "conta-123",
  "conta-456",
  100.00
);

const resultado = transferenciaService.realizarTransferencia(transferInfo);
console.log(`Transferência realizada com sucesso: ${resultado}`);
```

```java
package com.example.domain;

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.List;

// Entidade Produto
class Produto {
    private final String id;
    private final String nome;
    private final BigDecimal precoUnitario;
    private final int pesoEmGramas;
    private final String categoria;

    public Produto(String id, String nome, BigDecimal precoUnitario, int pesoEmGramas, String categoria) {
        this.id = id;
        this.nome = nome;
        this.precoUnitario = precoUnitario;
        this.pesoEmGramas = pesoEmGramas;
        this.categoria = categoria;
    }

    public String getId() {
        return id;
    }

    public String getNome() {
        return nome;
    }

    public BigDecimal getPrecoUnitario() {
        return precoUnitario;
    }

    public int getPesoEmGramas() {
        return pesoEmGramas;
    }

    public String getCategoria() {
        return categoria;
    }
}

// Entidade Pedido
class Pedido {
    private final String id;
    private final String clienteId;
    private final List<ItemPedido> itens;
    private final Endereco enderecoEntrega;
    private final LocalDateTime dataCriacao;
    private BigDecimal valorTotal;
    private BigDecimal valorFrete;

    public Pedido(String id, String clienteId, List<ItemPedido> itens, Endereco enderecoEntrega) {
        this.id = id;
        this.clienteId = clienteId;
        this.itens = itens;
        this.enderecoEntrega = enderecoEntrega;
        this.dataCriacao = LocalDateTime.now();
        this.valorTotal = BigDecimal.ZERO;
        this.valorFrete = BigDecimal.ZERO;
    }

    public String getId() {
        return id;
    }

    public List<ItemPedido> getItens() {
        return itens;
    }

    public Endereco getEnderecoEntrega() {
        return enderecoEntrega;
    }

    public BigDecimal getValorTotal() {
        return valorTotal;
    }

    public void setValorTotal(BigDecimal valorTotal) {
        this.valorTotal = valorTotal;
    }

    public BigDecimal getValorFrete() {
        return valorFrete;
    }

    public void setValorFrete(BigDecimal valorFrete) {
        this.valorFrete = valorFrete;
    }

    public int getPesoTotalEmGramas() {
        return itens.stream()
                .mapToInt(item -> item.getProduto().getPesoEmGramas() * item.getQuantidade())
                .sum();
    }
}

// Value Object
class ItemPedido {
    private final Produto produto;
    private final int quantidade;

    public ItemPedido(Produto produto, int quantidade) {
        this.produto = produto;
        this.quantidade = quantidade;
    }

    public Produto getProduto() {
        return produto;
    }

    public int getQuantidade() {
        return quantidade;
    }

    public BigDecimal getSubtotal() {
        return produto.getPrecoUnitario().multiply(BigDecimal.valueOf(quantidade));
    }
}

// Value Object
class Endereco {
    private final String cep;
    private final String rua;
    private final String numero;
    private final String complemento;
    private final String bairro;
    private final String cidade;
    private final String estado;

    public Endereco(String cep, String rua, String numero, String complemento, String bairro, String cidade, String estado) {
        this.cep = cep;
        this.rua = rua;
        this.numero = numero;
        this.complemento = complemento;
        this.bairro = bairro;
        this.cidade = cidade;
        this.estado = estado;
    }

    public String getCep() {
        return cep;
    }

    public String getEstado() {
        return estado;
    }

    public String getCidade() {
        return cidade;
    }
}

// Repositório
interface ProdutoRepository {
    Produto findById(String id);
}

// Regras para cálculo de frete (poderia ser extraído para um serviço separado)
enum RegiaoFrete {
    SUDESTE, SUL, CENTRO_OESTE, NORDESTE, NORTE
}

// Domain Service
public class CalculadoraFreteService {
    private final ProdutoRepository produtoRepository;

    // Taxas de frete por região (em reais por kg)
    private static final BigDecimal TAXA_SUDESTE = new BigDecimal("5.00");
    private static final BigDecimal TAXA_SUL = new BigDecimal("7.00");
    private static final BigDecimal TAXA_CENTRO_OESTE = new BigDecimal("7.50");
    private static final BigDecimal TAXA_NORDESTE = new BigDecimal("10.00");
    private static final BigDecimal TAXA_NORTE = new BigDecimal("12.00");

    // Valor mínimo para frete grátis
    private static final BigDecimal VALOR_MIN_FRETE_GRATIS = new BigDecimal("300.00");

    public CalculadoraFreteService(ProdutoRepository produtoRepository) {
        this.produtoRepository = produtoRepository;
    }

    // Método principal do serviço de domínio
    public BigDecimal calcularFrete(Pedido pedido) {
        // Produtos de categoria "DIGITAL" não têm frete
        boolean temSomenteDigital = pedido.getItens().stream()
                .allMatch(item -> "DIGITAL".equals(item.getProduto().getCategoria()));
        
        if (temSomenteDigital) {
            return BigDecimal.ZERO;
        }

        // Verifica regra de frete grátis por valor mínimo
        BigDecimal valorTotalProdutos = calcularValorTotalProdutos(pedido);
        pedido.setValorTotal(valorTotalProdutos);
        
        if (valorTotalProdutos.compareTo(VALOR_MIN_FRETE_GRATIS) >= 0) {
            return BigDecimal.ZERO;
        }

        // Calcula frete baseado na região e peso
        RegiaoFrete regiao = determinarRegiao(pedido.getEnderecoEntrega().getEstado());
        BigDecimal taxaPorKg = obterTaxaPorRegiao(regiao);
        
        // Calcula o valor do frete (peso em kg * taxa)
        int pesoEmGramas = pedido.getPesoTotalEmGramas();
        BigDecimal pesoEmKg = new BigDecimal(pesoEmGramas).divide(new BigDecimal(1000));
        BigDecimal valorFrete = pesoEmKg.multiply(taxaPorKg);
        
        // Valor mínimo de frete é R$ 10,00
        return valorFrete.max(new BigDecimal("10.00"));
    }

    // Calcula o valor total dos produtos no pedido
    private BigDecimal calcularValorTotalProdutos(Pedido pedido) {
        return pedido.getItens().stream()
                .map(ItemPedido::getSubtotal)
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    }

    // Determina a região com base no estado
    private RegiaoFrete determinarRegiao(String estado) {
        switch (estado.toUpperCase()) {
            case "SP":
            case "RJ":
            case "ES":
            case "MG":
                return RegiaoFrete.SUDESTE;
            case "PR":
            case "SC":
            case "RS":
                return RegiaoFrete.SUL;
            case "MT":
            case "MS":
            case "GO":
            case "DF":
                return RegiaoFrete.CENTRO_OESTE;
            case "BA":
            case "SE":
            case "AL":
            case "PE":
            case "PB":
            case "RN":
            case "CE":
            case "PI":
            case "MA":
                return RegiaoFrete.NORDESTE;
            default:
                return RegiaoFrete.NORTE;
        }
    }

    // Obtém a taxa por kg conforme a região
    private BigDecimal obterTaxaPorRegiao(RegiaoFrete regiao) {
        switch (regiao) {
            case SUDESTE:
                return TAXA_SUDESTE;
            case SUL:
                return TAXA_SUL;
            case CENTRO_OESTE:
                return TAXA_CENTRO_OESTE;
            case NORDESTE:
                return TAXA_NORDESTE;
            case NORTE:
                return TAXA_NORTE;
            default:
                throw new IllegalArgumentException("Região não suportada");
        }
    }
}
```

