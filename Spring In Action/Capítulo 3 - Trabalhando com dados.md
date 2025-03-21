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