
### Hello World

`docker ps` 
Este comando lista todos os containers Docker que estão **atualmente em execução** no seu sistema. Ele mostra informações como:

- ID do container
- Imagem utilizada
- Comando sendo executado
- Quando foi criado
- Status atual
- Portas expostas
- Nome do container

`docker run hello-world` 
1. Procura localmente a #imagem chamada "hello-world"
2. Se não encontrar, baixa automaticamente essa imagem do Docker Hub (repositório oficial)
3. Cria um novo #container a partir dessa imagem
4. Inicia o container, que executa um pequeno programa que exibe uma mensagem de boas-vindas
5. O container termina sua execução após exibir a mensagem

A imagem "hello-world" é frequentemente utilizada como primeiro teste para verificar se a instalação do Docker está funcionando corretamente. Quando executada com sucesso, ela mostra uma mensagem confirmando que sua instalação do Docker está funcionando e explica brevemente os passos que o #Docker executou para rodar o containers

`docker ps -a` 
Lista todos os containers Docker no seu sistema, incluindo os containers que estão parados, e não apenas os que estão em execução. Quando você executa este comando, o Docker mostra uma tabela com as seguintes informações para cada container:

- **CONTAINER ID**: O identificador único do container
- **IMAGE**: A imagem usada para criar o container
- **COMMAND**: O comando que está sendo executado dentro do container
- **CREATED**: Quando o container foi criado
- **STATUS**: O estado atual do container (running, exited, paused, etc.)
- **PORTS**: Quaisquer mapeamentos de porta configurados
- **NAMES**: O nome atribuído ao container

### Executando Ubuntu

`docker run -it ubuntu bash`

- **docker run**: Instrui o Docker a criar e iniciar um novo container.
- **-it**: São duas flags combinadas:
    - `-i` (interactive): Mantém o STDIN aberto, permitindo que você envie comandos para o container.
    - `-t` (tty): Aloca um pseudo-terminal, fornecendo uma experiência de terminal interativo.
- **ubuntu**: É o nome da imagem que será usada como base para o container. Neste caso, é a imagem oficial do Ubuntu Linux.
- **bash**: É o comando que será executado dentro do container assim que ele iniciar. Neste caso, o shell Bash será iniciado.

- O Docker verifica se a imagem Ubuntu já existe localmente no seu sistema.
- Se não existir, ele baixa a versão mais recente (tag `latest`) da imagem Ubuntu do Docker Hub.
- Cria um novo container baseado nessa imagem.
- Inicia o container.
- Executa o comando `bash` dentro do container.
- Devido às flags `-it`, você terá acesso interativo ao shell Bash rodando dentro do container.
- Você verá um prompt de comando do Ubuntu dentro do container, onde poderá executar comandos como se estivesse em uma máquina Ubuntu.
Este comando é frequentemente usado para "entrar" em um container Ubuntu e interagir com ele através da linha de comando, possibilitando instalar pacotes, editar arquivos ou testar aplicações dentro do ambiente isolado do container.


`docker start [nome-id]` 
Este comando inicia um container que foi previamente parado (stopped). Ele funciona da seguinte forma:
1. Localiza o container pelo nome ou ID fornecido.
2. Inicializa o sistema de arquivos do container.
3. Reinicia o processo principal definido na configuração do container.
4. Conecta o container às redes definidas.
5. Retorna o container ao estado "running".

O container mantém todas as configurações, volumes e dados que existiam antes de ser parado.


`docker stop [nome-id]` 
Este comando para um container em execução de forma segura:

1. Envia um sinal SIGTERM para o processo principal no container, pedindo para ele finalizar normalmente.
2. Aguarda até 10 segundos (por padrão) para o container encerrar graciosamente.
3. Se não encerrar no tempo limite, envia um sinal SIGKILL, forçando o encerramento imediato.
4. Muda o estado do container para "stopped" ou "exited".

Quando um container é parado, ele mantém seus dados e configurações, podendo ser iniciado novamente com `docker start`.

A diferença principal entre parar e remover um container é que ao parar, o container e seus dados permanecem no sistema, enquanto remover (`docker rm`) elimina completamente o container e seus dados não persistentes.

Exemplo:

`docker run -it --rm ubuntu bash`

Ao parar o container com docker stop o container e removido


### Publicando portas com nginx

`docker run nginx`

Quando você executa apenas `docker run nginx`, o Docker:
1. Busca a imagem #nginx (baixa do Docker Hub se não estiver disponível localmente)
2. Cria um novo container baseado nessa imagem
3. Inicia o container, que executa o servidor Nginx dentro dele
4. O container recebe seu próprio namespace de rede isolado
5. O Nginx está rodando na porta 80, mas **apenas dentro do container**
6. Não há mapeamento entre portas do container e do host, então você não consegue acessar via localhost:80

`docker run -p 8080:80 nginx`
Já este comando faz o mesmo que o anterior, com um passo crucial adicional:

1. Busca a imagem nginx
2. Cria um novo container
3. Configura um mapeamento de portas usando a flag `-p 8080:80`, que:
    - Mapeia a porta 8080 do host (seu computador)
    - Para a porta 80 do container (onde o Nginx está escutando)
4. Inicia o container com esse mapeamento de porta ativo
5. Agora você pode acessar o Nginx através de `localhost:8080` no seu navegador

Por que o segundo funcionou ?

O formato do mapeamento de portas é: `-p [PORTA_HOST]:[PORTA_CONTAINER]`
No seu caso:

- `8080` é a porta no seu computador (host) que você usa para acessar o serviço
- `80` é a porta dentro do container onde o Nginx está rodando

Este mapeamento cria uma ponte entre os dois ambientes de rede, permitindo que você acesse o serviço isolado do container a partir do seu computador.

Sem esse mapeamento explícito de portas, o container está isolado na rede Docker interna e não é acessível a partir do host, mesmo que o serviço esteja funcionando corretamente dentro do container.

`docker run -dp 8080:80 nginx`

Este comando combina duas flags:

1. `-d` (detached mode): Executa o container em segundo plano (background). O container é iniciado e o comando retorna imediatamente ao prompt, sem mostrar a saída do container. O container continua rodando "desacoplado" do seu terminal.
2. `-p 8080:80`: Mapeia a porta 8080 do host para a porta 80 do container, assim como explicado anteriormente.

Passo a passo de execução:

1. Docker verifica se a imagem nginx está disponível localmente (baixa se necessário)
2. Cria um novo container baseado na imagem nginx
3. Configura o mapeamento da porta 8080 do host para a porta 80 do container
4. Inicia o container em modo background (detached)
5. Retorna o controle para o seu terminal imediatamente, mostrando apenas o ID do container criado
6. O Nginx continua rodando no container em segundo plano
7. Você pode acessar o Nginx através de `localhost:8080` no navegador

Diferença do comando anterior:

A principal diferença do comando anterior (`docker run -p 8080:80 nginx`) é o modo de execução:

- Sem `-d`: O container roda em modo "attached" (vinculado ao terminal). Você vê os logs em tempo real, e o comando não retorna até que o container seja encerrado. Se você fechar o terminal, o container para.
- Com `-d`: O container roda em modo "detached" (desvinculado). Ele continua rodando independentemente do terminal, como um processo em segundo plano. É útil para serviços que precisam continuar rodando enquanto você faz outras coisas.

O modo detached (`-d`) é geralmente usado para serviços como o Nginx que devem continuar rodando por longos períodos.

### Removendo containers

`docker rm nome-do-container`

Este comando remove um container que está parado:

1. Identifica o container pelo nome ou ID fornecido
2. Verifica se o container está parado
3. Se o container estiver em execução, o comando falha com uma mensagem de erro
4. Se o container estiver parado, o Docker:
    - Remove o sistema de arquivos do container
    - Remove os metadados do container do sistema
    - Libera os recursos associados ao container

Este comando só funciona em containers parados, pois é considerado mais seguro evitar a remoção acidental de containers em execução.

 `docker rm -f nome-do-container`

A adição da flag `-f` (force/forçar) modifica o comportamento:

1. Identifica o container pelo nome ou ID fornecido
2. Independente do estado do container (rodando ou parado):
    - Envia um sinal SIGKILL para forçar a parada imediata do container
    - Não espera pelo encerramento normal dos processos
    - Remove o sistema de arquivos do container
    - Remove os metadados do container do sistema
    - Libera os recursos associados ao container

O `-f` permite remover containers em execução sem precisar pará-los primeiro com `docker stop`, mas deve ser usado com cuidado, pois:

- Não permite que os processos dentro do container se encerrem normalmente
- Pode resultar em perda de dados não salvos
- Pode interromper operações em andamento abruptamente

É importante lembrar que a remoção de um container é permanente e todos os dados não persistidos em volumes serão perdidos.


### Acessando e alterando arquivos de um container

`docker run --name nginx -dp 8080:80 nginx`

- Cria e inicia um novo container baseado na imagem nginx
- Atribui o nome "nginx" ao container para referência fácil
- Executa em modo detached (`-d`) para rodar em segundo plano
- Mapeia a porta 8080 do host para a porta 80 do container (`-p 8080:80`)
- Isso permite acessar o servidor Nginx via localhost:8080

`docker exec nginx ls`

- Executa o comando `ls` dentro do container em execução chamado "nginx"
- Lista os arquivos e diretórios no diretório atual do container
- É uma forma de ver os arquivos sem entrar interativamente no container

`docker exec -it nginx bash`

- Inicia uma sessão interativa de terminal (`-it`) dentro do container
- Executa o shell Bash dentro do container em execução
- Você agora está "dentro" do container e pode navegar e manipular seus arquivos
- Seu prompt de comando muda para indicar que está dentro do container

`cd /usr/share/nginx/html`
- Este comando navega para o diretório onde o Nginx armazena os arquivos HTML servidos por padrão
- Este é o diretório raiz da web padrão do Nginx

 `apt-get update`

Dentro do container:

- Atualiza a lista de pacotes disponíveis nos repositórios
- Prepara o container para instalação de novos pacotes
- É necessário antes de instalar qualquer software com apt-get

`apt-get install vim`

Dentro do container:

- Instala o editor de texto Vim no container
- Adiciona a ferramenta que você precisa para editar arquivos

`vim index.html`

Dentro do container:

- Abre o arquivo index.html (página principal do Nginx) no editor Vim
- Permite visualizar e modificar o conteúdo da página

Teclas `i`, depois alterações, depois `Esc` e `:w`

Dentro do editor Vim:

- `i`: Entra no modo de inserção do Vim, permitindo editar o texto
- Alterações feitas: você modificou o conteúdo do arquivo HTML
- `Esc`: Sai do modo de inserção para o modo de comando
- `:w`: Salva as alterações feitas no arquivo

Estas alterações modificam o conteúdo servido pelo Nginx no container. Após salvar, se você acessar localhost:8080 no navegador do host, verá a página HTML modificada.

### Iniciando com bind mounts
O bind #mount é um mecanismo do Docker que permite "montar" um diretório ou arquivo do sistema host diretamente em um container. Isso cria uma conexão direta entre o sistema de arquivos do host e o do container.

O que são Bind Mounts

Bind mounts estabelecem uma referência direta entre um caminho no host e um caminho no container. Quando você usa um bind mount:

- Qualquer arquivo existente no caminho de destino do container é "coberto" pelo conteúdo do host
- As alterações feitas nos arquivos de qualquer lado (host ou container) são imediatamente visíveis no outro lado
- Os arquivos permanecem no host mesmo depois que o container é removido

`docker run -d --name nginx -p 8080:80 -v ~/projects/fullcycle3/docker/html/:/usr/share/nginx/html nginx`

Este comando faz o seguinte:

1. `-d`: Executa o container em modo detached (segundo plano)
2. `--name nginx`: Define o nome do container como "nginx"
3. `-p 8080:80`: Mapeia a porta 8080 do host para a porta 80 do container
4. `-v ~/projects/fullcycle3/docker/html/:/usr/share/nginx/html`:
    - Esta é a parte do bind mount
    - Monta o diretório `~/projects/fullcycle3/docker/html/` do seu computador
    - No caminho `/usr/share/nginx/html` dentro do container
    - O símbolo `:` separa o caminho do host do caminho do container
5. `nginx`: Usa a imagem nginx para criar o container


Benefícios no desenvolvimento

Este tipo de configuração é muito útil para desenvolvimento porque:

1. Você pode editar os arquivos HTML, CSS e JavaScript no seu editor preferido no host
2. As alterações são instantaneamente refletidas no container sem precisar reconstruí-lo
3. O servidor Nginx dentro do container servirá automaticamente os arquivos atualizados
4. Você pode ver as mudanças imediatamente no navegador ao acessar localhost:8080

Este é um fluxo de trabalho de desenvolvimento muito mais ágil do que precisar modificar arquivos dentro do container usando `docker exec` e um editor como vim, como você estava fazendo anteriormente.



### Trabalhando com volumes
Os volumes são o mecanismo preferencial para persistir dados em containers Docker, gerenciados pelo próprio Docker.

`docker volume`

Este é o comando base para gerenciar volumes no Docker. Sozinho, ele mostra os subcomandos disponíveis.

 `docker volume ls`

Lista todos os volumes criados, mostrando:

- O nome do volume
- O driver utilizado (normalmente "local")

 `docker volume create meuvolume`

Cria um novo volume chamado "meuvolume" que pode ser usado por containers.

- O Docker cria um espaço de armazenamento no sistema de arquivos host
- Este espaço é gerenciado pelo Docker e isolado do restante do sistema

 `docker volume inspect meuvolume`

Mostra informações detalhadas sobre o volume, incluindo:

- Nome
- Driver
- Ponto de montagem no sistema host (Mountpoint)
- Escopo (local ou swarm)
- Data de criação
- Labels


 `docker run --name nginx -d -v meuvolume:/app nginx`

Este comando:

- Cria um container nginx
- Monta o volume "meuvolume" no diretório "/app" dentro do container
- Qualquer dado escrito no diretório "/app" será armazenado no volume

 `docker volume prune`

Remove todos os volumes não utilizados (volumes que não estão conectados a nenhum container).

- Recupera espaço em disco
- Cuidado: os dados nos volumes removidos são perdidos permanentemente

### Diferenças entre Bind Mounts e Volumes

**Volumes:**

- São gerenciados pelo Docker
- Mais fáceis de fazer backup e migrar
- Podem ser gerenciados através da CLI do Docker
- Funcionam em Windows e Linux
- Podem ser compartilhados com segurança entre múltiplos containers
- Podem usar drivers para armazenamento remoto ou funcionalidades como criptografia
- O conteúdo inicial de um volume pode ser povoado pelo container

**Bind Mounts:**

- Dependem da estrutura de diretórios do host
- Oferecem acesso a arquivos sensíveis do host
- O container pode modificar arquivos do host diretamente
- Têm melhor performance (por serem caminhos diretos no host)
- São bons para desenvolvimento quando os arquivos precisam ser acessíveis tanto no host quanto no container
- Menos portáteis, pois dependem de caminhos específicos no host

Os volumes são geralmente recomendados para aplicações em produção pela sua portabilidade e gerenciamento, enquanto bind mounts são mais comuns em desenvolvimento pela facilidade de edição direta de arquivos.