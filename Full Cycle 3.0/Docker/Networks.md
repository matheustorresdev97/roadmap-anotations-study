As redes no Docker são componentes fundamentais que permitem a comunicação entre contêineres e entre contêineres e o mundo exterior. Vamos explorar os diferentes tipos de redes e como utilizá-las.
### Entendendo tipos de Network

O Docker possui vários tipos de redes embutidos:

- **Bridge**: O driver de rede padrão. Se você não especificar um driver, este é o tipo criado. É uma rede interna isolada onde os contêineres podem se comunicar.
- **Host**: Remove o isolamento de rede entre o contêiner e o host, usando diretamente a rede do host.
- **None**: Desativa todas as redes. Útil para contêineres que não precisam de conexão de rede.
- **Overlay**: Permite que contêineres em diferentes hosts Docker se comuniquem. Geralmente usado com Docker Swarm.
- **Macvlan**: Permite atribuir um endereço MAC a um contêiner, fazendo-o aparecer como um dispositivo físico na rede.

Para listar as redes disponíveis

`docker network ls`

Para listar uma rede

`docker network inspect [nome-da-rede]`

### Trabalhando com bridge

A rede bridge é a mais comum e é a padrão do Docker. Quando você executa um contêiner sem especificar uma rede, ele se conecta à rede bridge padrão.

**Características da rede bridge:**

- Contêineres na mesma rede bridge podem se comunicar
- Fornece NAT (Network Address Translation) para conexões externas
- Cada contêiner recebe um IP interno

**Comandos importantes:**

Criar uma nova rede bridge personalizada:

`docker network create --driver bridge minha-rede`

Conectar um contêiner a uma rede específica:

`docker run --network=minha-rede nginx`

Conectar um contêiner existente a uma rede:

`docker network connect minha-rede meu-container`

**Vantagem de redes bridge personalizadas sobre a padrão:**

- Resolução de DNS automática entre contêineres (pelo nome)
- Melhor isolamento
- Possibilidade de conectar/desconectar contêineres em execução


### Trabalhando com host

A rede host remove o isolamento de rede entre o contêiner e o host Docker. Um contêiner na rede host usa diretamente a pilha de rede do host.

**Características da rede host:**

- Melhor desempenho (sem overhead de NAT)
- O contêiner usa as interfaces de rede do host
- Nenhum mapeamento de porta é necessário

Exemplo de uso:

`docker run --network=host nginx`
Neste exemplo, o Nginx estará disponível diretamente na porta 80 do host, sem necessidade de mapeamento de porta.

**Observações importantes:**

- Não há isolamento de portas (possíveis conflitos de porta)
- Se um contêiner escuta na porta 80, nenhum outro pode usar essa porta
- Disponível apenas para hosts Linux (não funciona da mesma forma no Windows/Mac)

### Container acessando nossa máquina

Quando um contêiner precisa acessar serviços rodando no host Docker, existem algumas abordagens:

1. **Usando a rede host** (já discutido acima)
2. **Usando o endereço IP especial:**
    - Em Linux: use `host.docker.internal` ou `172.17.0.1` (geralmente o IP gateway da bridge)
    - No Docker Desktop (Mac/Windows): use `host.docker.internal` que resolve para o IP do host

3. Mapeamento de portas bidirecional:
`docker run -p 8080:80 nginx`

- Isto mapeia a porta 8080 do host para a porta 80 do contêiner, mas a comunicação é possível em ambas as direções.

3. **Usando redes específicas:** Em alguns casos, pode ser útil criar uma rede personalizada que facilite a comunicação contêiner-host.

