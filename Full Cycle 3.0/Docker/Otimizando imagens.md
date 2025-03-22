### Otimização utilizando Multistage Building

Node.js

```Dockerfile
# Estágio de construção
FROM node:18-alpine AS builder

WORKDIR /app

# Copiar arquivos de configuração e instalar dependências
COPY package*.json ./
RUN npm ci

# Copiar o código fonte e construir
COPY . .
RUN npm run build

# Remover dependências de desenvolvimento
RUN npm prune --production


# Estágio de produção
FROM node:18-alpine

# Definir variáveis de ambiente para produção
ENV NODE_ENV=production

# Criar usuário não-root para segurança
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodeuser -u 1001 -G nodejs

WORKDIR /app

# Copiar apenas arquivos necessários do estágio de build
COPY --from=builder --chown=nodeuser:nodejs /app/node_modules /app/node_modules
COPY --from=builder --chown=nodeuser:nodejs /app/dist /app/dist
COPY --from=builder --chown=nodeuser:nodejs /app/package.json /app/

# Trocar para usuário não-root
USER nodeuser

# Expor porta
EXPOSE 3000

# Comando para iniciar a aplicação
CMD ["node", "dist/server.js"]
```

Java (Spring Boot)

```Dockerfile
# Estágio de construção
FROM maven:3.8.6-eclipse-temurin-17 AS builder

WORKDIR /build

# Copiar arquivos de configuração Maven
COPY pom.xml .
# Baixar dependências (cache-friendly)
RUN mvn dependency:go-offline

# Copiar código fonte
COPY src ./src

# Compilar e empacotar
RUN mvn package -DskipTests

# Verificar o JAR gerado e criar uma versão não compactada para facilitar extração de camadas
RUN mkdir -p target/extracted && \
    java -Djarmode=layertools -jar target/*.jar extract --destination target/extracted


# Estágio de produção
FROM eclipse-temurin:17-jre-alpine

# Criar usuário não-root
RUN addgroup -S spring && adduser -S spring -G spring

WORKDIR /app

# Copiar camadas da aplicação extraídas no estágio anterior
COPY --from=builder --chown=spring:spring /build/target/extracted/dependencies/ ./
COPY --from=builder --chown=spring:spring /build/target/extracted/spring-boot-loader/ ./
COPY --from=builder --chown=spring:spring /build/target/extracted/snapshot-dependencies/ ./
COPY --from=builder --chown=spring:spring /build/target/extracted/application/ ./

# Trocar para usuário não-root
USER spring:spring

# Expor porta
EXPOSE 8080

# Iniciar a aplicação
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```


### Nginx como Proxy Reverso

Node.js com Nginx

```Dockerfile
# Estágio de construção para o aplicativo Node.js
FROM node:18-alpine AS app-builder

WORKDIR /app

# Instalar dependências
COPY package*.json ./
RUN npm ci

# Copiar e construir a aplicação
COPY . .
RUN npm run build


# Estágio do Nginx
FROM nginx:alpine

# Copiar a configuração personalizada do Nginx
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copiar arquivos estáticos da aplicação Node.js
COPY --from=app-builder /app/dist /usr/share/nginx/html

# Expor porta 80
EXPOSE 80

# Iniciar Nginx
CMD ["nginx", "-g", "daemon off;"]
```


```conf
server {
    listen 80;
    server_name localhost;

    # Arquivos estáticos
    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
        try_files $uri $uri/ /index.html;
    }

    # Proxy para API Node.js
    location /api/ {
        # Redirecionar para o serviço Node.js (ajuste o nome do serviço conforme seu ambiente)
        proxy_pass http://nodejs-api:3000/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Configurações de segurança
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    # Otimização
    gzip on;
    gzip_comp_level 5;
    gzip_min_length 256;
    gzip_proxied any;
    gzip_types
        application/javascript
        application/json
        application/xml
        text/css
        text/javascript
        text/plain
        text/xml;
}
```

