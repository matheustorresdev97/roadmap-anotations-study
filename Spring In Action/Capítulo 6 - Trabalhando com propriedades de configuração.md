Este capítulo abrange:
	Ajsute fino de beans autoconfigurados
	Aplicação de propriedades de configuração a componentes de aplicativo
	Trabalho com perfis Spring

Você se lembra de quando o iPhone foi lançado? Uma pequena placa de metal e vidro dificilmente se encaixava na descrição do que o mundo passou a reconhecer como um telefone. E ainda assim, ele foi pioneiro na era moderna dos smartphones, mudando tudo sobre como nos comunicamos. Embora os telefones touch sejam, de muitas maneiras, mais fáceis e poderosos do que seu antecessor, o flip phone, quando o iPhone foi anunciado pela primeira vez, era difícil imaginar como um dispositivo com um único botão poderia ser usado para fazer chamadas. De certa forma, a autoconfiguração do Spring Boot é assim. A autoconfiguração simplifica muito o desenvolvimento de aplicativos Spring. Mas depois de uma década definindo valores de propriedade na configuração Spring XML e chamando métodos setter em instâncias de bean, não é imediatamente aparente como definir propriedades em beans para os quais não há configuração explícita. Felizmente, o Spring Boot fornece uma maneira de definir valores de propriedade em componentes de aplicativo com propriedades de configuração. Propriedades de configuração nada mais são do que propriedades em beans anotados com @ConfigurationProperties no contexto do aplicativo Spring. O Spring injetará valores de uma das várias fontes de propriedade —incluindo propriedades do sistema JVM, argumentos de linha de comando e variáveis ​​de ambiente — nas propriedades do bean. Veremos como usar @ConfigurationProperties emnossos próprios beans na seção 6.2. Mas o próprio Spring Boot fornece vários beans anotados com @ConfigurationProperties que configuraremos primeiro.
Neste capítulo, você dará um passo para trás na implementação de novos recursos no aplicativo Taco Cloud para explorar as propriedades de configuração. O que você levar sem dúvida será útil à medida que você avança nos capítulos seguintes.
Começaremos vendo como empregar propriedades de configuração para ajustar o que o Spring Boot configura automaticamente.

### 6.1 Ajuste fino da autoconfiguração
Antes de nos aprofundarmos muito nas propriedades de configuração, é importante estabelecer os seguintes tipos diferentes (mas relacionados) de configurações no Spring:
- Bean wiring - configuração que declara componentes do aplicativo a serem criados como beans no contexto do aplicativo Spring e como eles devem ser injetados uns nos outros
- Property injection - configuração que define valores em beans no contexto do aplicativo Spring

Na configuração XML e Java do Spring, esses dois tipos de configurações são frequentemente declarados explicitamente no mesmo lugar. Na configuração Java, um método @Beanannotated
provavelmente instancia um bean e, em seguida, define valores para suas propriedades. Por exemplo, considere o seguinte método @Bean que declara um DataSource para um
banco de dados H2 incorporado:

```java
@Bean
public DataSource dataSource() {
return new EmbeddedDatabaseBuilder()
.setType(H2)
.addScript("taco_schema.sql")
.addScripts("user_data.sql", "ingredient_data.sql")
.build();
}
```

Aqui, os métodos addScript() e addScripts() definem algumas propriedades String com o nome dos scripts SQL que devem ser aplicados ao banco de dados quando a fonte de dados estiver
pronta. Enquanto é assim que você pode configurar um bean DataSource se não estiver usando o Spring Boot, a autoconfiguração torna esse método completamente desnecessário.
Se a dependência H2 estiver disponível no classpath de tempo de execução, o Spring Boot cria automaticamente no contexto do aplicativo Spring um bean DataSource apropriado, que aplica os scripts SQL schema.sql e data.sql. Mas e se você quiser nomear os scripts SQL de outra forma? Ou se você precisar especificar mais de dois scripts SQL? É aí que as propriedades de configuração entram.
Mas antes de começar a usar as propriedades de configuração, você precisa entender de onde essas propriedades vêm.

### 6.1.1 Entendendo a abstração do ambiente do Spring

A abstração do ambiente do Spring é um balcão único para qualquer propriedade configurável. Ela abstrai as origens das propriedades para que os beans que precisam dessas propriedades possam consumi-las do próprio Spring. O ambiente do Spring extrai de várias fontes de propriedade,

Incluindo as seguintes:
- Propriedades do sistema JVM
- Variáveis ​​de ambiente do sistema operacional
-  Argumentos da linha de comando
- Arquivos de configuração de propriedade do aplicativo
Ele então agrega essas propriedades em uma única fonte da qual os beans do Spring podem ser injetados. A Figura 6.1 ilustra como as propriedades das fontes de propriedade fluem através da abstração do ambiente do Spring para os beans do Spring.


![[Captura de tela de 2025-03-27 19-42-52.png]]

Os beans que são configurados automaticamente pelo Spring Boot são todos configuráveis ​​por propriedades extraídas do ambiente Spring. Como um exemplo simples, suponha que você
gostaria que o contêiner de servlet subjacente do aplicativo escutasse solicitações em alguma porta diferente da porta padrão 8080. Para fazer isso, especifique uma porta diferente definindo
a propriedade server.port em src/main/resources/application.properties assim:

server.port=9090

Pessoalmente, prefiro usar YAML ao definir propriedades de configuração. Portanto, em vez de usar application.properties, posso definir o valor server.port em src/
main/resources/application.yml assim:

```yaml
server:
	port: 9090
```

Se você preferir configurar essa propriedade externamente, você também pode especificar a porta ao iniciar o aplicativo usando um argumento de linha de comando como segue:

```bash
$ java -jar tacocloud-0.0.5-SNAPSHOT.jar --server.port=9090
```


Se você quiser que o aplicativo sempre inicie em uma porta específica, você pode defini-lo uma vez como uma variável de ambiente do sistema operacional, conforme mostrado a seguir:

```shell
$ export SERVER_PORT=9090
```

Observe que ao definir propriedades como variáveis ​​de ambiente, o estilo de nomenclatura é ligeiramente diferente para acomodar restrições colocadas em nomes de variáveis ​​de ambiente pelo sistema operacional. Tudo bem. O Spring é capaz de resolver isso e interpretar SERVER_PORT como server.port sem problemas. Como eu disse, temos várias maneiras de definir propriedades de configuração. Na verdade, você poderia usar uma das várias centenas de propriedades de configuração para ajustar e ajustar como os Spring beans se comportam. Você já viu algumas: server.port neste capítulo, assim como spring.datasource.name e spring.thymeleaf.cache em capítulos anteriores.
É impossível examinar todas as propriedades de configuração disponíveis neste capítulo. Mesmo assim, vamos dar uma olhada em algumas das propriedades de configuração mais úteis que você pode encontrar comumente. Começaremos com algumas propriedades que permitem ajustar a fonte de dados autoconfigurada.

### 6.1.2 Configurando uma fonte de dados

Neste ponto, o aplicativo Taco Cloud ainda está inacabado, mas você terá vários capítulos para cuidar disso antes de estar pronto para implantar o aplicativo. Como
tal, o banco de dados H2 incorporado que você está usando como fonte de dados é perfeito para suas necessidades — por enquanto. Mas depois que você colocar o aplicativo em produção, provavelmente vai querer considerar uma solução de banco de dados mais permanente. Embora você possa configurar explicitamente seu próprio bean DataSource, isso geralmente é
desnecessário. Em vez disso, é mais simples configurar a URL e as credenciais para seu banco de dados por meio de propriedades de configuração. Por exemplo, se você fosse começar a usar um banco de dados MySQL, você poderia adicionar as seguintes propriedades de configuração ao application.yml:

```yaml
spring:
	datasource:
	  url: jdbc:mysql:/ /localhost/tacocloud
	  username: tacouser
	  password: tacopassword
	  ```

Embora você precise adicionar o driver JDBC apropriado à compilação, você não normalmente precisará especificar a classe do driver JDBC — o Spring Boot pode descobrir isso a partir
da estrutura da URL do banco de dados. Mas se houver um problema, você pode tentar definir a propriedade spring.datasource.driver-class-name assim:

```yaml
spring:
datasource:
url: jdbc:mysql:/ /localhost/tacocloud
username: tacouser
password: tacopassword
driver-class-name: com.mysql.jdbc.Driver
```


O Spring Boot usa esses dados de conexão ao autoconfigurar o bean DataSource. O bean DataSource será agrupado usando o pool de conexões HikariCP se estiver disponível no classpath. Caso contrário, o Spring Boot procura e usa uma das seguintes outras implementações de pool de conexões no classpath:

-  Tomcat JDBC Connection Pool
- Apache Commons DBCP2

Embora essas sejam as únicas opções de pool de conexão disponíveis por meio da autoconfiguração, você sempre pode configurar explicitamente um bean DataSource para usar qualquer implementação de pool de conexão que desejar. No início deste capítulo, sugerimos que pode haver uma maneira de especificar os scripts de inicialização do banco de dados a serem executados quando o aplicativo for iniciado. Nesse caso, as propriedades spring.datasource.schema e spring.datasource.data, mostradas aqui, são úteis:

```yaml
spring:
datasource:
schema:
- order-schema.sql
- ingredient-schema.sql
- taco-schema.sql
- user-schema.sql
data:
- ingredients.sql
```

Talvez a configuração explícita da fonte de dados não seja seu estilo. Em vez disso, talvez você prefira configurar sua fonte de dados na Java Naming and Directory Interface (JNDI)
(http://mng.bz/MvEo) e fazer com que o Spring a procure de lá. Nesse caso, configure sua fonte de dados configurando spring.datasource.jndi-name da seguinte forma:

```yaml
spring:
datasource:
jndi-name: java:/comp/env/jdbc/tacoCloudDS
```

Se você definir o spring.datasource.jndi-name property, as outras propriedades de conexão da fonte de dados (se definidas) são ignoradas.