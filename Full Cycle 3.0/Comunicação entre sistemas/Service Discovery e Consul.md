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
- Teste de feeatures, exemplos
- Roda como servidor
- Não escala
- Registra tudo em memória

### Iniciando um agente consul 
(podemos subir ele no modo client, server ou dev)

```yaml
services:
	consul01:
		image: consul:1.10
		container_name: consul01
		hostname: consul01
		command: ['tail', '-f', '/dev/null']


```

rode e entre no container
```bash
execute: consul agent -dev
```

Para consultar todos os cluster
```shell
consul members
```

