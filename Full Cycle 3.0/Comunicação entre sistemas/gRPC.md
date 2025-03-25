- #gRPC é um framework desenvolvido pelo google que tem o objetivo de facilitar o processo de comunicação entre sistemas de uma forma extremamente rápida, leve, independente de linguagem.
- Faz parte da CNCF (Cloud Native Computing Foundation)

Em quais casos podemos utilizar? 
- Ideal para microsserviços
- Mobile, Browsers e Backend
- Geração das bibliotecas de forma automática
- Streaming bidirecional utilizando HTTP/2

Linguagens 
- gRPC-GO
- gRPC-JAVA
- gRPC-C
	- C++
	- Python
	- Node.js

 RPC - Remote Procedure Call

![[Captura de tela de 2025-03-25 09-07-03.png]]

### Protocol Buffers
"Protocol buffers are Google language-neutral, platform-neutral, extensible mechanism for serializing structured data - think XML, but smaller, faster, and simpler."

Protocol Buffers vs JSON

- Arquivos binários < JSON
- Processo de serialização é mais leve (CPU) do que JSON
- Gasta menos recursos de rede
- Processo é mais veloz

```proto
syntax = "proto3"

message SearchRequest {
	string query = 1;
	int32 page_number = 2;
	int32 result_per_page = 3;
}
```

### HTTP/2
- Nome original criado pela Google era SPDY
- Lançado em 2015
- Dados trafegados são binários e não texto como no HTTP 1.1
- Utiliza a mesma conexão TCP para enviar e receber dados do cliente e do servidor (Multiplex)
- Server Push
- Headers são comprimidos
- Gasta menos recursos de rede
- Processo é mais veloz

### API "unary"

![[Captura de tela de 2025-03-25 09-19-26.png]]

### API "Server streaming" 

![[Captura de tela de 2025-03-25 09-20-37.png]]

### API "Client streaming"

![[Captura de tela de 2025-03-25 09-21-29.png]]

### API "BI directional streaming"

![[Captura de tela de 2025-03-25 09-22-29.png]]


| Rest                                         | gRPC                       |
| -------------------------------------------- | -------------------------- |
| Texto / JSON                                 | Protocol Buffers           |
| Unidirecional                                | Bidirecional e Assíncrono  |
| Alta latência                                | Baixa latência             |
| Sem contrato (maior chance de erros)         | Contrato definido (.proto) |
| Sem suporte a streaming (Request / Response) | Suporte a streaming        |
| Design pré-definido                          | Design é livre             |
| Bibiliotecas de terceiro                     | Geração de código          |

documentação oficial do gRPC -> https://grpc.io/
documentação oficial do Protocol Buffers -> https://protobuf.dev/

Como utilizar 
- Instale os compiladores e plugins da documentação oficial

