### Instalando framework em um container

Primeira etapa (build):
- Usa a imagem Maven com OpenJDK 17
- Copia os arquivos de configuração e o código fonte
- Compila e empacota a aplicação usando Maven

**Segunda etapa (execução)**:

- Usa uma imagem mais leve de OpenJDK 17
- Copia apenas o arquivo JAR gerado na etapa de build
- Expõe a porta 8080 (padrão do Spring Boot)
- Define o comando para executar a aplicação


```Dockerfile
FROM maven:3.8.5-openjdk-17 AS build
WORKDIR /app 

# Copiar os arquivos de configuração do Maven
COPY pom.xml . 
# Copiar o código fonte 
COPY src ./src 

# Empacotar a aplicação 
RUN mvn clean package -DskipTests 

# Segunda etapa - imagem de execução 
FROM openjdk:17-jdk-slim 
WORKDIR /app 

# Copiar o arquivo JAR da etapa de build 
COPY --from=build /app/target/*.jar app.jar 

# Expor a porta que a aplicação Spring Boot usa 
EXPOSE 8080

# Comando para executar a aplicação
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Agora, vamos ver como construir e executar o container:

```bash
# Construir a imagem 
docker build -t springapp . 

# Executar o container 
docker run -p 8080:8080 springapp

```


### Criando aplicação Node.js sem o Node

Isso é uma das grandes vantagens do Docker - você pode desenvolver e executar aplicações sem instalar suas dependências diretamente no seu sistema.

```Dockerfile
FROM node:18-alpine

# Criar diretório de trabalho
WORKDIR /app

# Copiar arquivos de definição de pacotes
COPY package*.json ./

# Instalar dependências
RUN npm install

# Copiar código fonte
COPY . .

# Expor a porta que a aplicação usará
EXPOSE 3000

# Comando para iniciar a aplicação
CMD ["node", "app.js"]
```

