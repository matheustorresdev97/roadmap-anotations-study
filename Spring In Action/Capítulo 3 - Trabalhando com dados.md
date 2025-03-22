Este Capítulo abrange:
	Usando o JdbcTemplate do Spring
	Criando repositórios JDBC do Spring Data
	Declarando repositórios JPA com Spring Data

A maioria dos aplicativos oferece mais do que apenas um rosto bonito. Embora a interface do usuário possa fornecer interação com um aplicativo, são os dados que ela apresenta e armazena que separam os aplicativos de sites estáticos.
No aplicativo Taco Cloud, você precisa ser capaz de manter informações
sobre ingredientes, tacos e pedidos. Sem um banco de dados para armazenar essas informações,  o aplicativo não seria capaz de progredir muito além do que você desenvolveu no capítulo 2.
Neste capítulo, você adicionará persistência de dados ao aplicativo Taco Cloud. Você começará usando o suporte Spring para JDBC (Java Database Connectivity) para eliminar o código boilerplate.  Em seguida, você retrabalhará os repositórios de dados para trabalhar com JPA (JAVA PERSISTENCE API), eliminando ainda mais código.

### 3.1 Lendo e escrevendo dados com JDBC
Por décadas, bancos de dados relacionais e SQL têm desfrutado de sua posição como a principal escolha para persistência de dados. Embora muitos tipos alternativos de bancos de dados tenham surgido nos últimos anos, o banco de dados relacional ainda é a melhor escolha para um armazenamento de dados de uso geral e provavelmente não será usurpado de sua posição tão cedo.
Quando se trata de trabalhar com dados relacionais, os desenvolvedores Java têm várias
opções. As duas escolhas mais comuns são JDBC e JPA. O Spring suporta ambos com
abstrações, tornando o trabalho com JDBC ou JPA mais fácil do que seria sem
Spring. Nesta seção, vamos nos concentrar em como o Spring suporta JDBC e, em seguida, veremos o suporte do Spring para JPA na seção 3.2. O suporte do Spring JDBC está enraizado na classe JdbcTemplate. O JdbcTemplate fornece um
meio pelo qual os desenvolvedores podem executar operações SQL em um banco de dados relacional sem toda a cerimônia e clichê normalmente necessários ao trabalhar com JDBC.
Para ter uma ideia do que o JdbcTemplate faz, vamos começar olhando um exemplo de como executar uma consulta simples em Java sem o JdbcTemplate.

Listagem 3.1 - Consultando um banco de dados sem JdbcTemplate

```java
@Override
public Optional<Ingredient> findById(String id) {
Connection connection = null;
PreparedStatement statement = null;
ResultSet resultSet = null;
try {
connection = dataSource.getConnection();
statement = connection.prepareStatement(
"select id, name, type from Ingredient where id=?");
statement.setString(1, id);
resultSet = statement.executeQuery();
Ingredient ingredient = null;
if(resultSet.next()) {
ingredient = new Ingredient(
resultSet.getString("id"),
resultSet.getString("name"),
Ingredient.Type.valueOf(resultSet.getString("type")));
}
return Optional.of(ingredient);
} catch (SQLException e) {
// ??? What should be done here ???
} finally {
if (resultSet != null) {
try {
resultSet.close();
} catch (SQLException e) {}
}
if (statement != null) {
try {
statement.close();
} catch (SQLException e) {}
}
if (connection != null) {
try {
connection.close();
} catch (SQLException e) {}
}
}
return Optional.empty();
}
```

Garanto que em algum lugar na listagem 3.1 há algumas linhas que consultam o banco de dados em busca de ingredientes. Mas aposto que você teve dificuldade em encontrar essa agulha de consulta no palheiro do JDBC. Ela é cercada por código que cria uma conexão, cria uma declaração e limpa fechando a conexão, a declaração e o conjunto de resultados.
Para piorar as coisas, várias coisas podem dar errado ao criar a conexão ou a declaração, ou ao executar a consulta. Isso requer que você capture uma SQLException, que pode ou não ser útil para descobrir o que deu errado ou como resolver o problema. SQLException é uma exceção verificada, que requer tratamento em um bloco catch. Mas os problemas mais comuns, como falha ao criar uma conexão com o banco de dados ou uma consulta digitada incorretamente, não podem ser resolvidos em um bloco catch e provavelmente serão relançados para tratamento upstream. Em contraste, considere o seguinte método que
usa o JdbcTemplate do Spring.

Listagem 3.2 -  Consultando um banco de dados com JdbcTemplate

```java
private JdbcTemplate jdbcTemplate;

public Optional<Ingredient> findById(String id) {
List<Ingredient> results = jdbcTemplate.query(
"select id, name, type from Ingredient where id=?",
this::mapRowToIngredient,
id);
return results.size() == 0 ?
Optional.empty() :
Optional.of(results.get(0));
}
private Ingredient mapRowToIngredient(ResultSet row, int rowNum)
throws SQLException {
return new Ingredient(
row.getString("id"),
row.getString("name"),
Ingredient.Type.valueOf(row.getString("type")));
}
```


O código na listagem 3.2 é claramente muito mais simples do que o exemplo JDBC bruto na listagem 3.1; não há nenhuma instrução ou conexão sendo criada. E, depois que o método é concluído, não há nenhuma limpeza desses objetos. Finalmente, não há nenhum tratamento de exceções que não podem ser tratadas corretamente em um bloco catch. O que resta é
código focado exclusivamente em executar uma consulta (a chamada para o método query() do JdbcTemplate) e mapear os resultados para um objeto Ingredient (manipulado pelo método mapRow-ToIngredient()).

O código na listagem 3.2 é um trecho do que você precisa fazer para usar o JdbcTemplate para persistir e ler dados no aplicativo Taco Cloud. Vamos dar os próximos passos necessários para equipar o aplicativo com persistência JDBC. Começaremos fazendo alguns ajustes nos objetos de domínio.

### 3.1.1 Adaptando o domínio para persistência
Ao persistir objetos em um banco de dados, geralmente é uma boa ideia ter um campo que
identifique exclusivamente o objeto. Sua classe Ingredient já tem um campo id, mas você
precisa adicionar campos id a Taco e TacoOrder também.
Além disso, pode ser útil saber quando um Taco é criado e quando um Taco-Order é feito. Você também precisará adicionar um campo a cada objeto para capturar a data e a hora em que os objetos são salvos. A listagem a seguir mostra os novos campos id e createdAt necessários na classe Taco.

Listagem 3.3 - Adicionando campos de ID e timestamp à classe Taco

```java
@Data
public class Taco {
private Long id;
private Date createdAt = new Date();
// ...
}
```

Como você usa o Lombok para gerar automaticamente métodos de acesso em tempo de execução, não há necessidade de fazer nada além de declarar as propriedades id e createdAt. Eles terão métodos getter e setter apropriados conforme necessário em tempo de execução. Mudanças semelhantes são necessárias na classe TacoOrder, conforme mostrado aqui:

```java
@Data
public class TacoOrder implements Serializable {
private static final long serialVersionUID = 1L;
private Long id;
private Date placedAt;
// ...
}
```


Novamente, o Lombok gera automaticamente os métodos de acesso, então essas são as únicas mudanças necessárias no TacoOrder. Se por algum motivo você escolher não usar o Lombok, você precisará escrever esses métodos você mesmo.
Suas classes de domínio agora estão prontas para persistência. Vamos ver como usar o Jdbc-Template para ler e gravá-las em um banco de dados.

### 3.1.2 Trabalhando com JdbcTemplate
Antes de começar a usar o JdbcTemplate, você precisa adicioná-lo ao classpath do seu projeto.
Você pode fazer isso facilmente adicionando a dependência inicial do JDBC do Spring Boot à compilação da seguinte forma:

```java
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

Você também vai precisar de um banco de dados onde seus dados serão armazenados. Para fins de desenvolvimento, um banco de dados incorporado será suficiente. Eu prefiro o banco de dados incorporado H2, então adicionei a seguinte dependência à compilação:

```java
<dependency>
<groupId>com.h2database</groupId>
<artifactId>h2</artifactId>
<scope>runtime</scope>
</dependency>
```

Por padrão, o nome do banco de dados é gerado aleatoriamente. Mas isso torna difícil determinar a URL do banco de dados se, por algum motivo, você precisar se conectar ao banco de dados usando o console H2 (que o Spring Boot DevTools habilita em http:/ /localhost:8080/h2-console). 
Então, é uma boa ideia fixar o nome do banco de dados definindo algumas propriedades em application.properties, como mostrado a seguir:

```
spring.datasource.generate-unique-name=false
spring.datasource.name=tacocloud
```

Ou, se preferir, renomeie application.properties para application.yml e adicione as propriedades no formato YAML assim:

```yaml
spring:
datasource:
generate-unique-name: false
name: tacocloud
```

A escolha entre o formato de arquivo de propriedades e o formato YAML é sua. O Spring Boot fica feliz em trabalhar com qualquer um deles. Dada a estrutura e a legibilidade aumentada do YAML, usaremos YAML para propriedades de configuração no restante do livro. Ao definir a propriedade spring.datasource.generate-unique-name como false,
estamos dizendo ao Spring para não gerar um valor aleatório exclusivo para o nome do banco de dados. Em vez disso, ele deve usar o valor definido para a propriedade spring.datasource.name. Neste caso, o nome do banco de dados será "tacocloud". Consequentemente, a URL do banco de dados será "jdbc:h2:mem:tacocloud", que você pode especificar na URL JDBC para a conexão do console H2. Mais tarde, você verá como configurar o aplicativo para usar um banco de dados externo. Mas por enquanto, vamos prosseguir para escrever um repositório que busca e salva dados do Ingredient.

DEFININDO REPOSITÓRIOS JDBC
Seu repositório Ingredient precisa executar as seguintes operações:
- Consultar todos os ingredientes em uma coleção de objetos Ingredient
- Consultar um único Ingredient pelo seu id
- Salvar um objeto Ingredient
A seguinte interface IngredienteRepository define essas três operações como declarações de método:

```java
package tacos.data;
import java.util.Optional;
import tacos.Ingredient;

public interface IngredientRepository {
	Iterable<Ingredient> findAll();
	
	Optional<Ingredient> findById(String id);
	
	Ingredient save(Ingredient ingredient);
}
```

Embora a interface capture a essência do que você precisa que um repositório de ingredientes faça, você ainda precisará escrever uma implementação de IngredientRepository que use JdbcTemplate para consultar o banco de dados. O código mostrado a seguir é o primeiro passo para escrever essa implementação.

Listagem 3.4 - Iniciando um repositório de ingredientes com JdbcTemplate

```java
package tacos.data;
import java.sql.ResultSet;
import java.sql.SQLException;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;
import tacos.Ingredient;

@Repository
public class JdbcIngredientRepository implements IngredientRepository {

private JdbcTemplate jdbcTemplate;

public JdbcIngredientRepository(JdbcTemplate jdbcTemplate) {
this.jdbcTemplate = jdbcTemplate;
}
// ...
}
```

Como você pode ver, JdbcIngredientRepository é anotado com @Repository. Esta anotação é uma das poucas anotações de estereótipo que o Spring define, incluindo @Controller e @Component. Ao anotar JdbcIngredientRepository com @Repository, você declara que ele deve ser descoberto automaticamente pela varredura de componentes do Spring e instanciado como um bean no contexto do aplicativo Spring.
Quando o Spring cria o bean JdbcIngredientRepository, ele o injeta com Jdbc-
Template. Isso ocorre porque, quando há apenas um construtor, o Spring aplica implicitamente #autowiring de dependências por meio dos parâmetros desse construtor. Se houver mais de um construtor, ou se você quiser apenas que o autowiring seja explicitamente declarado, então você pode anotar o construtor com @Autowired da seguinte forma:
```java
@Autowired
public JdbcIngredientRepository(JdbcTemplate jdbcTemplate) {
this.jdbcTemplate = jdbcTemplate;
}
```

O construtor atribui JdbcTemplate a uma variável de instância que será usada em outros métodos para consultar e inserir no banco de dados. Falando desses outros métodos, vamos dar uma olhada nas implementações de findAll() e findById(), mostradas no exemplo de código.

Listagem 3.5 -  Consultando o banco de dados com JdbcTemplate

```java
@Override
public Iterable<Ingredient> findAll() {
return jdbcTemplate.query(
"select id, name, type from Ingredient",
this::mapRowToIngredient);
}
@Override
public Optional<Ingredient> findById(String id) {
List<Ingredient> results = jdbcTemplate.query(
"select id, name, type from Ingredient where id=?",
this::mapRowToIngredient,
id);
return results.size() == 0 ?
Optional.empty() :
Optional.of(results.get(0));
}
private Ingredient mapRowToIngredient(ResultSet row, int rowNum)
throws SQLException {
return new Ingredient(
row.getString("id"),
row.getString("name"),
Ingredient.Type.valueOf(row.getString("type")));
}
```

Tanto findAll() quanto findById() usam JdbcTemplate de forma semelhante. O método findAll(), esperando retornar uma coleção de objetos, usa o método query() do JdbcTemplate. O método query() aceita o SQL para a consulta, bem como uma implementação do RowMapper do Spring com o propósito de mapear cada linha no conjunto de resultados para um objeto. query() também aceita como argumento(s) final(ais) uma lista de quaisquer parâmetros necessários na consulta. Mas, neste caso, não há parâmetros necessários. Em contraste, o método findById() precisará incluir uma cláusula where em sua consulta para comparar o valor da coluna id com o valor do parâmetro id
passado para o método. Portanto, a chamada para query() inclui, como seu parâmetro final,
o parâmetro id. Quando a consulta for realizada, o ? será substituído por este valor.  Conforme mostrado na listagem 3.5, o parâmetro RowMapper para findAll() e find-ById() é fornecido como uma referência de método para o método mapRowToIngredient(). As referências de método e lambdas do Java são convenientes ao trabalhar com JdbcTemplate como uma alternativa a uma implementação explícita do RowMapper. Se por algum motivo você quiser ou precisar de um RowMapper explícito, a seguinte implementação de findById() mostra como fazer isso:

```java
@Override
public Ingredient findById(String id) {
return jdbcTemplate.queryForObject(
"select id, name, type from Ingredient where id=?",
new RowMapper<Ingredient>() {
public Ingredient mapRow(ResultSet rs, int rowNum)
throws SQLException {
return new Ingredient(
rs.getString("id"),
rs.getString("name"),
Ingredient.Type.valueOf(rs.getString("type")));
};
}, id);
}
```

Ler dados de um banco de dados é apenas parte da história. Em algum momento, os dados devem ser gravados no banco de dados para que possam ser lidos. Vamos ver sobre a implementação do método save().

INSERINDO UMA LINHA

O método update() do JdbcTemplate pode ser usado para qualquer consulta que grave ou atualize dados no banco de dados. E, como mostrado na listagem a seguir, ele pode ser usado para inserir dados no banco de dados.

Listagem 3.6 - Inserindo dados com JdbcTemplate

```JAVA
@Override
public Ingredient save(Ingredient ingredient) {
jdbcTemplate.update(
"insert into Ingredient (id, name, type) values (?, ?, ?)",
ingredient.getId(),
ingredient.getName(),
ingredient.getType().toString());
return ingredient;
}
```

Como não é necessário mapear dados do ResultSet para um objeto, o método update() é
muito mais simples que query(). Ele requer apenas uma String contendo o SQL para executar bem como valores para atribuir a quaisquer parâmetros de consulta. Neste caso, a consulta tem três parâmetros, que correspondem aos três parâmetros finais do método save(), fornecendo o ID, nome e tipo do ingrediente.
Com JdbcIngredientRepository completo, agora você pode injetá-lo em Design-TacoController e usá-lo para fornecer uma lista de objetos Ingredient em vez de usar valores codificados (como você fez no capítulo 2). As alterações em DesignTacoController
são mostradas a seguir.

Listagem 3.7 Injetando e usando um repositório no controlador

```java
@Controller
@RequestMapping("/design")
@SessionAttributes("tacoOrder")
public class DesignTacoController {
private final IngredientRepository ingredientRepo;
@Autowired
public DesignTacoController(
IngredientRepository ingredientRepo) {
this.ingredientRepo = ingredientRepo;
}
@ModelAttribute
public void addIngredientsToModel(Model model) {
Iterable<Ingredient> ingredients = ingredientRepo.findAll();
Type[] types = Ingredient.Type.values();
for (Type type : types) {
model.addAttribute(type.toString().toLowerCase(),
filterByType(ingredients, type));
}
}
// ...
}
```


O método addIngredientsToModel() usa o método injetado findAll() do IngredientRepository para buscar todos os ingredientes do banco de dados. Ele então os filtra em tipos de ingredientes distintos antes de adicioná-los ao modelo. Agora que temos um IngredientRepository do qual buscar objetos Ingredient, também podemos simplificar o IngredientByIdConverter que criamos no capítulo 2, substituindo seus objetos Map of Ingredient codificados por uma chamada simples para o método IngredientRepository.findById(), conforme mostrado a seguir.

Listagem 3.8 -  Simplificando IngredientByIdConverter

```java
package tacos.web;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.convert.converter.Converter;
import org.springframework.stereotype.Component;
import tacos.Ingredient;
import tacos.data.IngredientRepository;
@Component
public class IngredientByIdConverter implements Converter<String, Ingredient> {
private IngredientRepository ingredientRepo;
@Autowired
public IngredientByIdConverter(IngredientRepository ingredientRepo) {
this.ingredientRepo = ingredientRepo;
}
@Override
public Ingredient convert(String id) {
return ingredientRepo.findById(id).orElse(null);
}
}
```

Você está quase pronto para iniciar o aplicativo e testar essas mudanças. Mas antes
de poder começar a ler dados da tabela Ingredient referenciada nas consultas, você
provavelmente deve criar essa tabela e preenchê-la com alguns dados de ingredientes.


### 3.1.3 Definindo um esquema e pré-carregando dados

Além da tabela Ingredient, você também vai precisar de algumas tabelas que contenham
informações de pedido e design. A Figura 3.1 ilustra as tabelas que você vai precisar, assim como os relacionamentos entre essas tabelas.
As tabelas na figura 3.1 atendem aos seguintes propósitos:
- Taco_Order - Contém detalhes essenciais do pedido
- Taco - Contém informações essenciais sobre o design de um taco
- Ingredient_Ref - Contém uma ou mais linhas para cada linha no Taco, mapeando o taco para os ingredientes para esse taco
- Ingredient - Contém informações sobre os ingredientes

![[Captura de tela de 2025-03-22 10-29-48.png]]

Em nossa aplicação, um Taco não pode existir fora do contexto de um Taco_Order. Assim,
Taco_Order e Taco são considerados membros de um agregado onde Taco_Order é
a raiz agregada. Objetos de ingrediente, por outro lado, são membros únicos de seu
próprio agregado e são referenciados pelo Taco por meio de Ingredient_Ref.
NOTA Agregados e raízes agregadas são conceitos centrais do design #orientado-a-domínio, uma abordagem de design que promove a ideia de que a estrutura e a linguagem do código de software devem corresponder ao domínio do negócio. Embora estejamos aplicando um pouco de design orientado a domínio (DDD) nos objetos de domínio
do Taco Cloud, há muito mais no DDD do que agregados e raízes agregadas.


Listagem 3.9 -  Definindo o esquema do Taco Cloud

```sql
create table if not exists Taco_Order (
id identity,
delivery_Name varchar(50) not null,
delivery_Street varchar(50) not null,
delivery_City varchar(50) not null,
delivery_State varchar(2) not null,
delivery_Zip varchar(10) not null,
cc_number varchar(16) not null,
cc_expiration varchar(5) not null,
cc_cvv varchar(3) not null,
placed_at timestamp not null
);

create table if not exists Taco (
id identity,
name varchar(50) not null,
taco_order bigint not null,
taco_order_key bigint not null,
created_at timestamp not null
);
create table if not exists Ingredient_Ref (
ingredient varchar(4) not null,
taco bigint not null,
taco_key bigint not null
);
create table if not exists Ingredient (
id varchar(4) not null,
name varchar(25) not null,
type varchar(10) not null
);
alter table Taco
add foreign key (taco_order) references Taco_Order(id);
alter table Ingredient_Ref
add foreign key (ingredient) references Ingredient(id);
```


A grande questão é onde colocar essa definição de esquema. Como se vê, o Spring Boot
responde a essa pergunta.
Se houver um arquivo chamado schema.sql na raiz do classpath do aplicativo, então
o SQL nesse arquivo será executado no banco de dados quando o aplicativo iniciar.
Portanto, você deve colocar o conteúdo da listagem 3.8 no seu projeto como um arquivo chamado schema.sql na pasta src/main/resources.
Você também precisa pré-carregar o banco de dados com alguns dados de ingredientes. Felizmente, o Spring Boot também executará um arquivo chamado data.sql da raiz do classpath quando o aplicativo iniciar. Portanto, você pode carregar o banco de dados com dados de ingredientes usando as instruções insert na próxima listagem, colocadas em src/main/resources/data.sql.


Listagem 3.10 -  Pré-carregamento do banco de dados com data.sql

```sql
delete from Ingredient_Ref;
delete from Taco;
delete from Taco_Order;
delete from Ingredient;
insert into Ingredient (id, name, type)
values ('FLTO', 'Flour Tortilla', 'WRAP');
insert into Ingredient (id, name, type)
values ('COTO', 'Corn Tortilla', 'WRAP');
insert into Ingredient (id, name, type)
values ('GRBF', 'Ground Beef', 'PROTEIN');
insert into Ingredient (id, name, type)
values ('CARN', 'Carnitas', 'PROTEIN');
insert into Ingredient (id, name, type)
values ('TMTO', 'Diced Tomatoes', 'VEGGIES');
insert into Ingredient (id, name, type)
values ('LETC', 'Lettuce', 'VEGGIES');
insert into Ingredient (id, name, type)
values ('CHED', 'Cheddar', 'CHEESE');
insert into Ingredient (id, name, type)
values ('JACK', 'Monterrey Jack', 'CHEESE');
insert into Ingredient (id, name, type)
values ('SLSA', 'Salsa', 'SAUCE');
insert into Ingredient (id, name, type)
values ('SRCR', 'Sour Cream', 'SAUCE');
```

Mesmo que você tenha desenvolvido apenas um repositório para dados de ingredientes, você pode iniciar o aplicativo Taco Cloud neste ponto e visitar a página de design para ver o JdbcIngredient-Repository em ação. Vá em frente... experimente. Quando voltar, você escreverá os repositórios para persistir dados do Taco e do TacoOrder.

### 3.1.4 Inserindo dados

Você já teve um vislumbre de como usar o JdbcTemplate para gravar dados no banco de dados. O método save() no JdbcIngredientRepository usou o método update() do JdbcTemplate para salvar objetos Ingredient no banco de dados.
Embora esse tenha sido um bom primeiro exemplo, talvez tenha sido um pouco simples demais. Como você verá em breve, salvar dados pode ser mais complexo do que o JdbcIngredientRepository precisava.
Em nosso design, TacoOrder e Taco são parte de um agregado no qual TacoOrder é
a raiz do agregado. Em outras palavras, os objetos Taco não existem fora do contexto de um
TacoOrder. Então, por enquanto, precisamos apenas definir um repositório para persistir os objetos TacoOrder e, por sua vez, os objetos Taco junto com eles. Tal repositório pode ser definido em uma interface OrderRepository como esta:

```java
package tacos.data;
import java.util.Optional;
import tacos.TacoOrder;

public interface OrderRepository {
TacoOrder save(TacoOrder order);
}
```

Parece simples o suficiente, certo? Não tão rápido. Quando você salva um TacoOrder, você também deve salvar os objetos Taco que vão com ele. E quando você salva os objetos Taco, você também precisa salvar um objeto que representa o link entre o Taco e cada Ingrediente que compõe o taco. A classe IngredientRef define esse link entre Taco
e Ingrediente da seguinte forma:

```java
package tacos;
import lombok.Data;
@Data

public class IngredientRef {
private final String ingredient;
}
```

Basta dizer que o método save() será um pouco mais interessante do que o método correspondente que você criou anteriormente para salvar um humilde objeto Ingredient.
Outra coisa que o método save() precisará fazer é determinar qual ID é atribuído ao pedido depois que ele for salvo. De acordo com o esquema, a propriedade id na tabela Taco_Order é uma identidade, o que significa que o banco de dados determinará o valor automaticamente. Mas se o banco de dados determinar o valor para você, então você precisará saber qual é esse valor para que ele possa ser retornado no objeto TacoOrder retornado do método save(). Felizmente, o Spring oferece um tipo GeneratedKeyHolder
útil que pode ajudar com isso. Mas envolve trabalhar com uma declaração preparada, como
mostrado na seguinte implementação do método save():

```java
package tacos.data;
import java.sql.Types;
import java.util.Arrays;
import java.util.Date;
import java.util.List;
import java.util.Optional;
import org.springframework.asm.Type;
import org.springframework.jdbc.core.JdbcOperations;
import org.springframework.jdbc.core.PreparedStatementCreator;
import org.springframework.jdbc.core.PreparedStatementCreatorFactory;
import org.springframework.jdbc.support.GeneratedKeyHolder;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;
import tacos.IngredientRef;
import tacos.Taco;
import tacos.TacoOrder;
@Repository
public class JdbcOrderRepository implements OrderRepository {
private JdbcOperations jdbcOperations;
public JdbcOrderRepository(JdbcOperations jdbcOperations) {
this.jdbcOperations = jdbcOperations;
}
@Override
@Transactional
public TacoOrder save(TacoOrder order) {
PreparedStatementCreatorFactory pscf =
new PreparedStatementCreatorFactory(
"insert into Taco_Order "
+ "(delivery_name, delivery_street, delivery_city, "
+ "delivery_state, delivery_zip, cc_number, "
+ "cc_expiration, cc_cvv, placed_at) "
+ "values (?,?,?,?,?,?,?,?,?)",
Types.VARCHAR, Types.VARCHAR, Types.VARCHAR,
Types.VARCHAR, Types.VARCHAR, Types.VARCHAR,
Types.VARCHAR, Types.VARCHAR, Types.TIMESTAMP
);
pscf.setReturnGeneratedKeys(true);
order.setPlacedAt(new Date());
PreparedStatementCreator psc =
pscf.newPreparedStatementCreator(
Arrays.asList(
order.getDeliveryName(),
order.getDeliveryStreet(),
order.getDeliveryCity(),
order.getDeliveryState(),
order.getDeliveryZip(),
order.getCcNumber(),
order.getCcExpiration(),
order.getCcCVV(),
order.getPlacedAt()));
GeneratedKeyHolder keyHolder = new GeneratedKeyHolder();
jdbcOperations.update(psc, keyHolder);
long orderId = keyHolder.getKey().longValue();
order.setId(orderId);
List<Taco> tacos = order.getTacos();
int i=0;
for (Taco taco : tacos) {
saveTaco(orderId, i++, taco);
}
return order;
}
}
```


Parece haver muita coisa acontecendo no método save(), mas podemos dividi-lo em apenas algumas etapas significativas. Primeiro, vocÊ cria um PreparedStatement-
CreatorFactory que descreve a consulta de inserção junto com os tipos de campos de entrada da consulta. Como mais tarde você precisará buscar o ID do pedido salvo, também precisará chamar setReturnGeneratedKeys(true).
Depois de definir o PreparedStatementCreatorFactory, você o usa para criar um PreparedStatementCreator, passando os valores do objeto TacoOrder que se persistido. O último campo fornecido ao PreparedStatementCreator é a data em que o pedido foi criado. que você também precisará definir no próprio objeto TacoOrder para que o TacoOrder retornado tenha essas informações disponíveis.
Agora que você tem essas informações disponivéis. Agora que você tem um PreparedStatementCreator em mãos, está pronto para realmente salvar os dados do pedido chamando o método update() no JdbcTemplate, passando o PreparedStatementCreator e um GeneratedKeyHolder. Após os dados do pedido terem sido salvos, o GeneratedKeyHolder conterá o valor do campo id conforme atribuído
pelo banco de dados e deve ser copiado para a propriedade id do objeto TacoOrder. Neste ponto, o pedido foi salvo, mas você também precisa salvar os objetos Taco associados ao pedido. Você pode fazer isso chamando saveTaco() para cada Taco no pedido.
O método saveTaco() é bem parecido com o método save(), como você pode ver aqui:

```java
private long saveTaco(Long orderId, int orderKey, Taco taco) {
taco.setCreatedAt(new Date());
PreparedStatementCreatorFactory pscf =
new PreparedStatementCreatorFactory(
"insert into Taco "
+ "(name, created_at, taco_order, taco_order_key) "
+ "values (?, ?, ?, ?)",
Types.VARCHAR, Types.TIMESTAMP, Type.LONG, Type.LONG
);
pscf.setReturnGeneratedKeys(true);
PreparedStatementCreator psc =
pscf.newPreparedStatementCreator(
Arrays.asList(
taco.getName(),
taco.getCreatedAt(),
orderId,
orderKey));
GeneratedKeyHolder keyHolder = new GeneratedKeyHolder();
jdbcOperations.update(psc, keyHolder);
long tacoId = keyHolder.getKey().longValue();
taco.setId(tacoId);
saveIngredientRefs(tacoId, taco.getIngredients());
return tacoId;
}
```


Passo a passo, saveTaco() espelha a estrutura de save(), embora para dados de Taco em vez de dados de TacoOrder. No final, ele faz uma chamada para saveIngredientRefs() para criar uma
linha na tabela Ingredient_Ref para vincular a linha de Taco a uma linha de Ingredient. O método saveIngredientRefs() se parece com isso:

Listagem 3.11 Injetando e usando OrderRepository

```java
package tacos.web;
import javax.validation.Valid;
import org.springframework.stereotype.Controller;
import org.springframework.validation.Errors;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.SessionAttributes;
import org.springframework.web.bind.support.SessionStatus;
import tacos.TacoOrder;
import tacos.data.OrderRepository;
@Controller
@RequestMapping("/orders")
@SessionAttributes("tacoOrder")
public class OrderController {
private OrderRepository orderRepo;
public OrderController(OrderRepository orderRepo) {
this.orderRepo = orderRepo;
}
// ...
@PostMapping
public String processOrder(@Valid TacoOrder order, Errors errors,
SessionStatus sessionStatus) {
if (errors.hasErrors()) {
return "orderForm";
}
orderRepo.save(order);
sessionStatus.setComplete();
return "redirect:/";
}
}
```


Como você pode ver, o construtor pega um OrderRepository como parâmetro e o atribui a uma variável de instância que ele usará no método processOrder(). Falando do método processOrder(), ele foi alterado para chamar o método save() no OrderRepository em vez de registrar o objeto TacoOrder. O JdbcTemplate do Spring torna o trabalho com bancos de dados relacionais significativamente mais simples do que com o JDBC vanilla. Mas mesmo com o JdbcTemplate, algumas tarefas de persistência ainda são desafiadoras, especialmente ao persistir objetos de domínio aninhados em um agregado. Se ao menos houvesse uma maneira de trabalhar com JDBC que fosse ainda mais simples. Vamos dar uma olhada no Spring Data JDBC, que torna o trabalho com #JDBC incrivelmente fácil — mesmo ao persistir agregados.

### 3.2 Trabalhando com Spring Data JDBC
O projeto #Spring-Data é um projeto guarda-chuva bastante grande que compreende vários subprojetos, a maioria dos quais é focada na persistência de dados com uma variedade de diferentes tipos de banco de dados. Alguns dos projetos Spring Data mais populares incluem:
- Spring Data JDBC—persistência JDBC em um banco de dados relacional
- Spring Data JPA—persistência JPA em um banco de dados relacional
- Spring Data MongoDB—persistência em um banco de dados de documentos Mongo
- Spring Data Neo4j—persistência em um banco de dados de gráficos Neo4j
- Spring Data Redis—persistência em um armazenamento de chave-valor Redis
- Spring Data Cassandra—persistência em um banco de dados de armazenamento de colunas Cassandra

Um dos recursos mais interessantes e úteis fornecidos pelo Spring Data para todos esses projetos é a capacidade de criar repositórios automaticamente, com base em uma interface de
especificação de repositório. Consequentemente, a persistência com projetos Spring Data tem pouca ou nenhuma lógica de persistência e envolve escrever apenas uma ou mais interfaces de repositório.
Vamos ver como aplicar o Spring Data JDBC ao nosso projeto para simplificar a persistência de dados com JDBC. Primeiro, você precisará adicionar o Spring Data JDBC à compilação do projeto.

### 3.2.1 Adicionando o Spring Data JDBC à compilação
O Spring Data JDBC está disponível como uma dependência inicial para aplicativos Spring Boot. Quando adicionado ao arquivo pom.xml do projeto, a dependência inicial se parece com o seguinte
trecho de código.

Listagem 3.12 -  Adicionando a dependência Spring Data JDBC à compilação

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
```

Você não precisará mais do iniciador JDBC que nos deu o JdbcTemplate, então você pode remover o iniciador que se parece com isso:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

No entanto, você ainda precisará de um banco de dados, então não remova a dependência H2.

### 3.2.2 Definindo interfaces de repositório
Felizmente, já criamos IngredientRepository e OrderRepository, então muito do trabalho na definição de nossos repositórios já está feito. Mas precisaremos fazer
uma mudança sutil neles para usá-los com Spring Data JDBC.
Spring Data gerará implementações automaticamente para nossas interfaces de repositório em tempo de execução. Mas ele fará isso apenas para interfaces que estendem uma das interfaces de repositório fornecidas pelo Spring Data. No mínimo, nossas interfaces de repositório precisarão estender Repository para que o Spring Data saiba criar a implementação
automaticamente. Por exemplo, aqui está como você pode escrever IngredientRepository de forma que ele estenda Repository:

```java
package tacos.data;
import java.util.Optional;
import org.springframework.data.repository.Repository;
import tacos.Ingredient;
public interface IngredientRepository extends Repository<Ingredient, String> {
Iterable<Ingredient> findAll();
Optional<Ingredient> findById(String id);
Ingredient save(Ingredient ingredient);
}
```

Como você pode ver, a interface Repository é parametrizada. O primeiro parâmetro é o tipo do objeto a ser persistido por este repositório — neste caso, Ingredient. O segundo parâmetro é o tipo do campo ID do objeto persistido. Para Ingredient, é String. Embora IngredientRepository funcione como mostrado aqui estendendo Repository, Spring Data também oferece CrudRepository como uma interface base para operações comuns, incluindo os três métodos que definimos em IngredientRepository. Então, em vez de estender Repository, geralmente é mais fácil estender CrudRepository, como mostrado a seguir.

Listagem 3.13  -  Definindo uma interface de repositório para persistir ingredientes
```java
package tacos.data;
import org.springframework.data.repository.CrudRepository;
import tacos.Ingredient;
public interface IngredientRepository extends CrudRepository<Ingredient, String> {
}
```

Da mesma forma, nosso OrderRepository pode estender CrudRepository como mostrado na próxima listagem.


Listagem 3.14 -  Definindo uma interface de repositório para persistir pedidos de taco
```java
package tacos.data;
import org.springframework.data.repository.CrudRepository;
import tacos.TacoOrder;

public interface OrderRepository extends CrudRepository<TacoOrder, Long> {
}
```


Em ambos os casos, como o CrudRepository já define os métodos que você precisa, não há necessidade de defini-los explicitamente nas interfaces IngredientRepository e OrderRepository.
E agora você tem seus dois repositórios. Você pode estar pensando que precisa escrever as implementações para ambos os repositórios, incluindo a dúzia de métodos
definidos no CrudRepository. Mas essa é a boa notícia sobre o Spring Data — não há necessidade de escrever uma implementação! Quando o aplicativo inicia, o Spring Data gera automaticamente uma implementação na hora. Isso significa que os repositórios estão prontos para uso desde o início. Basta injetá-los nos controladores e pronto.
Além disso, como o Spring Data cria automaticamente implementações dessas interfaces em tempo de execução, você não precisa mais das implementações explícitas em JdbcIngredientRepository e JdbcOrderRepository. Você pode excluir essas duas classes e nunca mais olhar para trás!

### 3.2.3 Anotando o domínio para persistência
A única outra coisa que precisaremos fazer é anotar nossas classes de domínio para que o Spring Data JDBC saiba como persisti-las. Em termos gerais, isso significa anotar
as propriedades de identidade com @Id — para que o Spring Data saiba qual campo representa a identidade do objeto — e, opcionalmente, anotar a classe com @Table.
Por exemplo, a classe TacoOrder pode ser anotada com @Table e @Id, conforme mostrado no código a seguir.

Listagem 3.15 -  Preparando a classe Taco para persistência

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
import org.springframework.data.relational.core.mapping.Table;
import lombok.Data;

@Data
@Table
public class TacoOrder implements Serializable {
private static final long serialVersionUID = 1L;
@Id
private Long id;
// ...
}
```

A anotação @Table é completamente opcional. Por padrão, o objeto é mapeado para uma tabela com base no nome da classe de domínio. Neste caso, TacoOrder é mapeado para uma tabela
chamada "Taco_Order". Se estiver tudo bem para você, então você pode deixar a anotação @Table completamente desativada ou usá-la sem parâmetros. Mas se você preferir mapeá-la para um
nome de tabela diferente, então você pode especificar o nome da tabela como um parâmetro para @Table assim:

```java
@Table("Taco_Cloud_Order")
public class TacoOrder {
...
}
```

Conforme mostrado aqui, TacoOrder será mapeado para uma tabela chamada "Taco_Cloud_Order". Quanto à  #anotação @Id, ela designa a propriedade id como sendo a identidade para um TacoOrder. Todas as outras propriedades em TacoOrder serão mapeadas automaticamente para colunas com base em seus nomes de propriedade. Por exemplo, a propriedade deliveryName será mapeada automaticamente para a coluna chamada delivery_name. Mas se você quiser definir explicitamente o mapeamento do nome da coluna, você pode anotar a propriedade com @Column assim:

```java
@Column("customer_name")
@NotBlank(message="Delivery name is required")
private String deliveryName;
```


Neste caso, @Column está especificando que a propriedade deliveryName será mapeada para a coluna cujo nome é customer_name.
Você também precisará aplicar @Table e @Id às outras classes de domínio. Isso inclui @Ingredient.

Listagem 3.16 -  Preparando a classe Ingredient para persistência

```java
package tacos;
import org.springframework.data.annotation.Id;
import org.springframework.data.domain.Persistable;
import org.springframework.data.relational.core.mapping.Table;
import lombok.AccessLevel;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
@Data
@Table
@AllArgsConstructor
@NoArgsConstructor(access=AccessLevel.PRIVATE, force=true)
public class Ingredient implements Persistable<String> {
@Id
private String id;
// ...
}
...and Taco.
```

Listagem 3.17 - Preparando a classe Taco para persistência

```java
package tacos;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;
import org.springframework.data.annotation.Id;
import org.springframework.data.relational.core.mapping.Table;
import lombok.Data;

@Data
@Table
public class Taco {
@Id
private Long id;
// ...
}
```

Quanto ao IngredientRef, ele será mapeado automaticamente para a tabela cujo nome é "Ingredient_Ref", o que é perfeito para nossa aplicação. Você pode anotá-lo com
@Table se quiser, mas não é necessário. E a tabela "Ingredient_Ref" não tem coluna de identidade, então não há necessidade de anotar nada no IngredientRef com @Id. Com essas pequenas alterações, sem mencionar a remoção completa das classes JdbcIngredientRepository e JdbcOrderRepository, agora você tem muito menoscódigo de persistência. Mesmo assim, as implementações de repositório que são geradas em tempo de execução pelo Spring Data ainda fazem tudo o que os repositórios que usam JdbcTemplate faziam.
Na verdade, eles têm potencial para fazer ainda mais, porque as duas interfaces de repositório estendem o CrudRepository, que oferece uma dúzia de operações para criar, ler, atualizar e excluir objetos.

### 3.2.4 Pré-carregamento de dados com CommandLineRunner
Ao trabalhar com JdbcTemplate, pré-carregamos os dados do Ingredient na inicialização do aplicativo usando data.sql, que foi executado no banco de dados quando o bean de origem de dados
foi criado. Essa mesma abordagem funcionará com o Spring Data JDBC. Na verdade, funcionará com qualquer mecanismo de persistência para o qual o banco de dados de apoio seja um banco de dados
relacional. Mas vamos ver outra maneira de preencher um banco de dados na inicialização que oferece um pouco mais de flexibilidade.
O Spring Boot oferece duas interfaces úteis para executar lógica quando um aplicativo é inicializado: CommandLineRunner e ApplicationRunner. Essas duas interfaces são bastante semelhantes. Ambas são interfaces funcionais que exigem que um único método run() seja implementado. Quando o aplicativo é inicializado, todos os beans no contexto do aplicativo que implementam CommandLineRunner ou ApplicationRunner terão seus métodos run() invocados após o contexto do aplicativo e todos os beans são conectados, mas antes que
qualquer outra coisa aconteça. Isso fornece um local conveniente para os dados serem carregados no banco de dados.
Como CommandLineRunner e ApplicationRunner são interfaces funcionais, eles podem ser facilmente declarados como beans em uma classe de configuração usando um método @Bean-annotated que retorna uma função lambda. Por exemplo, aqui está como você pode criar um bean CommandLineRunner de carregamento de dados:

```java
@Bean
public CommandLineRunner dataLoader(IngredientRepository repo) {
return args -> {
repo.save(new Ingredient("FLTO", "Flour Tortilla", Type.WRAP));
repo.save(new Ingredient("COTO", "Corn Tortilla", Type.WRAP));
repo.save(new Ingredient("GRBF", "Ground Beef", Type.PROTEIN));
repo.save(new Ingredient("CARN", "Carnitas", Type.PROTEIN));
repo.save(new Ingredient("TMTO", "Diced Tomatoes", Type.VEGGIES));
repo.save(new Ingredient("LETC", "Lettuce", Type.VEGGIES));
repo.save(new Ingredient("CHED", "Cheddar", Type.CHEESE));
repo.save(new Ingredient("JACK", "Monterrey Jack", Type.CHEESE));
repo.save(new Ingredient("SLSA", "Salsa", Type.SAUCE));
repo.save(new Ingredient("SRCR", "Sour Cream", Type.SAUCE));
};
}
```

Aqui, o IngredientRepository é injetado no método bean e usado dentro do lambda para criar objetos Ingredient. O método run() do CommandLineRunner
aceita um único parâmetro que é um String vararg contendo todos os argumentos da linha de comando para o aplicativo em execução. Não precisamos deles para carregar ingredientes
no banco de dados, então o parâmetro args é ignorado. Como alternativa, poderíamos ter definido o bean data-loader como uma implementação lambda do ApplicationRunner assim:

```java
@Bean
public ApplicationRunner dataLoader(IngredientRepository repo) {
return args -> {
repo.save(new Ingredient("FLTO", "Flour Tortilla", Type.WRAP));
repo.save(new Ingredient("COTO", "Corn Tortilla", Type.WRAP));
repo.save(new Ingredient("GRBF", "Ground Beef", Type.PROTEIN));
repo.save(new Ingredient("CARN", "Carnitas", Type.PROTEIN));
repo.save(new Ingredient("TMTO", "Diced Tomatoes", Type.VEGGIES));
repo.save(new Ingredient("LETC", "Lettuce", Type.VEGGIES));
repo.save(new Ingredient("CHED", "Cheddar", Type.CHEESE));
repo.save(new Ingredient("JACK", "Monterrey Jack", Type.CHEESE));
repo.save(new Ingredient("SLSA", "Salsa", Type.SAUCE));
repo.save(new Ingredient("SRCR", "Sour Cream", Type.SAUCE));
};
}
```

A principal diferença entre CommandLineRunner e ApplicationRunner está no parâmetro passado para os respectivos métodos run(). CommandLineRunner aceita um
String vararg, que é uma representação bruta de argumentos passados ​​na linha de comando. Mas ApplicationRunner aceita um parâmetro ApplicationArguments que oferece
métodos para acessar os argumentos como componentes analisados ​​da linha de comando.
Por exemplo, suponha que queremos que nosso aplicativo aceite uma linha de comando com argumentos como "--version 1.2.3" e precisamos considerar esse argumento em nosso bean carregador. Se estiver usando um CommandLineRunner, precisaríamos pesquisar o array por “--version” e então pegar o próximo valor do array. Mas com ApplicationRunner, podemos consultar os ApplicationArguments fornecidos para o argumento “--version” assim:

```java
public ApplicationRunner dataLoader(IngredientRepository repo) {
return args -> {
List<String> version = args.getOptionValues("version");
...
};
}
```


O método getOptionValues() retorna um List< String > para permitir que o argumento option seja especificado várias vezes. No caso de CommandLineRunner ou ApplicationRunner, no entanto,
não precisamos de argumentos de linha de comando para carregar dados. Então o parâmetro args é ignorado em nosso bean data-loader. O que é legal em usar CommandLineRunner ou ApplicationRunner para fazer um carregamento de dados inicial é que eles estão usando os repositórios para criar os objetos persistidos em vez de um script SQL. Isso significa que eles funcionarão igualmente bem para bancos de dados relacionais e não relacionais. Isso será útil no próximo capítulo quando vermos como usar Spring Data para persistir em bancos de dados não relacionais.
Mas antes de fazer isso, vamos dar uma olhada em outro projeto Spring Data para persistir dados em bancos de dados relacionais: Spring Data JPA.

### 3.3 Persistindo dados com Spring Data JPA
Enquanto o Spring Data JDBC facilita o trabalho de persistência de dados, a Java Persistence API (JPA) é outra opção popular para trabalhar com dados em um banco de dados relacional.
O Spring Data JPA oferece uma abordagem para persistência com JPA semelhante ao que o Spring Data JDBC nos deu para JDBC.
Para ver como o Spring Data funciona, você vai começar de novo, substituindo os repositórios baseados em JDBC do início deste capítulo por repositórios criados pelo Spring Data JPA.
Mas primeiro, você precisa adicionar o Spring Data JPA à construção do projeto.

### 3.3.1 Adicionando o Spring Data JPA ao projeto
O Spring Data JPA está disponível para aplicativos Spring Boot com o inicializador JPA. Esta dependência inicial não só traz o Spring Data JPA, mas também inclui transitivamente
o Hibernate como a implementação JPA, conforme mostrado aqui:
```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

Se você quiser usar uma implementação JPA diferente, precisará, pelo menos, excluir a dependência Hibernate e incluir a biblioteca JPA de sua escolha. Por exemplo, para usar o EclipseLink em vez do Hibernate, você precisará alterar a compilação da seguinte forma: 
```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-data-jpa</artifactId>
<exclusions>
<exclusion>
<groupId>org.hibernate</groupId>
<artifactId>hibernate-core</artifactId>
</exclusion>
</exclusions>
</dependency>
<dependency>
<groupId>org.eclipse.persistence</groupId>
<artifactId>org.eclipse.persistence.jpa</artifactId>
<version>2.7.6</version>
</dependency>
```

Observe que pode haver outras alterações necessárias, dependendo da sua escolha de implementação do JPA. Consulte a documentação da implementação JPA escolhida para detalhes. Agora, vamos revisitar seus objetos de domínio e anotá-los para persistência JPA.

### 3.3.2 Anotando o domínio como entidades
Como você já viu com o Spring Data JDBC, o Spring Data faz algumas coisas incríveis quando se trata de criar repositórios. Mas, infelizmente, não ajuda muito quando se trata de anotar seus objetos de domínio com anotações de mapeamento JPA. Você precisará abrir as classes Ingredient, Taco e TacoOrder e lançar algumas anotações. A primeira é a classe Ingredient, mostrada a seguir.

Listagem 3.18 -  Anotando Ingredient para persistência JPA
```java
package tacos;
import javax.persistence.Entity;
import javax.persistence.Id;
import lombok.AccessLevel;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@Entity
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

Para declarar isso como uma entidade JPA, Ingredient deve ser anotado com @Entity. E sua propriedaed id deve ser anotada com @Id para designá-la como a propriedade que identificará exclusivamente a entidade no banco de dados. Observe que essa anotação @Id é a variedade JPA do pacote javax.persistence, ao contrário do @Id fornecido pelo Spring Data no  pacote org.springframework.data.annotation Observe também que não precisamos mais da anotação @Table ou precisamos implementar Persistable. Embora ainda pudéssemos usar @Table aqui, ela é desnecessária ao trabalhar com JPA e o padrão é o nome da classe ("Ingredient", neste caso).  Quanto a Persistable, era necessário apenas com o Spring Data JDBC determinar se ou não uma entidade deveria ser criada nova ou atualizar uma entidade existente; o JPA classifica isso automaticamente.
Além das anotações específicas do JPA, você também notará que adicionou uma anotação @NoArgsConstructor no nível de classe. O JPA requer que as entidades tenham
um construtor sem argumentos, então o @NoArgsConstructor do Lombok faz isso para você.
Você não quer poder usá-lo, no entanto, então o torna privado definindo o atributo de acesso como AccessLevel.PRIVATE. E como você deve definir propriedades finais, você também define o atributo force como true, o que resulta no construtor gerado pelo Lombok definindo-os para um valor padrão de null, 0 ou false, dependendo do tipo de propriedade.
Você também adicionará um @AllArgsConstructor para facilitar a criação de um objeto Ingredient com todas as propriedades inicializadas.
Você também precisa de um @RequiredArgsConstructor. A anotação @Data implicitamente adiciona um construtor de argumentos obrigatórios, mas quando um @NoArgsConstructor é usado,
esse construtor é removido. Um @RequiredArgsConstructor explícito garante que você ainda terá um construtor de argumentos obrigatórios, além do construtor privado no-arguments .
Agora, vamos para a classe Taco e ver como anotá-la como uma entidade JPA.

Listagem 3.19 -  Anotando Taco como uma entidade
```java
package tacos;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.ManyToMany;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;
import lombok.Data;
@Data
@Entity
public class Taco {
@Id
@GeneratedValue(strategy = GenerationType.AUTO)
private Long id;
@NotNull
@Size(min=5, message="Name must be at least 5 characters long")
private String name;
private Date createdAt = new Date();
@Size(min=1, message="You must choose at least 1 ingredient")
@ManyToMany()
private List<Ingredient> ingredients = new ArrayList<>();
public void addIngredient(Ingredient ingredient) {
this.ingredients.add(ingredient);
}
}
```

Assim como com Ingredient, a classe Taco agora é anotada com @Entity e tem sua propriedade id anotada com @Id. Como você está contando com o banco de dados para gerar automaticamente o valor do ID, você também anota a propriedade id com @GeneratedValue, especificando uma estratégia de AUTO.
Para declarar o relacionamento entre um Taco e sua lista de Ingrediente associada, você anota os ingredientes com @ManyToMany. Um Taco pode ter muitos objetos Ingredient, e um Ingredient pode ser parte de muitos Tacos.
Finalmente, vamos anotar o objeto TacoOrder como uma entidade. A próxima listagem mostra a nova classe TacoOrder.

Listagem 3.20 Anotando TacoOrder como uma entidade JPA

```java
package tacos;
import java.io.Serializable;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

import javax.persistence.CascadeType;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

import javax.persistence.OneToMany;
import javax.validation.constraints.Digits;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Pattern;
import org.hibernate.validator.constraints.CreditCardNumber;
import lombok.Data;
@Data
@Entity
public class TacoOrder implements Serializable {
private static final long serialVersionUID = 1L;
@Id
@GeneratedValue(strategy = GenerationType.AUTO)
private Long id;
private Date placedAt = new Date();
...
@OneToMany(cascade = CascadeType.ALL)
private List<Taco> tacos = new ArrayList<>();
public void addTaco(Taco taco) {
this.tacos.add(taco);
}
}
```

Como você pode ver, as alterações no TacoOrder espelham de perto as alterações no Taco. Uma coisa significativa que vale a pena notar é que o relacionamento com a lista de objetos Taco é anotado com @OneToMany, indicando que os tacos são todos específicos para este pedido. Além disso, o atributo cascade é definido como CascadeType.ALL para que, se o pedido for excluído, seus tacos relacionados também serão excluídos.

### 3.3.3 Declarando repositórios JPA
Quando você criou as versões JdbcTemplate dos repositórios, você explicitamente declarou os métodos que queria que o repositório fornecesse. Mas com o Spring Data JDBC, você conseguiu dispensar as classes de implementação explícitas e, em vez disso, estender a interface CrudRepository. Como se vê, CrudRepository funciona igualmente bem para Spring Data JPA. Por exemplo, aqui está a nova interface IngredientRepository:

```java
package tacos.data;
import org.springframework.data.repository.CrudRepository;
import tacos.Ingredient;

public interface IngredientRepository
extends CrudRepository<Ingredient, String> {
}
```


Na verdade, a interface IngredientRepository que usaremos com o Spring Data JPA é idêntica à que definimos para uso com o Spring Data JDBC. A interface CrudRepository é comumente usada em muitos projetos do Spring Data, independentemente do mecanismo de persistência subjacente. Da mesma forma, você pode definir OrderRepository para o Spring Data JPA da mesma forma que foi para o Spring Data JDBC, como segue:

```java
package tacos.data;
import org.springframework.data.repository.CrudRepository;
import tacos.TacoOrder;
public interface OrderRepository extends CrudRepository<TacoOrder, Long> {
}
```

Os métodos fornecidos pelo CrudRepository são ótimos para persistência de propósito geral de entidades. Mas e se você tiver alguns requisitos além da persistência básica? Vamos
ver como personalizar os repositórios para executar consultas exclusivas para seu domínio.

### 3.3.4 Personalizando repositórios
Imagine que, além das operações CRUD básicas fornecidas pelo CrudRepository, você também precisa buscar todos os pedidos entregues em um determinado código postal. Como se vê, isso pode ser facilmente resolvido adicionando a seguinte declaração de método ao OrderRepository:

```java
List<TacoOrder> findByDeliveryZip(String deliveryZip);
```

Ao gerar a implementação do repositório, o Spring Data examina cada método na interface do repositório, analisa o nome do método e tenta entender o propósito do método no contexto do objeto persistido (um TacoOrder, neste caso). Em essência, o #Spring-Data define uma espécie de linguagem específica de domínio (DSL) em miniatura, onde os detalhes de persistência são expressos em assinaturas de método de repositório. O Spring Data sabe que esse método tem a intenção de encontrar Orders, porque você parametrizou CrudRepository com TacoOrder. O nome do método, findByDeliveryZip(), deixa claro que esse método deve encontrar todas as entidades TacoOrder combinando sua propriedade deliveryZip com o valor passado como um parâmetro para o método.
O método findByDeliveryZip() é bastante simples, mas o Spring Data pode lidar com nomes de métodos ainda mais interessantes também. Os métodos de repositório são compostos de um verbo, um sujeito opcional, a palavra By e um predicado. No caso de findByDeliveryZip(), o verbo é find e o predicado é DeliveryZip; o sujeito não é especificado e é implícito ser um TacoOrder.
Vamos considerar outro exemplo mais complexo. Suponha que você precise consultar todos os pedidos entregues em um determinado CEP dentro de um determinado intervalo de datas. Nesse caso, o
método a seguir, quando adicionado ao OrderRepository, pode ser útil:

```java
List<TacoOrder> readOrdersByDeliveryZipAndPlacedAtBetween(
String deliveryZip, Date startDate, Date endDate);
```

A Figura 3.2 ilustra como o Spring Data analisa e entende o método readOrdersByDeliveryZipAndPlacedAtBetween() ao gerar a implementação do repositório. Como você pode ver, o verbo em readOrdersByDeliveryZipAndPlacedAtBetween() é read. O Spring Data também entende find, read e get como sinônimos para buscar uma ou mais entidades. Como alternativa, você também pode usar count como o verbo se quiser que o método retorne apenas um int com a contagem de entidades correspondentes.

![[Captura de tela de 2025-03-22 19-17-18.png]]


Embora o assunto do método seja opcional, aqui ele diz Orders. O Spring Data ignora a maiora das palavras em um assunto, então você poderia nomear o método readPuppiesBy... e
ele ainda encontraria entidades TacoOrder, porque esse é o tipo com o qual CrudRepository é parametrizado. O predicado segue a palavra By no nome do método e é a parte mais interessante da assinatura do método. Neste caso, o predicado se refere a duas propriedades TacoOrder: deliveryZip e placedAt. A propriedade deliveryZip deve ser igual ao valor passado para o primeiro parâmetro do método. A palavra-chave Between indica que o valor de deliveryZip deve estar entre os valores passados ​​para os dois últimos parâmetros do método. Além de uma operação Equals implícita e da operação Between, as assinaturas do método Spring Data também podem incluir qualquer um dos seguintes operadores:

- IsAfter, After, IsGreaterThan, GreaterThan
- IsGreaterThanEqual, GreaterThanEqual
-  IsBefore, Before, IsLessThan, LessThan
- IsLessThanEqual, LessThanEqual
- IsBetween, Between
- IsNull, Null
- IsNotNull, NotNull
- IsIn, In
- IsNotIn, NotIn
- IsStartingWith, StartingWith, StartsWith
- IsEndingWith, EndingWith, EndsWith
- IsContaining, Containing, Contains
- IsLike, Like
-  IsNotLike, NotLike
-  IsTrue, True
- IsFalse, False
- Is, Equals
- IsNot, Not
- IgnoringCase, IgnoresCase


Como alternativas para IgnoringCase e IgnoresCase, você pode colocar AllIgnoringCase ou AllIgnoresCase no método para ignorar maiúsculas e minúsculas para todas as comparações de String.
Por exemplo, considere o seguinte método:

```java
List<TacoOrder> findByDeliveryToAndDeliveryCityAllIgnoresCase(String deliveryTo, String deliveryCity);
```

Finalmente, você também pode colocar OrderBy no final do nome do método para classificar os resultados por uma coluna especificada. Por exemplo, para ordenar pela propriedade deliveryTo, use o seguinte código:

```java
List<TacoOrder> findByDeliveryCityOrderByDeliveryTo(String city);
```

Embora a convenção de nomenclatura possa ser útil para consultas relativamente simples, não é preciso muita imaginação para ver que os nomes dos métodos podem sair do controle para consultas mais complexas. Nesse caso, sinta-se à vontade para nomear o método como quiser e anotá-lo com @Query para especificar explicitamente a consulta a ser realizada quando o método for chamado, como mostra este exemplo:

```java
@Query("Order o where o.deliveryCity='Seattle'")
List<TacoOrder> readOrdersDeliveredInSeattle();
```


Neste uso simples de @Query, você solicita todos os pedidos entregues em Seattle. Mas você pode usar @Query para realizar virtualmente qualquer consulta JPA que você possa imaginar, mesmo quando for difícil ou impossível realizar a consulta seguindo a convenção de nomenclatura.
Métodos de consulta personalizados também funcionam com Spring Data JDBC, mas com as seguintes diferenças principais:

- Todos os métodos de consulta personalizados exigem @Query. Isso ocorre porque, diferentemente do JPA, não há metadados de mapeamento para ajudar o Spring Data JDBC a inferir automaticamente a consulta do nome do método.
- Todas as consultas especificadas em @Query devem ser consultas SQL, não consultas JPA.

No próximo capítulo, expandiremos nosso uso do Spring Data para trabalhar com bancos de dados não relacionais. Quando o fizermos, você verá que os métodos de consulta personalizados funcionam de forma muito semelhante, embora a linguagem de consulta usada em @Query seja específica para o banco de dados subjacente.

### Resumo

- O JdbcTemplate do Spring simplifica muito o trabalho com JDBC.
- PreparedStatementCreator e KeyHolder podem ser usados ​​juntos quando você precisa saber o valor de um ID gerado pelo banco de dados.
- Spring Data JDBC e Spring Data JPA tornam o trabalho com dados relacionais tão fácil quanto escrever uma interface de repositório.