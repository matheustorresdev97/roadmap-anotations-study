### Entendendo imagens e DockerHub

Repositório com imagens Docker
https://hub.docker.com/ 

`docker pull ubuntu`

Este comando baixa a imagem oficial do Ubuntu do #Docker-Hub (o repositório central de imagens Docker). Quando você executa

`docker pull ubuntu:20.04`

O Docker baixa a versão mais recente (latest) da imagem Ubuntu. Se quiser uma versão específica, você pode especificar a tag

`docker images`

Este comando lista todas as imagens Docker que estão armazenadas localmente em sua máquina:
A saída mostrará informações como:

- REPOSITORY: nome da imagem
- TAG: versão da imagem
- IMAGE ID: identificador único da imagem
- CREATED: quando a imagem foi criada
- SIZE: tamanho da imagem em disco


`docker rmi ubuntu`

Este comando remove uma imagem Docker do armazenamento local. Você pode referenciar a imagem pelo nome ou pelo ID:

Se houver contêineres usando esta imagem, você precisará remover esses contêineres primeiro ou usar a flag `-f` (force) para forçar a remoção.
Estas são operações fundamentais para gerenciar imagens Docker. O Docker Hub funciona como um repositório central onde você pode encontrar imagens oficiais e da comunidade, enquanto os comandos pull, images e rmi permitem gerenciar quais imagens estão disponíveis localmente em seu sistema.

### Criando primeira imagem com Dockerfile

Um Dockerfile é um arquivo de texto que contém instruções para construir uma imagem Docker. Ele define o ambiente, dependências, arquivos e comandos necessários para sua aplicação.

```Dockerfile
FROM nginx:latest

RUN apt-get update
RUN apt-get install vim -y

```
 
 `docker build -t nginx .`

Este comando constrói uma imagem Docker a partir de um Dockerfile:

- `docker build`: comando para criar uma imagem
- `-t nginx`: a flag `-t` atribui uma tag/nome à imagem (neste caso, "nginx")
- `.`: o ponto indica o contexto de build (o diretório atual onde está o Dockerfile)

O Docker lerá o Dockerfile no diretório atual e seguirá suas instruções para criar uma imagem personalizada chamada "nginx".

`docker run -it nginx bash`

Este comando cria e inicia um contêiner a partir da imagem "nginx" que você acabou de construir:

- `docker run`: comando para criar e iniciar um contêiner
- `-it`: combinação de duas flags:
    - `-i` (interativo): mantém o STDIN aberto
    - `-t` (terminal): aloca um pseudo-TTY
- `nginx`: nome da imagem que você quer executar
- `bash`: comando a ser executado dentro do contêiner (neste caso, um shell bash)

Este comando essencialmente inicia um contêiner baseado na sua imagem nginx e abre um terminal bash interativo dentro dele, permitindo que você navegue e interaja com o ambiente do contêiner.

### Avançando com Dockerfile

```Dockerfile

FROM nginx:latest

WORKDIR /app

RUN apt-get update && apt-get install -y

COPY html /usr/share/nginx/nginx/html

```

FROM nginx

- Esta é a instrução base do Dockerfile
- Ela define que a imagem será construída a partir da imagem oficial mais recente do Nginx
- O Nginx é um servidor web popular e de alta performance
- `:latest` indica que você está usando a versão mais recente disponível

WORKDIR /app

- Define o diretório de trabalho dentro do contêiner como `/app`
- Todos os comandos subsequentes como RUN, COPY, CMD serão executados neste diretório
- Se o diretório não existir, o Docker o criará automaticamente

RUN apt-get update && apt-get install -y

- Executa um comando durante a construção da imagem
- `apt-get update` atualiza a lista de pacotes disponíveis nos repositórios
- `apt-get install -y` está incompleto pois não especifica quais pacotes instalar
- A flag `-y` confirma automaticamente as solicitações de instalação
- Nota: Esta linha tem um problema, pois não especifica quais pacotes devem ser instalados

COPY html /usr/share/nginx/nginx/html

- Copia arquivos do contexto de build para dentro da imagem
- Neste caso, copia a pasta `html` do seu diretório local para `/usr/share/nginx/nginx` na imagem

### ENTRYPOINT vs CMD

Tanto ENTRYPOINT quanto CMD são instruções do Dockerfile que definem os comandos a serem executados quando um contêiner é iniciado, mas com diferenças importantes na forma como funcionam e quando devem ser usados.

ENTRYPOINT
O ENTRYPOINT define o comando principal que será **sempre** executado quando o contêiner iniciar.

**Características principais:**

- Define o executável base que não pode ser facilmente substituído
- Permanece fixo mesmo quando parâmetros adicionais são passados ao `docker run`
- Quaisquer argumentos passados ao `docker run` são adicionados ao final do ENTRYPOINT
- Pode ser sobrescrito apenas usando a flag `--entrypoint` durante o `docker run`

O ENTRYPOINT define o comando principal que será **sempre** executado quando o contêiner iniciar.


CMD
O CMD define comandos e/ou parâmetros padrão que podem ser facilmente substituídos.

**Características principais:**

- Fornece comandos e argumentos padrão
- É completamente substituído por quaisquer argumentos passados ao `docker run`
- Se usado junto com ENTRYPOINT, funciona como parâmetros padrão para o ENTRYPOINT


```Dockerfile
FROM ubuntu:latest

ENTRYPOINT ["echo", "Hello"]
CMD ["echo", "World"]

```

- `docker run minha-imagem` → imprime "Hello World"
- `docker run minha-imagem Docker` → imprime "Hello Docker" (CMD foi substituído)
- `docker run --entrypoint ls minha-imagem` → lista arquivos (ENTRYPOINT foi substituído)

O ENTRYPOINT é útil quando você quer criar um contêiner como um executável específico, enquanto o CMD é melhor quando quer flexibilidade para alterar os parâmetros ao iniciar o contêiner.


