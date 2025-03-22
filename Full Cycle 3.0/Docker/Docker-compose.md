O #Docker-Compose é uma ferramenta que permite definir e executar aplicações Docker com múltiplos contêineres.
O Docker Compose é uma ferramenta para definir e executar aplicações Docker com múltiplos contêineres. Com o Compose, você usa um arquivo YAML para configurar os serviços, redes e volumes da sua aplicação. Depois, com um único comando, você cria e inicia todos os serviços a partir da sua configuração.

**Crie um arquivo docker-compose.yml** - Este é o arquivo de configuração principal onde você define seus serviços.
**Defina seus serviços** - Cada serviço representa um contêiner na sua aplicação.
**Execute docker-compose up** - Este comando constrói, cria e inicia todos os serviços definidos no seu arquivo docker-compose.yml.

```Yaml
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
  
  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE: myapp
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
  ```
Este exemplo define dois serviços:

- Um serviço web usando a imagem do Nginx
- Um serviço de banco de dados usando MySQL

### Comandos Básicos

- `docker-compose up` - Cria e inicia os contêineres
- `docker-compose up -d` - Executa os contêineres em segundo plano
- `docker-compose down` - Para e remove os contêineres e redes
- `docker-compose logs` - Visualiza a saída dos contêineres
- `docker-compose ps` - Lista os contêineres em execução
- `docker-compose build` - Constrói ou reconstrói os serviços

### Buildando Imagens com Docker Compose e Configurando Aplicação Node.js com Postgres

Quando você define serviços no Docker Compose, pode construir imagens de duas maneiras:
**Usando imagens existentes**:

```yaml
services:
  web:
    image: nginx:alpine]


```

**Construindo a partir de um Dockerfile**:

```yaml
services:
  app:
    build: ./app
    # Ou com mais opções
    build:
      context: ./app
      dockerfile: Dockerfile.dev
      args:
        ENV: development

```


### Criando DB Postgres

```yaml
services:
  postgres:
    image: postgres:14
    container_name: postgres-db
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: nodeapp
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network

volumes:
  postgres-data:

networks:
  app-network:
    driver: bridge

```

- Utiliza a imagem oficial do Postgres
- Define variáveis de ambiente para nome de usuário, senha e nome do banco de dados
- Mapeia a porta 5432 para acessar o banco de dados
- Cria um volume para persistência dos dados
- Conecta o serviço à rede definida


### Configurando App Node com Docker Compose
No Docker Compose, a dependência entre containers é gerenciada principalmente pelo atributo `depends_on`:
Isso garante que o container do Postgres seja iniciado antes do container do Node.js. No entanto, é importante entender que o `depends_on` apenas controla a ordem de inicialização, não garante que o serviço esteja pronto para aceitar conexões.

Para lidar com isso adequadamente, sua aplicação Node.js deve:

1. Implementar lógica de retry para conectar ao banco de dados
2. Ou usar ferramentas como o wait-for-it ou dockerize para aguardar o serviço estar disponível

```yaml

services:
  app:
    build:
      context: ./app
      dockerfile: Dockerfile
    container_name: node-app
    restart: always
    environment:
      DB_HOST: postgres
      DB_PORT: 5432
      DB_USER: postgres
      DB_PASSWORD: postgres
      DB_NAME: nodeapp
    volumes:
      - ./app:/usr/src/app
      - /usr/src/app/node_modules
    depends_on:
      - postgres
    networks:
      - app-network

  postgres:
    image: postgres:14
    container_name: postgres-db
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: nodeapp
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network

volumes:
  postgres-data:

networks:
  app-network:
    driver: bridge
```

