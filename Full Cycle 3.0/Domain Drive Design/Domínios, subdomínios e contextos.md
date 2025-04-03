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

### Padrões Estratégicos em DDD
### 1. Event Sourcing
Armazena todas as mudanças de estado como eventos, permitindo reconstruir o estado atual a partir da sequência de eventos.

```java
public class EventoContaCriada extends EventoDominio {
    private final String numeroConta;
    private final String titularId;
    private final double saldoInicial;
    
    // Getters e construtor
}

public class EventoValorDebitado extends EventoDominio {
    private final String numeroConta;
    private final double valor;
    
    // Getters e construtor
}

// Aplicando os eventos a uma conta
public class Conta {
    private String numeroConta;
    private String titularId;
    private double saldo;
    
    public void aplicarEvento(EventoContaCriada evento) {
        this.numeroConta = evento.getNumeroConta();
        this.titularId = evento.getTitularId();
        this.saldo = evento.getSaldoInicial();
    }
    
    public void aplicarEvento(EventoValorDebitado evento) {
        this.saldo -= evento.getValor();
    }
}
```

### 2. CQRS (Command Query Responsibility Segregation)

Separa as operações de leitura (queries) das operações de escrita (commands).

```java
// Command
public class DepositarValorCommand {
    private final String numeroConta;
    private final double valor;
    
    // Getters e construtor
}

// Command Handler
public class ContaCommandHandler {
    private final RepositorioContas repositorio;
    
    public void handle(DepositarValorCommand command) {
        Conta conta = repositorio.buscarPorNumero(command.getNumeroConta());
        conta.depositar(command.getValor());
        repositorio.salvar(conta);
    }
}

// Query Model (otimizado para leitura)
public class SaldoContaDTO {
    private String numeroConta;
    private String nomeTitular;
    private double saldo;
    
    // Getters e construtor
}

// Query Service
public class ContaQueryService {
    private final EntityManager em;
    
    public SaldoContaDTO buscarSaldo(String numeroConta) {
        return em.createQuery(
            "SELECT new com.exemplo.SaldoContaDTO(c.numeroConta, t.nome, c.saldo) " +
            "FROM Conta c JOIN c.titular t " +
            "WHERE c.numeroConta = :numero", 
            SaldoContaDTO.class)
            .setParameter("numero", numeroConta)
            .getSingleResult();
    }
}
```

### Domain Events (Eventos de Domínio)

Eventos de domínio representam coisas que aconteceram no domínio e podem desencadear outras ações.

```java
public abstract class EventoDominio {
    private final LocalDateTime ocorreuEm;
    private final UUID eventoId;
    
    protected EventoDominio() {
        this.eventoId = UUID.randomUUID();
        this.ocorreuEm = LocalDateTime.now();
    }
    
    // Getters
}

public class PedidoConfirmadoEvent extends EventoDominio {
    private final String numeroPedido;
    private final String clienteId;
    private final double valorTotal;
    
    // Getters e construtor
}

// Publicando eventos
public class Pedido {
    private List<EventoDominio> eventos = new ArrayList<>();
    
    public void confirmar() {
        if (itens.isEmpty()) {
            throw new IllegalStateException("Pedido não pode ser confirmado sem itens");
        }
        this.status = StatusPedido.CONFIRMADO;
        
        // Registra o evento
        eventos.add(new PedidoConfirmadoEvent(
            this.numeroPedido, 
            this.cliente.getId(),
            this.calcularValorTotal()
        ));
    }
    
    public List<EventoDominio> coletarEventos() {
        List<EventoDominio> eventosColetados = new ArrayList<>(this.eventos);
        this.eventos.clear();
        return eventosColetados;
    }
}

// Consumindo eventos
@Service
public class NotificacaoEventHandler {
    private final ServicoNotificacao servicoNotificacao;
    
    @EventListener
    public void ao(PedidoConfirmadoEvent evento) {
        servicoNotificacao.notificarClienteSobrePedido(
            evento.getClienteId(), 
            evento.getNumeroPedido()
        );
    }
}
```

### Benefícios e Desafios do DDD

 Benefícios:

1. **Alinhamento com o negócio**: O software reflete o modelo mental dos especialistas de domínio
2. **Redução da complexidade**: Dividindo o sistema em contextos menores e mais gerenciáveis
3. **Comunicação eficaz**: Linguagem ubíqua facilita a comunicação entre desenvolvedores e não-técnicos
4. **Adaptabilidade**: Facilita mudanças à medida que o entendimento do domínio evolui
5. **Foco estratégico**: Concentra esforços nas partes mais valiosas do sistema

 Desafios:

1. **Curva de aprendizado**: Requer tempo para dominar os conceitos e padrões
2. **Complexidade inicial**: Pode parecer mais complexo do que abordagens mais simples inicialmente
3. **Overhead potencial**: Nem todos os sistemas precisam da sofisticação do DDD
4. **Definição de contextos**: Identificar corretamente os bounded contexts pode ser difícil
5. **Resistência organizacional**: Pode exigir mudanças na forma como equipes trabalham juntas


 Quando usar DDD

O DDD é mais adequado para:

- Sistemas complexos com lógica de negócio rica
- Projetos de longo prazo que evoluirão com o tempo
- Equipes multidisciplinares que precisam colaborar estreitamente

Pode ser excessivo para:

- Aplicações CRUD simples
- Protótipos ou MVPs com prazo curto
- Sistemas pequenos com domínio trivial


### Exemplo Prático: Sistema de Gerenciamento de Biblioteca
 Bounded Contexts:

1. **Catálogo**: Gerenciamento de livros e outros materiais
2. **Empréstimos**: Controle de empréstimos e devoluções
3. **Membros**: Gerenciamento de usuários da biblioteca

```java
// Value Object
public final class PeriodoEmprestimo {
    private final LocalDate inicio;
    private final LocalDate previsaoDevolucao;
    
    public PeriodoEmprestimo(LocalDate inicio, int diasDuracao) {
        this.inicio = inicio;
        this.previsaoDevolucao = inicio.plusDays(diasDuracao);
    }
    
    public boolean estaEmAtraso(LocalDate dataReferencia) {
        return dataReferencia.isAfter(previsaoDevolucao);
    }
    
    // Getters, equals, hashCode
}

// Entity e Aggregate Root
public class Emprestimo {
    private final String id;
    private final String membroId;
    private final String exemplarId;
    private PeriodoEmprestimo periodo;
    private LocalDate dataDevolucao;
    private StatusEmprestimo status;
    
    public Emprestimo(String membroId, String exemplarId, PeriodoEmprestimo periodo) {
        this.id = UUID.randomUUID().toString();
        this.membroId = membroId;
        this.exemplarId = exemplarId;
        this.periodo = periodo;
        this.status = StatusEmprestimo.ATIVO;
    }
    
    public void renovar(int diasAdicionais) {
        if (status != StatusEmprestimo.ATIVO) {
            throw new IllegalStateException("Apenas empréstimos ativos podem ser renovados");
        }
        
        LocalDate novoInicio = LocalDate.now();
        this.periodo = new PeriodoEmprestimo(novoInicio, diasAdicionais);
    }
    
    public void devolver(LocalDate dataDevolucao) {
        if (status != StatusEmprestimo.ATIVO) {
            throw new IllegalStateException("Este empréstimo já foi finalizado");
        }
        
        this.dataDevolucao = dataDevolucao;
        this.status = StatusEmprestimo.DEVOLVIDO;
    }
    
    public boolean estaEmAtraso() {
        if (status == StatusEmprestimo.DEVOLVIDO) {
            return periodo.estaEmAtraso(dataDevolucao);
        }
        return periodo.estaEmAtraso(LocalDate.now());
    }
    
    // Getters, etc.
}

// Domain Service
public class ServicoEmprestimo {
    private final RepositorioEmprestimos repositorioEmprestimos;
    private final ServicoDisponibilidade servicoDisponibilidade;
    private final PoliticaEmprestimo politicaEmprestimo;
    
    public Emprestimo realizarEmprestimo(String membroId, String exemplarId) {
        // Verificar disponibilidade do exemplar
        if (!servicoDisponibilidade.estaDisponivel(exemplarId)) {
            throw new ExemplarIndisponivelException(exemplarId);
        }
        
        // Verificar se o membro pode pegar emprestado
        int diasPermitidos = politicaEmprestimo.calcularDiasPermitidos(membroId);
        if (diasPermitidos <= 0) {
            throw new EmprestimoNaoPermitidoException(membroId);
        }
        
        // Criar e salvar o empréstimo
        PeriodoEmprestimo periodo = new PeriodoEmprestimo(LocalDate.now(), diasPermitidos);
        Emprestimo emprestimo = new Emprestimo(membroId, exemplarId, periodo);
        repositorioEmprestimos.salvar(emprestimo);
        
        return emprestimo;
    }
}

// Application Service
public class EmprestimoApplicationService {
    private final ServicoEmprestimo servicoEmprestimo;
    private final RepositorioEmprestimos repositorioEmprestimos;
    
    public EmprestimoDTO realizarEmprestimo(RealizarEmprestimoCommand command) {
        Emprestimo emprestimo = servicoEmprestimo.realizarEmprestimo(
            command.getMembroId(), 
            command.getExemplarId()
        );
        
        return EmprestimoDTO.from(emprestimo);
    }
    
    public void devolverEmprestimo(DevolverEmprestimoCommand command) {
        Emprestimo emprestimo = repositorioEmprestimos.buscarPorId(command.getEmprestimoId());
        if (emprestimo == null) {
            throw new EmprestimoNaoEncontradoException(command.getEmprestimoId());
        }
        
        emprestimo.devolver(LocalDate.now());
        repositorioEmprestimos.salvar(emprestimo);
    }
}
```


### Domínio vs Subdomínio

 Domínio
O **domínio** é a esfera de conhecimento e atividade específica na qual a organização opera. É todo o universo conceitual relacionado ao problema de negócio que o software pretende resolver.
**Exemplo:** Para uma companhia aérea, o domínio inclui todo o conhecimento sobre voos, aeronaves, reservas, tripulação, rotas aéreas, manutenção, serviços de bordo, etc.

 Subdomínio
Um **subdomínio** é uma parte específica e coesa do domínio, representando uma área de especialização ou uma divisão lógica dentro do domínio maior.
Existem três tipos principais de subdomínios:
- **Subdomínio Principal (Core):** Representa a competência principal da organização, o diferencial competitivo.
    - _Exemplo na companhia aérea:_ Gerenciamento de voos e rotas.
- **Subdomínio de Suporte:** Apoia a operação do negócio, mas não é o diferencial.
    - _Exemplo na companhia aérea:_ Gerenciamento de tripulação.
- **Subdomínio Genérico:** Não é específico do negócio e poderia ser terceirizado ou adquirido.
    - _Exemplo na companhia aérea:_ Sistema de contabilidade.

**Importância da distinção:**
- Ajuda a priorizar investimentos de desenvolvimento
- Identificação de áreas onde mais especialização de domínio é necessária
- Decisões sobre construir vs. comprar soluções

### Espaço do Problema vs Espaço da Solução
Espaço do Problema
O **espaço do problema** refere-se ao domínio do mundo real e os desafios que a organização enfrenta. É o "o quê" e o "por quê" - a compreensão dos problemas que precisam ser resolvidos.

**Características:**
- Focado no entendimento do domínio
- Independente de tecnologia
- Corresponde aos subdomínios
- Está relacionado à descoberta e análise

**Exemplo:** Na área de saúde, entender o fluxo de pacientes em um hospital, os requisitos de agendamento, o histórico médico necessário.

Espaço da Solução
O **espaço da solução** representa como os problemas identificados serão resolvidos no software. É o "como" - a implementação técnica que atende às necessidades do domínio.

**Características:**
- Focado na implementação
- Dependente de tecnologia
- Corresponde aos Bounded Contexts
- Está relacionado ao design e implementação

**Exemplo:** Uma arquitetura de microsserviços com um serviço para agendamento de consultas, outro para gestão de prontuários, interfaces definidas, bancos de dados específicos.

**Relação entre eles:**
- Um subdomínio (espaço do problema) pode ser implementado por um ou mais Bounded Contexts (espaço da solução)
- A transição do espaço do problema para o espaço da solução requer tradução de conceitos do domínio para implementações técnicas

### O que é um Contexto Delimitado (Bounded Context) ?
Um **Bounded Context** é uma fronteira explícita (conceitual e técnica) dentro da qual um modelo de domínio específico é definido e aplicável. É a principal ferramenta estratégica no DDD para lidar com modelos grandes e complexos.

**Características essenciais:**
- Define uma fronteira lógica para um modelo de domínio
- Possui seu próprio vocabulário e modelo conceitual internamente consistente
- Encapsula detalhes de implementação específicos
- Interage com outros contextos delimitados através de interfaces bem definidas
- Pode ser implementado como uma aplicação, serviço ou módulo separado

**Exemplo:** Em um e-commerce:
- **Bounded Context de Catálogo:** Gerencia produtos, categorias, descrições, imagens
- **Bounded Context de Pedidos:** Lida com carrinho, checkout, status de pedido
- **Bounded Context de Pagamentos:** Processa transações financeiras, reembolsos

O mesmo conceito (ex: "Produto") pode significar coisas diferentes em cada contexto:
- No contexto de Catálogo: Detalhes completos, estoque, categoria
- No contexto de Pedidos: ID, preço no momento da compra, quantidade
- No contexto de Envio: Dimensões, peso, localização

### Contexto é Rei
"Contexto é Rei" (ou "Context is King") é um princípio fundamental no DDD que enfatiza a importância do contexto na definição do significado e comportamento dos elementos de domínio.

**O que significa:**

- Termos e conceitos só têm significado preciso dentro de um contexto específico
- O mesmo termo pode ter significados diferentes em contextos diferentes
- A modelagem deve ser feita dentro dos limites do contexto
- As regras de negócio são válidas apenas dentro do contexto onde foram definidas

**Exemplo prático:** O termo "Cliente" em diferentes contextos:

- **Contexto de Vendas:** Uma pessoa ou empresa interessada em comprar produtos (com dados de contato, histórico de compras)
- **Contexto de Suporte:** Um usuário com problemas a serem resolvidos (com histórico de tickets, nível de SLA)
- **Contexto Financeiro:** Uma entidade com faturamento e pagamentos (com dados fiscais, limites de crédito)

**Implicações:**

- Evite criar modelos universais que tentam atender a todos os contextos
- Aceite que a duplicação de conceitos entre contextos é normal e até desejável
- Concentre-se em criar modelos que façam sentido dentro de um contexto específico
- Defina explicitamente como os contextos se comunicam e traduzem conceitos entre si

### Elementos Transversais
**Elementos transversais** (ou Cross-cutting concerns) são aspectos que afetam múltiplos contextos delimitados e não podem ser encapsulados em um único contexto.

**Tipos comuns:**

1. **Identidade:** Como entidades são identificadas de forma consistente através do sistema
2. **Autenticação e Autorização:** Quem pode acessar o quê em diferentes contextos
3. **Auditoria e Logging:** Registro de atividades e mudanças no sistema
4. **Configuração:** Parâmetros compartilhados do sistema
5. **Validação:** Regras que se aplicam em múltiplos contextos

**Abordagens para gerenciar elementos transversais:**

1. **Shared Kernel:** Compartilhar uma parte do modelo entre contextos
2. **Serviços de Utilidade:** Criar serviços específicos para preocupações transversais
3. **Aspectos (AOP):** Aplicar comportamentos de forma declarativa
4. **Event-driven:** Usar eventos para manter consistência eventual

**Exemplo - Identidade do Cliente:**

- Criar um serviço de identidade central que gera IDs globais
- Cada contexto mantém seus próprios dados de cliente conforme sua necessidade
- Usar o ID global para referenciamento cruzado

### Visão Estratégica
A **visão estratégica** no DDD refere-se à abordagem de alto nível para modelar e organizar o sistema com base nos objetivos de negócio, permitindo decisões arquiteturais fundamentadas.

**Componentes da visão estratégica:**

1. **Mapa de Contexto (Context Map):** Uma representação visual de todos os Bounded Contexts e suas relações.
2. **Análise de Subdomínios:** Identificação e categorização dos subdomínios (Core, Supporting, Generic).
3. **Priorização Estratégica:** Concentrar recursos em subdomínios core onde a diferenciação competitiva é necessária.
4. **Arquitetura Evolutiva:** Projetar o sistema para evoluir incrementalmente conforme o entendimento do domínio amadurece.

**Benefícios:**

- Alinhamento entre negócio e tecnologia
- Melhor alocação de recursos técnicos
- Decisões mais claras sobre construir, comprar ou terceirizar
- Base para evolução sustentável do sistema

**Exemplo:** Uma seguradora pode identificar que:

- O cálculo de risco é um subdomínio core (necessita desenvolvimento interno especializado)
- O processamento de sinistros é um subdomínio de suporte (pode ser desenvolvido internamente com menos especialização)
- A contabilidade é um subdomínio genérico (pode usar software pronto de mercado)

### Context Mapping na Prática
É o processo de identificar Bounded Contexts e documentar as relações entre eles. É uma ferramenta essencial para gerenciar a complexidade em sistemas grandes.

 Processo de Context Mapping:

1. **Identificar Bounded Contexts:**
    - Analisar o domínio e identificar áreas coesas de funcionalidade
    - Prestar atenção a diferentes vocabulários usados por equipes distintas
    - Observar onde os modelos conceituais divergem
2. **Documentar Relacionamentos:**
    - Definir como os contextos interagem
    - Identificar dependências e fluxos de dados
    - Estabelecer padrões de integração
3. **Definir Estratégias de Implementação:**
    - Escolher padrões de integração adequados
    - Definir responsabilidades de tradução entre contextos
    - Estabelecer contratos e acordos de serviço

 Representação Visual:

Um mapa de contexto normalmente é representado por um diagrama que mostra:

- Bounded Contexts como caixas ou círculos
- Relações como linhas conectando os contextos
- Padrões de integração indicados por símbolos específicos
- Equipes responsáveis por cada contexto

### Padrões de Context Mapping e Starter Kit
 Padrões de Integração entre Bounded Contexts:

1. **Partnership (Parceria):**
    - Duas equipes têm uma relação de dependência mútua
    - Colaboração próxima para garantir que os contextos funcionem juntos
    - Exemplo: Sistema de Pedidos e Sistema de Pagamentos em um e-commerce
2. **Shared Kernel (Núcleo Compartilhado):**
    - Parte do modelo é compartilhada entre dois contextos
    - Requer coordenação e governança cuidadosa
    - Exemplo: Biblioteca comum de entidades de domínio básicas usada por múltiplos contextos
3. **Customer-Supplier (Cliente-Fornecedor):**
    - Um contexto (fornecedor) fornece serviços para outro (cliente)
    - O fornecedor considera as necessidades do cliente ao planejar
    - Exemplo: Serviço de Autenticação (fornecedor) e Portal de Usuários (cliente)
4. **Conformist (Conformista):**
    - Um contexto segue o modelo de outro sem influência
    - Geralmente quando não há poder de negociação
    - Exemplo: Integração com uma API externa que não pode ser modificada
5. **Anticorruption Layer (Camada Anticorrupção):**
    - Camada de tradução que protege um modelo de outro
    - Útil quando integrando com sistemas legados ou externos
    - Exemplo: Adaptador para um sistema legado com modelo de dados incompatível
6. **Open Host Service (Serviço de Hospedagem Aberto):**
    - Um contexto publica uma API bem definida para integração
    - Projetado para ser usado por múltiplos consumidores
    - Exemplo: Serviço de Pagamentos com API pública documentada
7. **Published Language (Linguagem Publicada):**
    - Formato de comunicação bem documentado entre contextos
    - Geralmente usando um esquema ou formato padrão
    - Exemplo: Esquema JSON ou XML para intercâmbio de dados entre serviços
8. **Separate Ways (Caminhos Separados):**
    - Decisão consciente de não integrar contextos
    - Evita complexidade quando a integração não é necessária
    - Exemplo: Sistema de Relatórios Financeiros e Sistema de Gerenciamento de Conteúdo que não precisam interagir


### DDD Starter Kit:

Para começar a aplicar DDD em um projeto, considere este kit inicial:
#### 1. Ferramentas para Análise de Domínio:

- **Event Storming:** Técnica colaborativa para mapear processos de negócios
- **Domain Storytelling:** Método para capturar histórias de domínio com notação visual
- **Example Mapping:** Técnica para descobrir regras de negócios através de exemplos

#### 2. Templates para Documentação:

- Modelo para documentação de Bounded Contexts
- Template para Context Map
- Formato padrão para Ubiquitous Language (Glossário)

#### 3. Padrões de Implementação:

- Estrutura básica para Aggregates, Entities e Value Objects
- Templates para Domain Events
- Implementações de referência para Repositories e Domain Services

#### 4. Práticas Recomendadas:

- Sessões regulares de modelagem com especialistas de domínio
- Revisões de modelo para refinamento contínuo
- Refatoração guiada por insights de domínio

#### 5. Métricas para Avaliação:

- Coerência do modelo (alinhamento com a linguagem do domínio)
- Eficácia das fronteiras de contexto (minimização de acoplamento)
- Flexibilidade para mudanças de requisitos


### Aplicando DDD Estratégico na Prática
A abordagem estratégica do Domain-Driven Design oferece um conjunto poderoso de ferramentas conceituais para gerenciar a complexidade em sistemas de software empresariais. Ao aplicar esses conceitos:

- **Comece com o Domínio, não com a Tecnologia:**
    - Invista tempo para entender o domínio antes de escolher tecnologias
    - Trabalhe proximamente com especialistas de domínio para criar um modelo significativo
- **Identifique Fronteiras Estratégicas:**
    - Reconheça onde diferentes modelos são necessários
    - Defina Bounded Contexts claros com responsabilidades específicas
- **Priorize Áreas Críticas:**
    - Foque recursos nos subdomínios core que trazem vantagem competitiva
    - Considere soluções mais simples para subdomínios genéricos
- **Documente Relacionamentos:**
    - Crie e mantenha um Context Map como artefato vivo
    - Use os padrões de integração para guiar a implementação
- **Evolua o Modelo:**
    - Refine continuamente o modelo à medida que o entendimento do domínio amadurece
    - Esteja aberto a refatorações quando novos insights surgirem

O DDD estratégico não é apenas sobre design de software, mas sobre criar uma linguagem compartilhada e um entendimento comum entre equipes técnicas e de negócios, permitindo que colaborem efetivamente na construção de sistemas que realmente atendem às necessidades do negócio.