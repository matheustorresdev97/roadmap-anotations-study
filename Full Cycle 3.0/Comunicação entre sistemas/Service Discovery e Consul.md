Service Discovery é um mecanismo que permite que serviços e aplicações em um ambiente distribuído encontrem uns aos outros automaticamente, sem precisar de configurações estáticas. Em essência, é um diretório que mantém registros de quais serviços estão disponíveis, onde estão localizados (endereço IP e porta) e como se comunicar com eles.

![[Captura de tela de 2025-03-27 17-57-21.png]]

Caracteristicas: 

- Descobre as máquinas disponíveis para acesso
- Segmentação de máquinas para garantir segurança
- Resoluções via DNS
- Health check
- Como saber se tenho permissão para acessar


### Hashicorp Consul

- Service Discovery
- Service Segmentation
- Load Balancer na Borda (Layer 7)
- Key / Value Configuration
- Opensource / Enterprise


![[Captura de tela de 2025-03-27 18-06-52.png]]

![[Captura de tela de 2025-03-27 18-09-16.png]]

![[Captura de tela de 2025-03-27 18-09-55.png]]

### Agent, Client e Server

- Agent: Processo que fica sendo executado em todos nós do cluster. Pode estar sendo executado em Client Mode ou Server Mode.
- Client: Registra os serviços localmente, Health check, encaminha as informações e configurações dos serviços para o Server.
- Server: Mantém o estado do cluster, registra os serviços, Membership (quem é client e quem é server), retorno de queries (DNS ou API), troca de informações entre datacenters, etc


Dev Mode
- NUNCA utilize em produção
- Teste de features, exemplos
- Roda como servidor
- Não escala
- Registra tudo em memória

### Iniciando um agente consul 
O Consul pode ser executado em três modos principais:
- **client**: Encaminha todas as consultas para um servidor, registra serviços locais
- **server**: Participa do consenso Raft, armazena e replica dados
- **dev**: Modo de desenvolvimento local (não recomendado para produção)

### Configuração Inicial com Docker
Vamos começar com um único contêiner Docker para testes:

```yaml
services:
	consul01:
		image: consul:1.10
		container_name: consul01
		hostname: consul01
		command: ['tail', '-f', '/dev/null']


```

Entre no contêiner e inicie o agente no modo dev para testes:
```bash
docker-compose up -d
docker exec -it consul01 sh

# Dentro do contêiner:
consul agent -dev
```

Para consultar os membros do cluster:
```shell
consul members
```

Saída esperada:
```shell
Node      Address         Status  Type    Build  Protocol  DC   Segment
consul01  127.0.0.1:8301  alive   server  1.10.0  2         dc1  <all>
```


### Criando nosso cluster
Crie um docker-compose mais completo para um ambiente mais realista:

```yaml
# docker-compose-cluster.yml
services:
  consul-server-1:
    image: consul:1.10
    container_name: consul-server-1
    hostname: consul-server-1
    ports:
      - "8500:8500"
    command: >
      consul agent -server -bootstrap-expect=3
      -node=consul-server-1
      -bind=0.0.0.0
      -client=0.0.0.0
      -ui
      -data-dir=/consul/data
    volumes:
      - ./server1:/consul/data

  consul-server-2:
    image: consul:1.10
    container_name: consul-server-2
    hostname: consul-server-2
    command: >
      consul agent -server -bootstrap-expect=3
      -node=consul-server-2
      -bind=0.0.0.0
      -client=0.0.0.0
      -join=consul-server-1
      -data-dir=/consul/data
    volumes:
      - ./server2:/consul/data
    depends_on:
      - consul-server-1

  consul-server-3:
    image: consul:1.10
    container_name: consul-server-3
    hostname: consul-server-3
    command: >
      consul agent -server -bootstrap-expect=3
      -node=consul-server-3
      -bind=0.0.0.0
      -client=0.0.0.0
      -join=consul-server-1
      -data-dir=/consul/data
    volumes:
      - ./server3:/consul/data
    depends_on:
      - consul-server-1
      - ```

Inicie o cluster:
```bash
docker-compose -f docker-compose-cluster.yml up -d
```

### Criando primeiro client
Adicione um cliente ao seu docker-compose:
```yaml
consul-client-1:
    image: consul:1.10
    container_name: consul-client-1
    hostname: consul-client-1
    command: >
      consul agent
      -node=consul-client-1
      -bind=0.0.0.0
      -client=0.0.0.0
      -join=consul-server-1
      -data-dir=/consul/data
    volumes:
      - ./client1:/consul/data
    depends_on:
      - consul-server-1
      - ```


### Registrando Serviço
Método 1: Via API HTTP
```bash
docker exec -it consul-client-1 sh

# Dentro do contêiner client:
curl -X PUT -d '{
  "ID": "web-app-1",
  "Name": "web-app",
  "Tags": ["v1", "production"],
  "Address": "10.0.0.10",
  "Port": 8080
}' http://localhost:8500/v1/agent/service/register
```

Método 2: Via arquivo de configuração
```json
// web-app.json
{
  "service": {
    "id": "web-app-1",
    "name": "web-app",
    "tags": ["v1", "production"],
    "address": "10.0.0.10",
    "port": 8080
  }
}
```

Coloque-o no diretório de configuração e reinicie o agente:
```bash
docker cp web-app.json consul-client-1:/consul/config/
docker exec -it consul-client-1 consul reload
```


### Registrando segundo serviço com retry join
Para maior resiliência, use `retry_join` em vez de apenas `join`:

```yaml
consul-client-2:
    image: consul:1.10
    container_name: consul-client-2
    hostname: consul-client-2
    command: >
      consul agent
      -node=consul-client-2
      -bind=0.0.0.0
      -client=0.0.0.0
      -retry-join=consul-server-1
      -retry-join=consul-server-2
      -retry-join=consul-server-3
      -data-dir=/consul/data
    volumes:
      - ./client2:/consul/data
      - ./client2-config:/consul/config
    depends_on:
      - consul-server-1
      - consul-server-2
      - consul-server-3
      - ```

Crie um arquivo de definição para o segundo serviço:

```json
// db-service.json
{
  "service": {
    "id": "db-1",
    "name": "database",
    "tags": ["mysql", "production"],
    "address": "10.0.0.20",
    "port": 3306
  }
}
```

### Implementando checks
Adicione verificações de saúde aos serviços:

```json
// web-app-with-check.json
{
  "service": {
    "id": "web-app-1",
    "name": "web-app",
    "tags": ["v1", "production"],
    "address": "10.0.0.10",
    "port": 8080,
    "check": {
      "id": "web-health",
      "name": "Web app health",
      "http": "http://10.0.0.10:8080/health",
      "interval": "10s",
      "timeout": "1s"
    }
  }
}
```

Para um check de TCP:

```json
// db-with-check.json
{
  "service": {
    "id": "db-1",
    "name": "database",
    "tags": ["mysql", "production"],
    "address": "10.0.0.20",
    "port": 3306,
    "check": {
      "id": "db-tcp",
      "name": "Database TCP check",
      "tcp": "10.0.0.20:3306",
      "interval": "10s",
      "timeout": "1s"
    }
  }
}
```

Também é possível adicionar multiple checks:

```json
{
  "service": {
    "id": "web-app-2",
    "name": "web-app",
    "checks": [
      {
        "id": "http-check",
        "http": "http://localhost:8080/health",
        "interval": "10s"
      },
      {
        "id": "disk-check",
        "args": ["/usr/local/bin/check-disk.sh"],
        "interval": "30s"
      }
    ]
  }
}
```

### Sincronizando servers via arquivo
Para uma configuração mais organizada, use arquivos de configuração em vez de parâmetros de linha de comando:

```json
// server.json
{
  "node_name": "consul-server-1",
  "server": true,
  "bootstrap_expect": 3,
  "data_dir": "/consul/data",
  "client_addr": "0.0.0.0",
  "ui_config": {
    "enabled": true
  },
  "retry_join": ["consul-server-2", "consul-server-3"],
  "bind_addr": "0.0.0.0"
}
```

Inicie o Consul com o diretório de configuração:

```bash
consul agent -config-dir=/consul/config
```


### Trabalhando com criptografia
Para ambientes de produção, é essencial habilitar a criptografia. Primeiro, gere chaves de criptografia:
```bash
consul keygen
# Saída: algo como "3TZTE4tEPk58XY5oZmFq3g=="
```

Configure a criptografia nos arquivos de configuração:

```json
// server-secure.json
{
  "node_name": "consul-server-1",
  "server": true,
  "bootstrap_expect": 3,
  "data_dir": "/consul/data",
  "client_addr": "0.0.0.0",
  "ui_config": {
    "enabled": true
  },
  "retry_join": ["consul-server-2", "consul-server-3"],
  "bind_addr": "0.0.0.0",
  "encrypt": "3TZTE4tEPk58XY5oZmFq3g==",
  "verify_incoming": true,
  "verify_outgoing": true,
  "verify_server_hostname": true,
  "ca_file": "/consul/config/ssl/ca.pem",
  "cert_file": "/consul/config/ssl/consul.pem",
  "key_file": "/consul/config/ssl/consul-key.pem"
}
```

Para gerar os certificados TLS:

```bash
# Gerar uma CA privada
openssl genrsa -out ca-key.pem 2048
openssl req -x509 -new -nodes -key ca-key.pem -days 1825 -out ca.pem

# Gerar certificados para os servidores
openssl genrsa -out consul-key.pem 2048
openssl req -new -key consul-key.pem -out consul.csr
openssl x509 -req -in consul.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out consul.pem -days 1825
```


### User Interface e dicas para Produção

A UI do Consul está disponível em:

```http://localhost:8500/ui/```

#### Dicas para Produção:

1. **Alta Disponibilidade**:
    - Use pelo menos 3-5 servidores para tolerância a falhas
    - Distribua servidores em zonas de disponibilidade diferentes
2. **Segurança**:
    - Sempre habilite ACLs em produção
    - Configure tokens de acesso com princípio de menor privilégio
    - Habilite TLS para toda comunicação
3. **Monitoramento**:
    - Configure alertas para falhas de nós
    - Monitore métricas importantes (uso de CPU, latência de Raft, etc.)
    - Use o Telemetry do Consul com Prometheus/Grafana
4. **Backup**:
    - Faça backup regular do diretório de dados
    - Considere usar o snapshot API para backups consistentes
5. **Configuração**:
    - Use variáveis de ambiente ou sistemas de gerenciamento de segredos para senhas/tokens
    - Mantenha a configuração em sistemas de controle de versão
6. **Governança**:
    - Implemente políticas para registro de serviços
    - Padronize as tags e os nomes de serviços


### Comandos úteis do Consul

```bash
# Listar membros
consul members

# Ver logs
consul monitor

# Verificar status de liderança
consul operator raft list-peers

# Verificar serviços registrados
consul catalog services

# Verificar nós
consul catalog nodes

# Verificar saúde do serviço
consul health service <nome-do-serviço>

# Fazer snapshot
consul snapshot save backup.snap

# DNS lookup (usando o DNS do Consul)
dig @127.0.0.1 -p 8600 <nome-do-serviço>.service.consul SRV
```


