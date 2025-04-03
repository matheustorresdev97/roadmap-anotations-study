A modelagem tática no Domain-Driven Design (DDD) refere-se às técnicas e padrões específicos para implementar modelos de domínio eficazes. Enquanto o DDD estratégico foca na visão geral e na organização do sistema em bounded contexts, a modelagem tática concentra-se na implementação detalhada dentro de cada contexto.
### Elementos táticos
Quando estamos falando sobre DDD e precisamos olhar mais a fundo um bounded context. Precisamos ser capazes de modelarmos de forma mais assertiva os seus principais componentes, comportamentos e individualidades, bem como suas relações.

### Ressignificando Conceitos Tradicionais

O DDD tático ressignifica diversos conceitos que já existiam em desenvolvimento de software:

1. **Da Orientação a Objetos "tradicional" para Modelagem Rica**:
    - **Antes**: Classes eram frequentemente vistas como estruturas de dados com comportamentos.
    - **Com DDD**: Objetos são representações ricas de conceitos de domínio, com responsabilidades claras e bem encapsuladas.
2. **Da Persistência para Modelo de Domínio**:
    - **Antes**: O design era frequentemente dirigido pelo banco de dados (anêmico).
    - **Com DDD**: O modelo de domínio é independente da persistência, com comportamentos ricos dentro das entidades e value objects.
3. **De Transações ACID para Consistência Eventual**:
    - **Antes**: Sistemas dependiam fortemente de transações ACID para manter consistência.
    - **Com DDD**: Conceitos como agregados e eventos de domínio permitem gerenciar consistência em sistemas distribuídos.
4. **De APIs para Contratos entre Bounded Contexts**:
    - **Antes**: APIs eram vistas principalmente como interfaces técnicas.
    - **Com DDD**: Interfaces entre contextos são contratos significativos de domínio.

### Por que Modelagem Tática é Importante

1. **Expressividade**: Produz um código que reflete claramente o modelo mental do domínio.
2. **Encapsulamento**: Protege a integridade do modelo de domínio.
3. **Testabilidade**: Facilita testes de unidade e de integração focados em comportamentos de domínio.
4. **Colaboração**: Fornece uma linguagem comum entre desenvolvedores e especialistas de domínio.
5. **Evolução**: Permite que o sistema evolua à medida que o entendimento do domínio amadurece.