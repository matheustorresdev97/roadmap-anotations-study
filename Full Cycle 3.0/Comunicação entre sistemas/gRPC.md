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

### Instalando compilador e plugins
- **protoc**: O compilador Protocol Buffers que gera código a partir dos arquivos .proto
- **Plugins específicos da linguagem**: Para gerar código cliente e servidor na linguagem desejada (como grpc-go, grpc-java, etc.)
documentação oficial do gRPC -> https://grpc.io/
documentação oficial do Protocol Buffers -> https://protobuf.dev/

Como utilizar 
No caso de Go, um exemplo de instalação seria:
```bash
# Instalar protoc
$ apt-get install -y protobuf-compiler
# ou
$ brew install protobuf

# Instalar plugins Go
$ go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
$ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

### Fazendo setup do projeto

Para configurar um projeto gRPC, você precisa:

1. Criar a estrutura de diretórios do projeto (cmd, pb, services, etc.)
2. Inicializar o módulo Go (se estiver usando Go)
3. Adicionar as dependências necessárias, como:
    - google.golang.org/grpc
    - google.golang.org/protobuf

### Criando nosso protofile
O arquivo .proto define a interface do seu serviço. Exemplo para um serviço de categorias:

```protobuf
syntax = "proto3";
package pb;
option go_package = "./pb";

message Category {
  string id = 1;
  string name = 2;
  string description = 3;
}

message CreateCategoryRequest {
  string name = 1;
  string description = 2;
}

message CategoryResponse {
  Category category = 1;
}

service CategoryService {
  rpc CreateCategory(CreateCategoryRequest) returns (CategoryResponse) {}
}
```

### Fazendo geração de código com protoc
Após definir o arquivo .proto, você gera o código usando o protoc:

```bash
protoc --proto_path=proto proto/*.proto \
  --go_out=. \
  --go-grpc_out=.
  ```
Isso vai gerar:

- Arquivos com estruturas de dados (.pb.go)
- Arquivos com interfaces e stubs de cliente/servidor (.grpc.pb.go)

### Implementando createCategory

Agora você implementa o serviço definido no .proto:

```go
package services

import (
    "context"
    "github.com/seu-username/seu-projeto/pb"
    "github.com/google/uuid"
)

type CategoryService struct {
    pb.UnimplementedCategoryServiceServer
    categories []*pb.Category
}

func NewCategoryService() *CategoryService {
    return &CategoryService{}
}

func (c *CategoryService) CreateCategory(ctx context.Context, in *pb.CreateCategoryRequest) (*pb.CategoryResponse, error) {
    category := &pb.Category{
        Id:          uuid.New().String(),
        Name:        in.GetName(),
        Description: in.GetDescription(),
    }
    
    c.categories = append(c.categories, category)
    
    return &pb.CategoryResponse{
        Category: category,
    }, nil
}
```

### Criando servidor gRPC
Agora você cria o servidor gRPC:

```go
package main

import (
    "log"
    "net"
    
    "github.com/seu-username/seu-projeto/pb"
    "github.com/seu-username/seu-projeto/services"
    "google.golang.org/grpc"
    "google.golang.org/grpc/reflection"
)

func main() {
    lis, err := net.Listen("tcp", ":50051")
    if err != nil {
        log.Fatalf("Failed to listen: %v", err)
    }
    
    grpcServer := grpc.NewServer()
    categoryService := services.NewCategoryService()
    pb.RegisterCategoryServiceServer(grpcServer, categoryService)
    reflection.Register(grpcServer)
    
    log.Println("Starting gRPC server on port 50051")
    if err := grpcServer.Serve(lis); err != nil {
        log.Fatalf("Failed to serve: %v", err)
    }
}
```

### Interagindo com Evans
Evans é um cliente gRPC interativo que permite testar serviços gRPC:

```bash
# Instalar Evans
$ brew install evans

# Conectar ao servidor gRPC
$ evans --host localhost --port 50051 -r repl

# Chamar um método
evans> call CreateCategory
```

### Criando categoryList no protofile
Adicione o método de listagem no arquivo .proto:

```protobuf
message Empty {}

message CategoryList {
  repeated Category categories = 1;
}

service CategoryService {
  // Métodos existentes
  rpc CreateCategory(CreateCategoryRequest) returns (CategoryResponse) {}
  
  // Novo método
  rpc ListCategories(Empty) returns (CategoryList) {}
}
```

### Listando categories
Implemente o método de listagem:

```go
func (c *CategoryService) ListCategories(ctx context.Context, in *pb.Empty) (*pb.CategoryList, error) {
    return &pb.CategoryList{
        Categories: c.categories,
    }, nil
}
```

### Buscando uma categoria
Adicione ao .proto:
```protobuf
message GetCategoryRequest {
  string id = 1;
}

service CategoryService {
  // Métodos existentes
  
  // Novo método
  rpc GetCategory(GetCategoryRequest) returns (CategoryResponse) {}
}
```

Implementação:

```go
func (c *CategoryService) GetCategory(ctx context.Context, in *pb.GetCategoryRequest) (*pb.CategoryResponse, error) {
    for _, cat := range c.categories {
        if cat.Id == in.GetId() {
            return &pb.CategoryResponse{Category: cat}, nil
        }
    }
    
    return nil, status.Error(codes.NotFound, "Category not found")
}
```


### Trabalhando com stream
gRPC suporta streaming, permitindo enviar múltiplas mensagens em uma única chamada. Adicione ao .proto:

```protobuf
service CategoryService {
  // Métodos existentes
  
  // Streaming do servidor para o cliente
  rpc CreateCategoryStream(CreateCategoryRequest) returns (stream CategoryResponse) {}
}
```

Implementação:

```go
func (c *CategoryService) CreateCategoryStream(in *pb.CreateCategoryRequest, stream pb.CategoryService_CreateCategoryStreamServer) error {
    // Cria categoria principal
    category := &pb.Category{
        Id:          uuid.New().String(),
        Name:        in.GetName(),
        Description: in.GetDescription(),
    }
    
    // Envia a categoria principal
    stream.Send(&pb.CategoryResponse{
        Category: category,
    })
    
    // Envia algumas categorias relacionadas
    for i := 0; i < 5; i++ {
        relatedCategory := &pb.Category{
            Id:          uuid.New().String(),
            Name:        fmt.Sprintf("Related to %s - %d", in.GetName(), i),
            Description: "Related category",
        }
        
        stream.Send(&pb.CategoryResponse{
            Category: relatedCategory,
        })
        
        time.Sleep(time.Second) // Simula processamento
    }
    
    return nil
}
```


### Trabalhando com streams bidirecionais
Streams bidirecionais permitem que cliente e servidor enviem múltiplas mensagens. Adicione ao .proto:

```protobuf
service CategoryService {
  // Métodos existentes
  
  // Stream bidirecional
  rpc CreateCategoryStreamBidirectional(stream CreateCategoryRequest) returns (stream CategoryResponse) {}
}
```

Implementação:

```go
func (c *CategoryService) CreateCategoryStreamBidirectional(stream pb.CategoryService_CreateCategoryStreamBidirectionalServer) error {
    for {
        req, err := stream.Recv()
        if err == io.EOF {
            return nil
        }
        if err != nil {
            return err
        }
        
        category := &pb.Category{
            Id:          uuid.New().String(),
            Name:        req.GetName(),
            Description: req.GetDescription(),
        }
        
        c.categories = append(c.categories, category)
        
        err = stream.Send(&pb.CategoryResponse{
            Category: category,
        })
        if err != nil {
            return err
        }
    }
}
```

Com isso, você terá um serviço gRPC completo com suporte a diferentes tipos de comunicação: unária, streaming do servidor, streaming do cliente e streaming bidirecional.