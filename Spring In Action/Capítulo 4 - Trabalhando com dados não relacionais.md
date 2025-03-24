Este Capítulo abrange:

	Persistindo dados no Cassandra
	Modelagem de dados no Cassandra
	Trabalhando com dados de documentos no MongoDB

Dizem que a variedade é o tempero da vida.
Você provavelmente tem um sabor favorito de sorvete. É aquele sabor que você escolhe com mais frequência porque ele satisfaz aquele desejo cremoso mais do que qualquer outro. Mas a maioria das pessoas, apesar de ter um sabor favorito, experimenta sabores diferentes de vez em vez para misturar as coisas. Bancos de dados são como sorvete. Por décadas, o banco de dados relacional tem sido o sabor favorito para armazenar dados. Mas hoje em dia, temos mais opções disponíveis do que nunca. Os chamados bancos de dados “NoSQL” (https://aws.amazon.com/nosql/) oferecem diferentes conceitos e estruturas nas quais os dados podem ser armazenados. E embora a escolha ainda possa ser baseada no gosto, alguns bancos de dados são mais adequados para persistir diferentes tipos de dados do que outros. Felizmente, a Spring Data cobre você para muitos dos bancos de dados #NoSQL, incluindo #MongoDB, #Cassandra, Couchbase, Neo4j, Redis e muitos mais. E felizmente, o modelo de programação é quase idêntico, independentemente do banco de dados que você escolher. Não há espaço suficiente neste capítulo para cobrir todos os bancos de dados que o Spring Data suporta. Mas para dar a você uma amostra dos outros "sabores" do Spring Data, veremos dois bancos de dados NoSQL populares, Cassandra e MongoDB, e veremos como criar repositórios para persistir dados neles. Vamos começar observando como criar repositórios Cassandra com o Spring Data.

### 4.1 Trabalhando com repositórios Cassandra
Cassandra é um banco de dados NoSQL distribuído, de alto desempenho, sempre disponível, eventualmente consistente, de armazenamento em colunas particionadas.
São muitos adjetivos para descrever um banco de dados, mas cada um deles fala com precisão sobre o poder de trabalhar com o Cassandra. Para colocar em termos mais simples, o Cassandra lida com linhas de dados gravadas em tabelas, que são particionadas em nós distribuídos de um para muitos. Nenhum nó carrega todos os dados, mas qualquer linha pode ser replicada em vários nós, eliminando assim qualquer ponto único de falha.
O Spring Data Cassandra fornece suporte automático de repositório para o banco de dados Cassandra que é bem parecido com — e ainda bem diferente — do que é oferecido pelo Spring Data JPA para bancos de dados relacionais. Além disso, o Spring Data Cassandra oferece anotações para mapear tipos de domínio de aplicativo para as estruturas de banco de dados de apoio. Antes de explorarmos o Cassandra mais a fundo, é importante entender que embora o Cassandra compartilhe muitos conceitos semelhantes aos bancos de dados relacionais como Oracle e SQL Server, o Cassandra não é um banco de dados relacional e é, de muitas maneiras, uma fera bem diferente. Explicarei as idiossincrasias do Cassandra no que diz respeito ao trabalho com o Spring Data. Mas eu o encorajo a ler a própria documentação do Cassandra (http://cassandra.apache.org/doc/latest/) para uma compreensão completa do que o faz funcionar.
Vamos começar habilitando o Spring Data Cassandra no projeto Taco Cloud.

### 4.1.1 Habilitando o Spring Data Cassandra
Para começar a usar o Spring Data Cassandra, você precisará adicionar a dependência do Spring Boot starter para o Spring Data Cassandra não reativo. Na verdade, há duas dependências separadas do Spring Data Cassandra starter para escolher: uma para persistência de dados reativos e uma para persistência padrão não reativa. Falaremos mais sobre como escrever repositórios reativos mais adiante no capítulo 15. Por enquanto, porém, usaremos o starter não reativo em nossa compilação, conforme mostrado aqui:

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-cassandra</artifactId>
</dependency>
```

Esta dependência também está disponível no Initializr marcando a caixa de seleção Cassandra.
É importante entender que essa dependência está no lugar das dependências Spring Data JPA starter ou Spring Data JDBC que usamos no capítulo anterior. Em vez de persistir dados do Taco Cloud em um banco de dados relacional com JPA ou JDBC, você usará o Spring Data para persistir dados em um banco de dados Cassandra. Portanto, você desejará remover as dependências Spring Data JPA ou Spring Data JDBC starter e quaisquer dependências de banco de dados relacional (como drivers JDBC ou a dependência H2) da compilação.
A dependência Spring Data Cassandra starter traz um punhado de dependências para o projeto, especificamente, a biblioteca Spring Data Cassandra. Como resultado do Spring Data Cassandra estar no classpath de tempo de execução, a autoconfiguração para criar repositórios Cassandra é acionada. Isso significa que você pode começar a escrever repositórios Cassandra com configuração explícita mínima.
Cassandra opera como um cluster de nós que juntos agem como um sistema de banco de dados completo. Se você ainda não tem um cluster Cassandra para trabalhar, você pode iniciar um cluster de nó único para fins de desenvolvimento usando o #Docker assim:

```shell
 docker network create cassandra-net
 docker run --name my-cassandra \
--network cassandra-net \
-p 9042:9042 \
-d cassandra:latest
```

Isso inicia o cluster de nó único e expõe a porta do nó (9042) na máquina host para que seu aplicativo possa acessá-lo.
No entanto, você precisará fornecer uma pequena quantidade de configuração. No mínimo, você precisará configurar o nome de um keyspace dentro do qual seus repositórios irão operar. Para fazer isso, primeiro você precisará criar esse keyspace.
NOTA No Cassandra, um keyspace é um agrupamento de tabelas em um nó Cassandra. É aproximadamente análogo a como tabelas, visualizações e restrições são agrupadas em um banco de dados relacional.
Embora seja possível configurar o Spring Data Cassandra para criar o keyspace automaticamente, normalmente é muito mais fácil criá-lo manualmente (ou usar um keyspace existente). Usando o shell Cassandra CQL (Cassandra Query Language), você pode criar um keyspace para o aplicativo Taco Cloud. Você pode iniciar o shell CQL usando Docker assim:

```shell
$ docker run -it --network cassandra-net --rm cassandra cqlsh my-cassandra
```

NOTA Se este comando falhar ao iniciar o shell CQL com um erro indicando “Não foi possível conectar a nenhum servidor”, aguarde um ou dois minutos e tente novamente. Você precisa ter certeza de que o cluster Cassandra foi totalmente iniciado antes que o shell CQL possa se conectar a ele.
Quando o shell estiver pronto, use o comando create keyspace como este:

```shell
cqlsh> create keyspace tacocloud
... with replication={'class':'SimpleStrategy', 'replication_factor':1}
... and durable_writes=true;
```

Simplificando, isso criará um keyspace chamado tacocloud com replicação simples e gravações duráveis. Ao definir o fator de replicação como 1, você pede ao Cassandra para manter uma cópia de cada linha. A estratégia de replicação determina como a replicação é tratada. A estratégia de replicação SimpleStrategy é boa para uso em um único data center (e para código de demonstração), mas você pode considerar a NetworkTopologyStrategy se tiver seu cluster Cassandra espalhado por vários data centers. Recomendo a você a documentação do Cassandra para mais detalhes sobre como as estratégias de replicação funcionam e maneiras alternativas de criar keyspaces.
Agora que você criou um keyspace, precisa configurar a propriedade spring.data.cassandra.keyspace-name para dizer ao Spring Data Cassandra para usar esse keyspace, conforme mostrado a seguir:

```yaml
spring:
data:
cassandra:
keyspace-name: taco_cloud
schema-action: recreate
local-datacenter: datacenter1
```

Aqui, você também define o spring.data.cassandra.schema-action para recriar. Essa configuração é muito útil para fins de desenvolvimento porque garante que todas as tabelas e tipos definidos pelo usuário serão descartados e recriados sempre que o aplicativo for iniciado.
O valor padrão, none, não realiza nenhuma ação contra o esquema e é útil em configurações de produção onde você prefere não descartar todas as tabelas sempre que um aplicativo for iniciado.
Finalmente, a propriedade spring.data.cassandra.local-datacenter identifica o nome do data center local para fins de configuração da política de balanceamento de carga do Cassandra. Em uma configuração de nó único, "datacenter1" é o valor a ser usado. Para obter mais informações sobre as políticas de balanceamento de carga do Cassandra e como definir o data center local, consulte a documentação de referência do driver DataStax Cassandra (http://mng.bz/XrQM).
Essas são as únicas propriedades que você precisará para trabalhar com um banco de dados Cassandra em execução local. Além dessas duas propriedades, no entanto, você pode desejar definir outras, dependendo de como você configurou seu cluster Cassandra.
Por padrão, o Spring Data Cassandra assume que o Cassandra está sendo executado localmente e escutando na porta 9042. Se esse não for o caso, como em uma configuração de produção, você pode desejar definir as propriedades spring.data.cassandra.contact-points e spring.data.cassandra.port da seguinte forma:

```yaml
spring:
data:
cassandra:
keyspace-name: tacocloud
local-datacenter: datacenter1
contact-points:
- casshost-1.tacocloud.com
- casshost-2.tacocloud.com
- casshost-3.tacocloud.com
port: 9043
```

Observe que a propriedade spring.data.cassandra.contact-points é onde você identifica o(s) nome(s) do host do Cassandra. Um ponto de contato é o host onde um nó do Cassandra está em execução. Por padrão, ele é definido como localhost, mas você pode defini-lo como uma lista de nomes de host. Ele tentará cada ponto de contato até conseguir se conectar a um. Isso é para garantir que não haja um único ponto de falha no cluster do Cassandra e que o aplicativo consiga se conectar ao cluster por meio de um dos pontos de contato fornecidos.
Você também pode precisar especificar um nome de usuário e uma senha para seu cluster do Cassandra.
Isso pode ser feito definindo as propriedades spring.data.cassandra.username e spring.data.cassandra.password, conforme mostrado a seguir:

```yaml
spring:
data:
cassandra:
...
username: tacocloud
password: s3cr3tP455w0rd
```

Estas são as únicas propriedades que você precisará para trabalhar com um banco de dados Cassandra em execução local. Além dessas duas propriedades, no entanto, você pode desejar definir outras, dependendo de como você configurou seu cluster Cassandra.
Agora que o Spring Data Cassandra está habilitado e configurado em seu projeto, você está quase pronto para mapear seus tipos de domínio para tabelas Cassandra e escrever repositórios. Mas primeiro, vamos dar um passo para trás e considerar alguns pontos básicos da modelagem de dados Cassandra.

### 4.1.2 Entendendo a modelagem de dados Cassandra
Como mencionei, o Cassandra é bem diferente de um banco de dados relacional. Antes de você começar a mapear seus tipos de domínio para tabelas Cassandra, é importante entender algumas das maneiras pelas quais a modelagem de dados Cassandra é diferente de como você pode modelar seus dados para persistência em um banco de dados relacional.
Algumas das coisas mais importantes para entender sobre a modelagem de dados Cassandra seguem:

- As tabelas Cassandra podem ter qualquer número de colunas, mas nem todas as linhas usarão necessariamente todas essas colunas.
- Os bancos de dados Cassandra são divididos em várias partições. Qualquer linha em uma determinada tabela pode ser gerenciada por uma ou mais partições, mas é improvável que todas as partições tenham todas as linhas.
- Uma tabela Cassandra tem dois tipos de chaves: chaves de partição e chaves de cluster. As operações de hash são executadas na chave de partição de cada linha para determinar por qual(is) partição(ões) essa linha será gerenciada. As chaves de cluster determinam a ordem em que as linhas são mantidas dentro de uma partição (não necessariamente a ordem em que elas podem aparecer nos resultados de uma consulta). Consulte a documentação do Cassandra (http://mng.bz/yJ6E) para uma explicação mais detalhada da modelagem de dados no Cassandra, incluindo partições, clusters e suas respectivas chaves.
- O Cassandra é altamente otimizado para operações de leitura. Como tal, é comum e desejável que as tabelas sejam altamente desnormalizadas e que os dados sejam duplicados em várias tabelas. (Por exemplo, as informações do cliente podem ser mantidas em uma tabela de clientes, bem como duplicadas em uma tabela contendo pedidos feitos por clientes.) Basta dizer que adaptar os tipos de domínio do Taco Cloud para trabalhar com o Cassandra não será uma questão de simplesmente trocar algumas anotações JPA por anotações do Cassandra. Você terá que repensar como modelar os dados.

### 4.1.3 Mapeando tipos de domínio para persistência do Cassandra
No capítulo 3, você marcou seus tipos de domínio (Taco, Ingredient, TacoOrder e assim por diante) com anotações fornecidas pela especificação JPA. Essas anotações mapearam seus tipos de domínio como entidades a serem persistidas em um banco de dados relacional. Embora essas anotações não funcionem para persistência do Cassandra, o Spring Data Cassandra fornece seu próprio conjunto de anotações de mapeamento para um propósito semelhante. Vamos começar com a classe Ingredient, porque é a mais simples de mapear para o Cassandra. A nova classe Ingredient pronta para o Cassandra se parece com isso:

```java
package tacos;
import org.springframework.data.cassandra.core.mapping.PrimaryKey;
import org.springframework.data.cassandra.core.mapping.Table;
import lombok.AccessLevel;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.RequiredArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor(access=AccessLevel.PRIVATE, force=true)
@Table("ingredients")
public class Ingredient {

@PrimaryKey
private String id;
private String name;
private Type type;

public enum Type {
WRAP, PROTEIN, VEGGIES, CHEESE, SAUCE
}
}
```

A classe Ingredient parece contradizer tudo o que eu disse sobre apenas trocar algumas anotações. Em vez de anotar a classe com @Entity como você fez para persistência JPA, ela é anotada com @Table para indicar que os ingredientes devem ser persistidos em uma tabela chamada ingredients. E em vez de anotar a propriedade id com @Id, desta vez ela é anotada com @PrimaryKey. Até agora, parece que você está apenas trocando algumas anotações. Mas não se deixe enganar pelo mapeamento Ingredient. A classe Ingredient é um dos seus tipos de domínio mais simples. As coisas ficam mais interessantes quando você mapeia a classe Taco
para persistência Cassandra, como mostrado na próxima listagem.

Listagem 4.1 - Anotando a classe Taco para persistência Cassandra

```java
package tacos;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.UUID;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;
import org.springframework.data.cassandra.core.cql.Ordering;
import org.springframework.data.cassandra.core.cql.PrimaryKeyType;
import org.springframework.data.cassandra.core.mapping.Column;
import org.springframework.data.cassandra.core.mapping.PrimaryKeyColumn;
import org.springframework.data.cassandra.core.mapping.Table;
import com.datastax.oss.driver.api.core.uuid.Uuids;
import lombok.Data;
@Data
@Table("tacos") // Persiste na tabela Tacos
public class Taco { 

@PrimaryKeyColumn(type=PrimaryKeyType.PARTITIONED) //Define a chave de partição
private UUID id = Uuids.timeBased();

@NotNull
@Size(min = 5, message = "Name must be at least 5 characters long")
private String name;

@PrimaryKeyColumn(type=PrimaryKeyType.CLUSTERED, //Define a chave de clustering
ordering=Ordering.DESCENDING)
private Date createdAt = new Date();

@Size(min=1, message="You must choose at least 1 ingredient") //Maps the list to the "ingredients" column
@Column("ingredients")
private List<IngredientUDT> ingredients = new ArrayList<>();

public void addIngredient(Ingredient ingredient) {
this.ingredients.add(TacoUDRUtils.toIngredientUDT(ingredient));
}
}
```

Como você pode ver, mapear a classe Taco é um pouco mais complexo. Assim como com Ingredient, a anotação @Table é usada para identificar tacos como o nome da tabela na qual os tacos devem ser gravados. Mas essa é a única coisa semelhante a Ingredient. A propriedade id ainda é sua chave primária, mas é apenas uma das duas colunas de chave primária. Mais especificamente, a propriedade id é anotada com @PrimaryKeyColumn com um tipo de PrimaryKeyType.PARTITIONED. Isso especifica que a propriedade id serve como a chave de partição, usada para determinar em qual(is) partição(ões) Cassandra cada linha de dados do taco será gravada. Você também notará que a propriedade id agora é um UUID em vez de um Long. Embora não seja obrigatório, as propriedades que contêm um valor de ID gerado são comumente do tipo
UUID. Além disso, o UUID é inicializado com um valor UUID baseado em tempo para novos objetos Taco (mas que pode ser substituído ao ler um Taco existente do banco de dados).
Um pouco mais abaixo, você vê a propriedade createdAt que é mapeada como outra coluna de chave primária. Mas neste caso, o atributo type de @PrimaryKeyColumn é definido  como PrimaryKeyType.CLUSTERED, que designa a propriedade createdAt como uma chave de cluster. Conforme mencionado anteriormente, as chaves de cluster são usadas para determinar a ordem de linhas dentro de uma partição. Mais especificamente, a ordem é definida como ordem decrescente portanto, dentro de uma partição específica, as linhas mais novas aparecem primeiro na tabela tacos.
Finalmente, a propriedade ingredients agora é uma List of IngredientUDT objects em vez de uma List of Ingredient objects. Como você deve se lembrar, as tabelas Cassandra são altamente desnormalizadas e podem conter dados duplicados de outras tabelas. Embora a tabela de ingredientes sirva como a tabela de registro para todos os ingredientes disponíveis, os ingredientes escolhidos para um taco serão duplicados na coluna de ingredientes. Em vez de simplesmente referenciar uma ou mais linhas na tabela de ingredientes, a propriedade de ingredientes conterá dados completos para cada ingrediente escolhido.
Mas por que você precisa introduzir uma nova classe IngredientUDT? Por que você não pode simplesmente reutilizar a classe Ingredient? Simplificando, colunas que contêm coleções de dados, como a coluna de ingredientes, devem ser coleções de tipos nativos (inteiros, strings e
assim por diante) ou tipos definidos pelo usuário.
No Cassandra, os tipos definidos pelo usuário permitem que você declare colunas de tabela que são mais ricas do que tipos nativos simples. Frequentemente, eles são usados ​​como um análogo desnormalizado para chaves estrangeiras relacionais. Em contraste com as chaves estrangeiras, que apenas mantêm uma referência a uma linha em outra tabela, colunas com tipos definidos pelo usuário realmente carregam dados que podem ser copiados de uma linha em outra tabela. No caso da coluna de ingredientes na
tabela tacos, ela conterá uma coleção de estruturas de dados que definem os próprios ingredientes. Você não pode usar a classe Ingredient como um tipo definido pelo usuário, porque a anotação @Table já a mapeou como uma entidade para persistência no Cassandra. Portanto, você deve criar uma nova classe para definir como os ingredientes serão armazenados na coluna ingredients da tabela taco. IngredientUDT (onde UDT significa tipo definido pelo usuário) é a
classe para o trabalho, conforme mostrado aqui:

```java
package tacos;
import org.springframework.data.cassandra.core.mapping.UserDefinedType;
import lombok.AccessLevel;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.RequiredArgsConstructor;
@Data
@RequiredArgsConstructor
@NoArgsConstructor(access = AccessLevel.PRIVATE, force = true)
@UserDefinedType("ingredient")
public class IngredientUDT {
private final String name;
private final Ingredient.Type type;
}
```

Embora IngredientUDT se pareça muito com Ingredient, seus requisitos de mapeamento são muito mais simples. Ele é anotado com @UserDefinedType para identificá-lo como um tipo definido pelo usuário no Cassandra. Mas, de outra forma, é uma classe simples com algumas propriedades.
Você também notará que a classe IngredientUDT não inclui uma propriedade id.
Embora possa incluir uma cópia da propriedade id do Ingredient de origem, isso não é necessário. Na verdade, o tipo definido pelo usuário pode incluir quaisquer propriedades que você desejar — não precisa ser um mapeamento um para um com nenhuma definição de tabela.
Percebo que pode ser difícil visualizar como os dados em um tipo definido pelo usuário se relacionam com os dados que são persistidos em uma tabela. A Figura 4.1 mostra o modelo de dados para todo o banco de dados Taco Cloud, incluindo tipos definidos pelo usuário.

![[Captura de tela de 2025-03-24 08-01-16.png]]
Figura 4.1 Em vez de usar chaves estrangeiras e junções, as tabelas do Cassandra são desnormalizadas, com tipos definidos pelo usuário contendo dados copiados de tabelas relacionadas.

Específico para o tipo definido pelo usuário que você acabou de criar, observe como o Taco tem uma lista de objetos IngredientUDT, que contém dados copiados de objetos Ingredient. Quando um Taco é persistido, é o objeto Taco e a lista de objetos IngredientUDT que são persistidos na tabela tacos. A lista de objetos IngredientUDT é persistida inteiramente dentro da coluna ingredients.
Outra maneira de ver isso que pode ajudar você a entender como os tipos definidos pelo usuário são usados ​​é consultar o banco de dados para obter linhas da tabela tacos. Usando CQL e a ferramenta cqlsh que vem com o Cassandra, você vê os seguintes resultados:

```Nosql
cqlsh:tacocloud> select id, name, createdAt, ingredients from tacos;
```


| id          | name      | creadteAt    | ingredientes                                                                                                                                                                                       |
| ----------- | --------- | ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 827390...\| | Carnivore | 2025-04...\| | [{name:  'Flour Tortilla', type: 'WRAP'},<br>{name: 'Carnitas', type: 'PROTEIN'},<br>{name: 'Sour Cream', type: 'SAUCE'},<br>{name: 'Salsa', type: 'SAUCE'},<br>{name: 'Cheddar', type: 'CHEESE'}] |

Como você pode ver, as colunas id, name e createdAt contêm valores simples. Nesse respeito, elas não são muito diferentes do que você esperaria de uma consulta semelhante
em um banco de dados relacional. Mas a coluna ingredients é um pouco diferente. Como
ela é definida como contendo uma coleção do tipo de ingrediente definido pelo usuário (definido por IngredientUDT), seu valor aparece como uma matriz JSON preenchida com objetos JSON.
Você provavelmente notou outros tipos definidos pelo usuário na figura 4.1. Você certamente criará mais alguns conforme continuar mapeando seu domínio para tabelas Cassandra, incluindo algumas que serão usadas pela classe TacoOrder. A próxima listagem mostra a classe TacoOrder, modificada para persistência Cassandra.

```java
import lombok.Data;
@Data
@Table("orders") //Mapeia para a tabela de pedidos
public class TacoOrder implements Serializable {
private static final long serialVersionUID = 1L;
@PrimaryKey
private UUID id = Uuids.timeBased(); //Declara a chave primária
private Date placedAt = new Date();
// delivery and credit card properties omitted for brevity's sake
@Column("tacos")
private List<TacoUDT> tacos = new ArrayList<>(); //Mapeia uma lista para a coluna de tacos
public void addTaco(TacoUDT taco) {
this.tacos.add(taco);
}
}
```

A Listagem 4.2 omite propositalmente muitas das propriedades de TacoOrder que não se prestam a uma discussão sobre modelagem de dados do Cassandra. O que resta são algumas propriedades e mapeamentos, semelhantes a como o Taco foi definido. @Table é usado para mapear TacoOrder para a tabela orders, assim como @Table foi usado antes. Neste caso, você não está preocupado com a ordenação, então a propriedade id é simplesmente anotada com @PrimaryKey, designando-a como uma chave de partição e uma chave de cluster com ordenação padrão.
A propriedade tacos é de algum interesse, pois é uma List< TacoUDT > em vez de uma lista de objetos Taco. O relacionamento entre TacoOrder e Taco/TacoUDT aqui é semelhante ao relacionamento entre Taco e Ingredient/IngredientUDT. Ou seja, em vez de juntar dados de várias linhas em uma tabela separada por meio de chaves estrangeiras, a tabela orders conterá todos os dados pertinentes do taco, otimizando a tabela para leituras rápidas.
A classe TacoUDT é bem parecida com a classe IngredientUDT, embora ela inclua uma coleção que faz referência a outro tipo definido pelo usuário, como a seguir:

```java
package tacos;
import java.util.List;
import org.springframework.data.cassandra.core.mapping.UserDefinedType;
import lombok.Data;
@Data
@UserDefinedType("taco")
public class TacoUDT {
private final String name;
private final List<IngredientUDT> ingredients;
}
```

Embora fosse bom reutilizar as mesmas classes de domínio que você criou no capítulo 3, ou no máximo trocar algumas anotações JPA por anotações Cassandra, a natureza da persistência do Cassandra é tal que requer que você repense como seus dados são modelados. Mas agora que você mapeou seu domínio, está pronto para escrever repositórios.

4.1.4 Escrevendo repositórios Cassandra

Como você viu no capítulo 3, escrever um repositório com Spring Data envolve simplesmente declarar uma interface que estende uma das interfaces de repositório base do Spring Data e

opcionalmente declarar métodos de consulta adicionais para consultas personalizadas. Como se vê,
escrever repositórios Cassandra não é muito diferente.
Na verdade, há muito pouco que você precisará mudar nos repositórios que
já escrevemos para fazê-los funcionar para persistência Cassandra. Por exemplo, considere
o seguinte IngredientRepository que criamos no capítulo 3:

```java
package tacos.data;
import org.springframework.data.repository.CrudRepository;
import tacos.Ingredient;
public interface IngredientRepository
extends CrudRepository<Ingredient, String> {
}
```

### 4.1.4 Escrevendo repositórios Cassandra

Como você viu no capítulo 3, escrever um repositório com Spring Data envolve simplesmente declarar uma interface que estende uma das interfaces de repositório base do Spring Data e opcionalmente declarar métodos de consulta adicionais para consultas personalizadas. Como se vê, escrever repositórios Cassandra não é muito diferente.
Na verdade, há muito pouco que você precisará mudar nos repositórios que já escrevemos para fazê-los funcionar para persistência Cassandra. Por exemplo, considere
o seguinte IngredientRepository que criamos no capítulo 3:

```java
package tacos.data;
import org.springframework.data.repository.CrudRepository;
import tacos.Ingredient;

public interface IngredientRepository extends CrudRepository<Ingredient, String> {
}
```

Ao estender CrudRepository como mostrado aqui, IngredientRepository está pronto para persistir objetos Ingredient cuja propriedade ID (ou, no caso do Cassandra, a propriedade chave primária) é uma String. Isso é perfeito! Nenhuma alteração é necessária para IngredientRepository.
As alterações necessárias para OrderRepository são apenas um pouco mais envolvidas.
Em vez de um parâmetro Long, o tipo de parâmetro ID especificado ao estender CrudRepository será alterado para UUID da seguinte forma:

```java
package tacos.data;
import java.util.UUID;
import org.springframework.data.repository.CrudRepository;
import tacos.TacoOrder;

public interface OrderRepository extends CrudRepository<TacoOrder, UUID> {
}
```

Há muito poder no Cassandra, e quando ele é unido ao Spring Data, você pode exercer esse poder em seus aplicativos Spring. Mas vamos mudar nossa atenção para outro banco de dados para o qual o suporte ao repositório Spring Data está disponível: MongoDB.

### 4.2 Escrevendo repositórios MongoDB
#MongoDB é outro banco de dados  #NoSQL bem conhecido. Enquanto Cassandra é um banco de dados de armazenamento de colunas, MongoDB é considerado um banco de dados de documentos. Mais especificamente, MongoDB armazena documentos no formato BSON (Binary JSON), que pode ser consultado e recuperado de uma forma que é aproximadamente similar a como você pode consultar dados em qualquer outro banco de dados.
Assim como com Cassandra, é importante entender que MongoDB não é um banco de dados relacional. A maneira como você gerencia seu cluster de servidor MongoDB, bem como como você modela seus dados, requer uma mentalidade diferente do que quando se trabalha com outros tipos de bancos de dados.
Dito isso, trabalhar com MongoDB e Spring Data não é drasticamente diferente de como você pode usar Spring Data para trabalhar com JPA ou Cassandra. Você anotará suas classes de domínio com anotações que mapeiam o tipo de domínio para uma estrutura de documento. E você escreverá interfaces de repositório que seguem muito o mesmo modelo de programação que você viu para JPA e Cassandra. Antes de poder fazer qualquer disso, no entanto, você deve habilitar o Spring Data MongoDB em seu projeto.

### 4.2.1 Habilitando o Spring Data MongoDB
Para começar a usar o Spring Data MongoDB, você precisará adicionar o Spring Data MongoDB starter à compilação do projeto. Assim como o Spring Data Cassandra, o Spring Data MongoDB tem dois starters separados para escolher: um reativo e um não reativo. Veremos as opções reativas para persistência no capítulo 13. Por enquanto, adicione a seguinte dependência à compilação para trabalhar com o starter não reativo do MongoDB:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>
spring-boot-starter-data-mongodb
</artifactId>
</dependency>
```

Esta dependência também está disponível no Spring Initializr marcando a caixa de seleção MongoDB em NoSQL. Ao adicionar o starter à compilação, a autoconfiguração será acionada para habilitar o suporte do Spring Data para escrever interfaces de repositório automáticas, como aquelas que você escreveu para JPA no capítulo 3 ou para Cassandra anteriormente neste capítulo. Por padrão, o Spring Data MongoDB assume que você tem um servidor MongoDB em execução localmente e escutando na porta 27017. Se você tiver o Docker instalado em sua máquina, uma maneira fácil de colocar um servidor MongoDB em execução é com a seguinte linha de comando:

```shell
docker run -p 27017:27017 -d mongo:latest
```

Mas para conveniência em testes ou desenvolvimento, você pode escolher trabalhar com um banco de dados Mongo incorporado. Para fazer isso, adicione a seguinte dependência do MongoDB incorporado do Flapdoodle à sua compilação:

```xml
<dependency>
<groupId>de.flapdoodle.embed</groupId>
<artifactId>de.flapdoodle.embed.mongo</artifactId>
<!-- <scope>test</scope> -->
</dependency>
```

O banco de dados incorporado do Flapdoodle oferece a você a mesma conveniência de trabalhar com um banco de dados Mongo na memória que você obteria com o H2 ao trabalhar com dados relacionais. Ou seja, você não precisará ter um banco de dados separado em execução, mas todos os dados serão limpos quando você reiniciar o aplicativo.
Bancos de dados incorporados são bons para desenvolvimento e teste, mas quando você levar seu aplicativo para produção, você vai querer ter certeza de definir algumas propriedades para deixar o Spring Data MongoDB saber onde e como seu banco de dados Mongo de produção pode ser acessado, conforme mostrado a seguir:

```yaml
spring:
data:
mongodb:
host: mongodb.tacocloud.com
port: 27017
username: tacocloud
password: s3cr3tp455w0rd
database: tacoclouddb
```

Nem todas essas propriedades são necessárias, mas elas estão disponíveis para ajudar a apontar o Spring Data MongoDB na direção certa no caso de seu banco de dados Mongo não estar sendo executado localmente. Para resumir, aqui está o que cada propriedade configura:

- spring.data.mongodb.host — O nome do host onde o Mongo está sendo executado (padrão:localhost)
- spring.data.mongodb.port — A porta em que o servidor Mongo está escutando (padrão: 27017)
- spring.data.mongodb.username — O nome de usuário para acessar um banco de dados Mongo seguro
- spring.data.mongodb.password — A senha para acessar um banco de dados Mongo seguro
- spring.data.mongodb.database — O nome do banco de dados (padrão: test)

Agora que você habilitou o Spring Data MongoDB em seu projeto, precisa anotar seus objetos de domínio para persistência como documentos no MongoDB.

### 4.2.2 Mapeando tipos de domínio para documentos

O Spring Data MongoDB oferece um punhado de anotações que são úteis para mapear tipos de domínio para estruturas de documentos a serem persistidas no MongoDB. Embora o Spring Data MongoDB forneça meia dúzia de anotações para mapeamento, apenas as seguintes quatro são úteis para os casos de uso mais comuns:

- @Id - Designa uma propriedade como o ID do documento (do Spring Data Commons)
- @Document - Declara um tipo de domínio como um documento a ser persistido no MongoDB
- @Field - Especifica o nome do campo (e, opcionalmente, a ordem) para armazenar uma propriedade no documento persistido
- @Transient - Especifica que uma propriedade não deve ser persistida.
Dessas três anotações, apenas as anotações @Id e @Document são estritamente necessárias. A menos que você especifique o contrário, propriedades que não são anotadas com @Field ou @Transient assumirão um nome de campo igual ao nome da propriedade. Aplicando essas anotações à classe Ingredient, você obtém o seguinte:

```java
package tacos;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import lombok.AccessLevel;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@Document
@AllArgsConstructor
@NoArgsConstructor(access=AccessLevel.PRIVATE, force=true)
public class Ingredient {

@Id
private String id;
private String name;

private Type type;

public enum Type {
WRAP, PROTEIN, VEGGIES, CHEESE, SAUCE
}
}
```


Como você pode ver, você coloca a anotação @Document no nível de classe para indicar que Ingredient é uma entidade de documento que pode ser escrita e lida de um banco de dados Mongo. Por padrão, o nome da coleção (o análogo do Mongo para uma tabela de banco de dados relacional) é baseado no nome da classe, com a primeira letra minúscula. Porque você não especificou caso contrário, os objetos Ingredient serão persistidos em uma coleção chamada ingrediente. Mas você pode mudar isso definindo o atributo collection de @Document da seguinte forma:

```java
@Data
@AllArgsConstructor
@NoArgsConstructor(access=AccessLevel.PRIVATE, force=true)
@Document(collection="ingredients")
public class Ingredient {
...
}
```

Você também notará que a propriedade id foi anotada com @Id. Isso designa a propriedade como sendo o ID do documento persistido. Você pode usar @Id em qualquer propriedade cujo tipo seja Serializable, incluindo String e Long. Neste caso, você já está usando a propriedade id definida por String como um identificador natural, então não há necessidade de alterá-la para nenhum outro tipo.
Até agora, tudo bem. Mas você se lembrará do início deste capítulo que Ingredient era o tipo de domínio fácil de mapear para Cassandra. Os outros tipos de domínio, como Taco, foram um pouco mais desafiadores. Vamos ver como você pode mapear a classe Taco para ver quais surpresas ela pode conter.
A abordagem do MongoDB para persistência de documentos se presta muito bem à maneira de design orientado a domínio de aplicar persistência no nível raiz agregado. Documentos no MongoDB tendem a ser definidos como raízes agregadas, com membros do agregado como subdocumentos.
O que isso significa para o Taco Cloud é que, como o Taco é sempre persistido apenas como um membro do agregado com raiz TacoOrder, a classe Taco não precisa ser anotada como um @Document, nem precisa de uma propriedade @Id. A classe Taco pode permanecer limpa de quaisquer anotações de persistência, como mostrado aqui:

```java
package tacos;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;
import lombok.Data;
@Data
public class Taco {
@NotNull
@Size(min=5, message="Name must be at least 5 characters long")
private String name;
private Date createdAt = new Date();
@Size(min=1, message="You must choose at least 1 ingredient")
private List<Ingredient> ingredients = new ArrayList<>();
public void addIngredient(Ingredient ingredient) {
this.ingredients.add(ingredient);
}
}
```

A classe TacoOrder, no entanto, sendo a raiz do agregado, precisará ser anotada com @Document e ter uma propriedade @Id, como segue:

```java
package tacos;
import java.io.Serializable;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import javax.validation.constraints.Digits;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Pattern;
import org.hibernate.validator.constraints.CreditCardNumber;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import lombok.Data;
@Data
@Document
public class TacoOrder implements Serializable {
private static final long serialVersionUID = 1L;
@Id
private String id;
private Date placedAt = new Date();
// outras propriedades omitidas por uma questão de brevidade
private List<Taco> tacos = new ArrayList<>();
public void addTaco(Taco taco) {
this.tacos.add(taco);
}
}
```

Para ser breve, cortei os vários campos de entrega e cartão de crédito. Mas pelo que sobrou, está claro que tudo o que você precisa é @Document e @Id, como nos outros
tipos de domínio. Observe, no entanto, que a propriedade id foi alterada para ser uma String (em oposição a um Long na versão JPA ou um UUID na versão Cassandra). Como eu disse antes, @Id pode ser aplicado a qualquer tipo Serializable. Mas se você escolher usar uma propriedade String como ID, você obtém o benefício do Mongo atribuir automaticamente um valor a ela quando é salva (assumindo que é nula). Ao escolher String, você obtém uma atribuição de ID gerenciada pelo banco de dados e não precisa se preocupar em definir essa propriedade manualmente. Embora existam alguns casos de uso mais avançados e incomuns que exigem mapeamento adicional, você verá que, para a maioria dos casos, @Document e @Id, juntamente com um ocasional @Field ou @Transient, são suficientes para o mapeamento do MongoDB. Eles certamente fazem o trabalho para os tipos de domínio do Taco Cloud. Tudo o que resta é escrever as interfaces do repositório.

### 4.2.3 Escrevendo interfaces de repositório do MongoDB
O Spring Data MongoDB oferece suporte automático ao repositório semelhante ao que é fornecido pelo Spring Data JPA e Spring Data Cassandra.
Você começará definindo um repositório para persistir objetos Ingredient como documentos. Como antes, você pode escrever IngredientRepository para estender CrudRepository, como mostrado aqui:

```java
package tacos.data;
import org.springframework.data.repository.CrudRepository;
import tacos.Ingredient;

public interface IngredientRepository extends CrudRepository<Ingredient, String> {
}
```

Espere um minuto! Isso parece idêntico à interface IngredientRepository que você escreveu na seção 4.1 para Cassandra! De fato, é a mesma interface, sem alterações. Isso
destaca um dos benefícios de estender o CrudRepository — ele é mais portátil em vários tipos de banco de dados e funciona igualmente bem para MongoDB e Cassandra.
Passando para a interface OrderRepository, você pode ver no seguinte snippet que é bem direto:

```java
package tacos.data;
import org.springframework.data.repository.CrudRepository;
import tacos.TacoOrder;
public interface OrderRepository extends CrudRepository<TacoOrder, String> {
}
```

Assim como IngredientRepository, OrderRepository estende CrudRepository para obter as otimizações oferecidas em seus métodos insert(). Caso contrário, não há nada terrivelmente especial sobre este repositório, em comparação com alguns dos outros repositórios que você definiu até agora. Observe, no entanto, que o parâmetro ID ao estender CrudRepository agora é String em vez de Long (como para JPA) ou UUID (como para Cassandra). Isso reflete a mudança que fizemos no TacoOrder para oferecer suporte à atribuição automática de IDs. No final, trabalhar com o Spring Data MongoDB não é drasticamente diferente dos outros projetos Spring Data com os quais trabalhamos. Os tipos de domínio são anotados de forma diferente. Mas, além do parâmetro de ID especificado ao estender CrudRepository, as interfaces do repositório são quase idênticas.

Resumo:
- O Spring Data oferece suporte a repositórios para uma variedade de bancos de dados NoSQL, incluindo Cassandra, MongoDB, Neo4j e Redis.
- O modelo de programação para criar repositórios difere muito pouco entre diferentes bancos de dados subjacentes.
- Trabalhar com bancos de dados não relacionais exige uma compreensão de como modelar dados apropriadamente para como o banco de dados finalmente armazena os dados.