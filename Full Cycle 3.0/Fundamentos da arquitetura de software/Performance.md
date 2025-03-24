
- É o desempenho que um software possui para completar um determinado workload
- As unidades de medida para avaliarmos a performance de um software são
	- Latência ou "response time"
	- Throughput
- Ter um software performático é diferente de ter um software escalável

### Métricas para medir a performance
- Diminuir a latẽncia
	- Normalmente medida em miliseconds
	- É afetada pelo tempo de processamento da aplicação, rede e chamadas externas
- Aumentando throughput
	- Quantidade de requisições
	- Diretamente ligado a latência

### Principais razões para baixa performance
- Processamento ineficiente
- Recursos computacionais limitados
- Trabalhar de forma bloqueante
- Acesso serial a recursos

### Principais formas para aumentar a eficiência
- Escala da capacidade computacional (CPU, Disco, Memória, Rede)
- Lógica por trás do software (Algoritmos, queries, overhead de frameworks)
- Concorrência e paralelismo
- Banco de dados (tipos de bancos, schema)
- Caching

### Capacidade computacional:
Escala vertical vs horizontal

![[Captura de tela de 2025-03-24 10-06-04.png]]

### Diferença entre concorrência e paralelismo

"Concorrência é sobre lidar com muitas coisas ao mesmo tempo. Paralelismo é fazer muitas coisas ao mesmo tempo." Rob Pike

![[Captura de tela de 2025-03-24 10-09-51.png]]

![[Captura de tela de 2025-03-24 10-10-14.png]]

### Caching
- Cache na borda / Edge computing
- Dados estáticos
- Páginas web
- Funções internas
	- Evita reprocessamento de algoritmos pesados
	- Acesso ao banco de dados
- Objetos

Exclusivo vs Compartilhado

Exclusivo
- Baixa latência
- Duplicado entre os nós
- Problemas relacionados a sessões

Compartilhado
- Maior latência
- Não há duplicação
- Sessões compartilhadas
- Banco de dados externo
	- MySQL
	- Redis
	- Memcache

 Edge computing
 - Cache realizado mais próximo ao usuário
 - Evita a requisição chegar até o Cloud Provider / Infra
 - Normalmente arquivos estáticos
 - CDN - Content Delivery Network
 - Cloudflare workers
 - Vercel
 - Akamai

