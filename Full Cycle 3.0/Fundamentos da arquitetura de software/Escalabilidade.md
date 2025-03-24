É a capacidade de sistemas suportarem o aumento (ou a redução) dos workloads incrementando (ou reduzindo) o custo em menor ou igual proproção.

### Escalabilidade vs Perfomance
Enquanto performance tem o foco em reduzir a latência e aumentar o throughput, a escalabilidade visa termos a possibilidade de aumentar ou diminuir o throughput adicionando ou removendo a capacidade computacional.

![[Captura de tela de 2025-03-24 19-05-47.png]]

### Escalando software - Descentralização

![[Captura de tela de 2025-03-24 19-08-04.png]]

### Escala de banco de dados

- Aumentando recursos computacionais
- Distribuindo responsabilidades (escrita vs leitura)
- Shards de forma horizontal
- Serverless

Otimização de queries e índices
- Trabalho com índices de forma consciente
- APM (Application performance monitoring) nas queries
- Explain na queries
- CQRS (Command Query Responsibility Segregation)

### Proxy reverso
Um proxy reverso é um servidor que fica na frente dos servidores web e encaminha as solicitações do cliente (por exemplo, navegador web) para esses servidores web.

![[Captura de tela de 2025-03-24 19-14-17.png]]


Soluções populares de Proxy Reverso

- #Nginx
- #HAProxy (HA = High Availability)
- #Traefik