- Resiliência é um conjunto de estratégias adotadas intencionalmente para a adaptação de um sistema quando uma falha ocorre.
- Ter estratégias de resiliência nos possibilita minimizar os riscos de perda de dados e transações importantes para o negócio.

### Proteger e ser Protegido

- Um sistema em uma arquitetura distribuída precisa adotar mecanismos de autopreservação para garantir ao máximo sua operação com qualidade.
- Um sistema precisa não pode ser "egoísta" ao ponto de ealizar mais requisições em um sistema que está falhando.
- Um sistema lento no ar muitas vezes é pior do que um sistema fora do ar. (Efeito dominó)


### Health check

- Sem sinais vitais, não é possivel saber a "saúde" de um sistema
- Um sistema que não está saudável possui uma chance de se recuperar caso o tráfego pare de ser direcionado a ele temporariamente.
- Health check de qualidade

### Rate limiting

- Protege o sistema baseado no que ele foi projetado para suportar
- Preferência programada por tipo de client

### Circuit breaker

- Protege o sistema fazendo com que as requisiçoes feitas para ele sejam negadas. Ex: 500
- Circuito fechado = Requisições chegam normalmente
- Circuito aberto = Requisições não chegam ao sistema. Erro instantâneo ao client
- Meio aberto = Permite uma quantidade limitada de requisições para verificação se o sistema tem condições de voltar ao ar integralmente

### API Gateway

- Garante que requisições "inapropriadas" cheguem até o sistema:
 Ex: usuário não autenticado
 - Implementa políticas de Rate Limiting, Health check, etc
### Service Mesh

- Controla o tráfego de rede
- Evita implementações de proteção pelo próprio sistema
- mTLS
- Circuit breaker, retry, timeout, fault injection, etc


### Comunicação assíncrona

- Evita perda de dados 
- Não há perda de dados no envio de uma transação de o server estiver fora
- Servidor pode processar a transação em seu tempo quando estiver online
- Entender com profundidade o message broker / sistema de stream

### Garantias de entrega com #Kafka

![[Captura de tela de 2025-03-24 19-33-10.png]]

### Situações complexas

- O que acontece se o message broker cair?
- Haverá perda de mensagens?
- Seu sistema ficará fora do ar?
- Como garantir resiliência?