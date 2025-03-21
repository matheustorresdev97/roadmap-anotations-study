Este Capítulo abrange:
	Apresentação de dados do modelo no navegador
	Processamento e validação de entrada de formulário
	Escolha de uma biblioteca de modelos de visualização

As primeiras impressões são importantes. O apelo do meio-fio pode vender uma casa muito antes do comprador entrar pela porta. A pintura vermelho cereja de um carro vai virar mais cabeças do que o que está sob o capô. E a literatura está repleta de histórias de amor à primeira vista. O que está dentro é importante, mas o que está fora — o que é visto primeiro — também é importante.
Os aplicativos que você criará com o #Spring farão todos os tipos de coisas, incluindo
#processamento-de-dados, leitura de informações de um #banco-de-dados e interação com outros aplicativos. Mas a primeira impressão que os usuários do seu aplicativo terão vem da interface do usuário. E em muitos aplicativos, essa IU é um aplicativo da web apresentado em um navegador.
No capítulo 1, você criou seu primeiro controlador Spring MVC para exibir a página inicial do seu aplicativo. Mas o Spring MVC pode fazer muito mais do que simplesmente exibir conteúdo estático.
Neste capítulo, você desenvolverá a primeira grande funcionalidade em seu aplicativo Taco Cloud — a capacidade de criar tacos personalizados. Ao fazer isso, você se aprofundará no
Spring #MVC e verá como exibir dados do modelo e processar a entrada do formulário.

### 2.1 Exibindo informações

Fundamentalmente, o Taco Cloud é um lugar onde você pode pedir tacos online. Mas mais
do que isso, o Taco Cloud quer permitir que seus clientes expressem seu lado criativo e
criem tacos personalizados a partir de uma rica paleta de ingredientes. Portanto, o aplicativo da web Taco Cloud precisa de uma página que exiba a seleção
de ingredientes para os artistas de taco escolherem. As escolhas de ingredientes podem mudar a qualquer momento, então elas não devem ser codificadas em uma página HTML.
Em vez disso, a lista de ingredientes disponíveis deve ser buscada de um banco de dados e entregue à página para ser exibida ao cliente.
Em um aplicativo da web Spring, é trabalho de um controlador buscar e processar dados. E
é trabalho de uma visualização renderizar esses dados em HTML que será exibido no navegador.
Você criará os seguintes componentes para dar suporte à página de criação de tacos:
- Uma classe de domínio que define as propriedades de um ingrediente de taco
- Uma classe de controlador Spring MVC que busca informações de ingredientes de taco
- Uma classe de controlador Spring MVC que busca informações de ingredientes e as passa para a visualização
- Um modelo de visualização que renderiza uma lista de ingredientes no navegador do usuário 

![[Captura de tela de 2025-03-21 07-29-38.png]]

Figura 2.1 Um fluxo típico de solicitação do Spring MVC

Como este capítulo se concentra na estrutura da web do Spring, adiaremos qualquer coisa do banco de dados para o capítulo 3. Por enquanto, o controlador é o único responsável por fornecer os ingredientes para a visualização. No capítulo 3, você retrabalhará o controlador para colaborar com um repositório que busca dados de ingredientes de um banco de dados. Antes de escrever o #controlador e a visualização, vamos definir o tipo de #domínio que representa um ingrediente. Isso estabelecerá uma base na qual você pode desenvolver seus componentes da web.

### 2.1.1 Estabelecendo o domínio
O domínio de um aplicativo é a área de assunto que ele aborda — as ideias e conceitos
que influenciam a compreensão do aplicativo. No aplicativo Taco Cloud,
o domínio inclui objetos como designs de tacos, os ingredientes dos quais esses designs
são compostos, clientes e pedidos de tacos feitos pelos clientes.
A Figura 2.2 mostra essas entidades e como elas estão relacionadas.

![[Captura de tela de 2025-03-21 07-33-06.png]]

Para começar, vamos nos concentrar nos ingredientes do taco. No seu domínio, os ingredientes do taco são objetos bem simples. Cada um tem um nome e um tipo para que possa ser categorizado visualmente (proteínas, queijos, molhos e assim por diante). Cada um também tem um ID pelo qual pode ser facilmente e inequivocamente referenciado. A classe Ingredient a seguir define o objeto de domínio que você precisa.

Listagem 2.1 Definindo ingredientes para tacos
```java

package tacos;
import lombok.Data;

@Data
public class Ingredient {
	private final String id;
	private final String name;
	private final Type type;

	public enum Type {
	WRAP, PROTEIN, VEGGIES, CHEESE, SAUCE
	}
}
```

Como você pode ver, esta é uma classe de domínio Java comum, definindo as três propriedades necessárias para descrever um ingrediente. Talvez a coisa mais incomum sobre a classe Ingredient conforme definida na listagem 2.1 é que parece estar faltando o conjunto usual de métodos getter e setter, sem mencionar métodos úteis como equals(), hashCode(), toString() e outros.
Você não os vê na listagem em parte para economizar espaço, mas também porque você está usando uma biblioteca incrível chamada #Lombok para gerar automaticamente esses métodos em tempo de compilação para que eles estejam disponíveis em tempo de execução. Na verdade, a anotação @Data no nível de classe é fornecida pelo Lombok e diz ao Lombok para gerar todos esses métodos ausentes, bem como um construtor que aceita todas as propriedades finais como argumentos. Ao usar o Lombok, você pode manter o código para Ingredient fino e enxuto. Lombok não é uma biblioteca Spring, mas é tão incrivelmente útil que acho difícil desenvolver sem ela. Além disso, é um salva-vidas quando preciso manter exemplos de código em um livro curto e agradável. Para usar o Lombok, você precisará adicioná-lo como uma dependência em seu projeto. Se você estiver
usando o Spring Tool Suite, é fácil clicar com o botão direito do mouse no arquivo pom.xml e
selecionar Adicionar Starters no menu de contexto do Spring. A mesma seleção de dependências que você recebeu no capítulo 1 (na figura 1.4) aparecerá, dando a você a chance de adicionar ou alterar suas dependências selecionadas. Encontre o Lombok em Ferramentas do desenvolvedor, certifique-se de que ele esteja selecionado e clique em OK; o Spring Tool Suite o adiciona automaticamente à sua especificação de construção.
Como alternativa, você pode adicioná-lo manualmente com a seguinte entrada em pom.xml:

```xml
<dependency>
<groupId>org.projectlombok</groupId>
<artifactId>lombok</artifactId>
</dependency>
```


Se você decidir adicionar manualmente o Lombok à sua compilação, você também vai querer excluí-lo do plugin Spring Boot Maven na seção < build > do arquivo pom.xml:

```xml
<build>
<plugins>
<plugin>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-maven-plugin</artifactId>
<configuration>
<excludes>
<exclude>
<groupId>org.projectlombok</groupId>
<artifactId>lombok</artifactId>
</exclude>
</excludes>
</configuration>
</plugin>
</plugins>
</build>
```

A mágica do Lombok é aplicada em tempo de compilação, então não há necessidade de que ele esteja disponível em tempo de execução.
Excluí-lo dessa forma o mantém fora do arquivo JAR ou WAR resultante.
A dependência do Lombok fornece anotações do Lombok (como @Data) em tempo de desenvolvimento e com geração automática de métodos em tempo de compilação.
Mas você também precisará adicionar o Lombok como uma extensão em seu IDE, ou seu IDE reclamará, com erros sobre métodos ausentes e propriedades finais que não estão sendo definidas. Visite https://projectlombok.org/ para descobrir como instalar o Lombok em seu IDE de escolha.
Acho que você achará o Lombok muito útil, mas saiba que ele é opcional. Você não
precisa dele para desenvolver aplicativos Spring, então se preferir não usá-lo, sinta-se à vontade para escrever esses métodos ausentes manualmente. Vá em frente... Eu espero.
Os ingredientes são os blocos de construção essenciais de um taco. Para capturar como esses ingredientes são reunidos, definiremos a classe de domínio Taco, conforme mostrado a seguir.

Listagem 2.2 - Um objeto de domínio definindo um design de taco

```java
package tacos;
import java.util.List;
import lombok.Data;

@Data
public class Taco {
private String name;
private List<Ingredient> ingredients;
}

```

Como você pode ver, Taco é um objeto de domínio Java direto com algumas propriedades. Como Ingredient, a classe Taco é anotada com @Data para que o Lombok gere automaticamente métodos JavaBean essenciais para você em tempo de compilação.
Agora que definimos Ingredient e Taco, precisamos de mais uma classe de domínio
que defina como os clientes especificam os tacos que desejam pedir, junto com informações de pagamento e entrega. Esse é o trabalho da classe TacoOrder, mostrada aqui.

Listagem 2.3 - Um objeto de domínio para pedidos de tacos

```java
package tacos;
import java.util.List;
import java.util.ArrayList;
import lombok.Data;

@Data
public class TacoOrder {
private String deliveryName;
private String deliveryStreet;
private String deliveryCity;
private String deliveryState;
private String deliveryZip;
private String ccNumber;
private String ccExpiration;
private String ccCVV;

private List<Taco> tacos = new ArrayList<>();

public void addTaco(Taco taco) {
	this.tacos.add(taco);
}

```

Além de ter mais propriedades do que Ingredient ou Taco, não há nada
particularmente novo para discutir sobre TacoOrder. É uma classe de domínio simples com nove propriedades: cinco para informações de entrega, três para informações de pagamento e uma que é a lista de objetos Taco que compõem o pedido.
Há também um método addTaco() que é adicionado para a conveniência de adicionar tacos ao pedido.
Agora que os tipos de domínio estão definidos, estamos prontos para colocá-los para funcionar.
Vamos adicionar alguns controladores para lidar com solicitações da web no aplicativo.

### 2.1.2 Criando uma classe de controlador
Os controladores são os principais participantes do framework MVC do Spring. Sua principal função é lidar com solicitações #HTTP e entregar uma solicitação a uma visualização para renderizar HTML (exibido no navegador) ou gravar dados diretamente no corpo de uma resposta #RESTful . Neste capítulo, estamos nos concentrando nos tipos de controladores que usam visualizações para produzir conteúdo para navegadores da web. Quando chegarmos ao capítulo 7, veremos como escrever controladores que lidam com solicitações em uma API REST.
Para o aplicativo Taco Cloud, você precisa de um controlador simples que fará o
seguinte:
- Lidar com solicitações HTTP GET onde o caminho da solicitação é /design
- Construir uma lista de ingredientes
- Entregar a solicitação e os dados do ingrediente a um modelo de visualização para ser renderizado como HTML e enviado ao navegador da web solicitante

A classe DesignTacoController na próxima listagem aborda esses requisitos.

Listagem 2.4 O início de uma classe controladora Spring
```java
package tacos.web;
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.SessionAttributes;
import lombok.extern.slf4j.Slf4j;
import tacos.Ingredient;
import tacos.Ingredient.Type;
import tacos.Taco;

@Slf4j
@Controller
@RequestMapping("/design")
@SessionAttributes("tacoOrder")
public class DesignTacoController {
@ModelAttribute
public void addIngredientsToModel(Model model) {
List<Ingredient> ingredients = Arrays.asList(
new Ingredient("FLTO", "Flour Tortilla", Type.WRAP),
new Ingredient("COTO", "Corn Tortilla", Type.WRAP),
new Ingredient("GRBF", "Ground Beef", Type.PROTEIN),
new Ingredient("CARN", "Carnitas", Type.PROTEIN),
new Ingredient("TMTO", "Diced Tomatoes", Type.VEGGIES),
new Ingredient("LETC", "Lettuce", Type.VEGGIES),
new Ingredient("CHED", "Cheddar", Type.CHEESE),
new Ingredient("JACK", "Monterrey Jack", Type.CHEESE),
new Ingredient("SLSA", "Salsa", Type.SAUCE),
new Ingredient("SRCR", "Sour Cream", Type.SAUCE)
);
Type[] types = Ingredient.Type.values();
for (Type type : types) {
model.addAttribute(type.toString().toLowerCase(),
filterByType(ingredients, type));
}
}
@ModelAttribute(name = "tacoOrder")
public TacoOrder order() {
return new TacoOrder();
}
@ModelAttribute(name = "taco")
public Taco taco() {
return new Taco();
}
@GetMapping
public String showDesignForm() {
return "design";
}
private Iterable<Ingredient> filterByType(
List<Ingredient> ingredients, Type type) {
return ingredients
.stream()
.filter(x -> x.getType().equals(type))
.collect(Collectors.toList());
}
}
```

A primeira coisa a ser notada sobre DesignTacoController é o conjunto de anotações aplicadas no nível da classe. A primeira, @Slf4j, é uma #anotação fornecida pelo Lombok que, no momento da compilação, gerará automaticamente uma propriedade estática #SLF4J (Simple Logging Facade for Java, https://www.slf4j.org/) Logger na classe. Esta modesta anotação tem o mesmo efeito que se você adicionasse explicitamente as seguintes linhas dentro da classe:

```java
private static final org.slf4j.Logger log =
org.slf4j.LoggerFactory.getLogger(DesignTacoController.class);
```

Você usará este Logger um pouco mais tarde.
A próxima anotação aplicada a DesignTacoController é @Controller. Esta
anotação serve para identificar esta classe como um controlador e marcá-la como candidata para varredura de componentes, para que o Spring a descubra e crie automaticamente uma instância de DesignTacoController como um #bean no contexto do aplicativo Spring. DesignTacoController também é anotado com @RequestMapping. A anotação @RequestMapping, quando aplicada no nível de classe, especifica o tipo de solicitações que este controlador manipula.
Neste caso, ela especifica que DesignTacoController manipulará solicitações cujo caminho começa com /design.
Finalmente, você vê que DesignTacoController é anotado com @SessionAttributes ("tacoOrder"). Isso indica que o objeto TacoOrder que é colocado no modelo um
pouco mais tarde na classe deve ser mantido em sessão. Isso é importante porque a criação de um taco também é o primeiro passo na criação de um pedido, e o pedido que criamos precisará ser carregado na sessão para que possa abranger várias solicitações.

### MANIPULANDO UMA SOLICITAÇÃO GET
A especificação @RequestMapping de nível de classe é refinada com a anotação @GetMapping que adorna o método showDesignForm(). @GetMapping, pareado com o @RequestMapping de nível de classe, especifica que quando uma solicitação #HTTP #GET é recebida para /design, o Spring MVC chamará showDesignForm() para manipular a solicitação. @GetMapping é apenas um membro de uma família de anotações de mapeamento de solicitação.
A Tabela 2.1 lista todas as anotações de mapeamento de solicitação disponíveis no #Spring-MVC.

Tabela 2.1 -Anotações de mapeamento de solicitação do Spring MVC


| Annotation      | Description                             |
| --------------- | --------------------------------------- |
| @RequestMapping | Tratamento de solicitações de uso geral |
| @GetMapping     | Lida com solicitações HTTP GET          |
| @PostMapping    | Lida com solicitações HTTP POST         |
| @PutMapping     | Lida com solicitações HTTP PUT          |
| @DeleteMapping  | Lida com solicitações HTTP DELETE       |
| @PatchMapping   | Lida com solicitações HTTP PATCH        |
Quando showDesignForm() manipula uma solicitação GET para /design, ele realmente não faz muito. A principal coisa que ele faz é retornar um valor String de "design", que é o nome lógico da visualização que será usada para renderizar o modelo para o navegador. Mas antes de fazer isso, ele também preenche o Model fornecido com um objeto Taco vazio sob uma chave cujo nome é "design". Isso permitirá que o formulário tenha uma lousa em branco na qual criar uma obra-prima de taco. Parece que uma solicitação GET para /design não faz muito. Mas, pelo contrário, há um pouco mais envolvido do que o que é encontrado no método showDesignForm().
Você também notará um método chamado addIngredientsToModel() que é anotado com
@ModelAttribute. Este método também será invocado quando uma solicitação for manipulada e construirá uma lista de objetos Ingredient para serem colocados no modelo. A lista está codificada por enquanto. Quando chegarmos ao capítulo 3, você extrairá a lista de ingredientes de taco disponíveis de um banco de dados.
Assim que a lista de ingredientes estiver pronta, as próximas linhas de addIngredientsTo-
Model() filtram a lista por tipo de ingrediente usando um método auxiliar chamado filterBy-
Type(). Uma lista de tipos de ingredientes é então adicionada como um atributo ao objeto Model que será passado para showDesignForm(). Model é um objeto que transporta dados entre um controlador e qualquer visualização encarregada de renderizar esses dados. Por fim, os dados que são colocados nos atributos Model são copiados para os atributos de solicitação do servlet, onde a visualização pode encontrá-los e usá-los para renderizar uma página no navegador do usuário.
Após addIngredientsToModel(), há mais dois métodos que também são anotados com @ModelAttribute. Esses métodos são muito mais simples e criam apenas um novo
objeto TacoOrder e Taco para colocar no modelo. O objeto TacoOrder, mencionado
anteriormente na anotação @SessionAttributes, mantém o estado do pedido que está sendo criado enquanto o usuário cria tacos em várias solicitações. O objeto Taco é colocado no modelo para que a visualização renderizada em resposta à solicitação GET para /design tenha um objeto não nulo para exibir.
Seu DesignTacoController está realmente começando a tomar forma. Se você fosse executar o aplicativo agora e apontar seu navegador para o caminho /design, o showDesignForm() e addIngredientsToModel() do DesignTaco-Controller seriam acionados,
colocando ingredientes e um Taco vazio no modelo antes de passar a solicitação
para a visualização. Mas como você ainda não definiu a visualização, a solicitação tomaria um desvio horrível, resultando em um erro HTTP 500 (Erro Interno do Servidor). Para corrigir isso, vamos mudar nossa atenção para a visualização onde os dados serão decorados com HTML para serem apresentados no navegador da web do usuário.

### 2.1.3 Projetando a visualização
Depois que o controlador termina seu trabalho, é hora de a visualização começar. O Spring
oferece várias opções excelentes para definir visualizações, incluindo JavaServer Pages (JSP),Thymeleaf, FreeMarker, Mustache e modelos baseados em Groovy. Por enquanto, usaremosThymeleaf, a escolha que fizemos no capítulo 1 ao iniciar o projeto. Consideraremosalgumas das outras opções na seção 2.5.
Já adicionamos o Thymeleaf como uma dependência no capítulo 1. No tempo de execução,
a autoconfiguração do Spring Boot vê que o Thymeleaf está no classpath e cria automaticamente os beans que suportam visualizações do Thymeleaf para o Spring MVC.
Bibliotecas de visualização como o Thymeleaf são projetadas para serem desacopladas de qualquer estrutura da web específica. Como tal, elas não têm conhecimento da abstração do modelo do Spring e são incapazes de trabalhar com os dados que o controlador coloca no Model. Mas eles podem trabalhar com atributos de solicitação de servlet. Portanto, antes que o Spring entregue a solicitação para uma visão, ele copia os dados do modelo em atributos de solicitação aos quais o Thymeleaf e outras opções de modelagem de visualização têm acesso imediato.
Os modelos do Thymeleaf são apenas HTML com alguns atributos de elementos adicionais que orientam um modelo na renderização de dados de solicitação. Por exemplo, se houvesse um atributo de solicitação cuja chave é "mensagem", e você quisesse que ele fosse renderizado em uma tag HTML < p > pelo Thymeleaf, você escreveria o seguinte em seu modelo do Thymeleaf: 

```html
<p th:text="${message}">placeholder message</p>
```

Quando o modelo é renderizado em #HTML, o corpo do elemento < p > será substituído pelo valor do atributo de solicitação do servlet cuja chave é "mensagem". O atributo th:text é um atributo de namespace do Thymeleaf que realiza a substituição. O operador ${} informa para usar o valor de um atributo de solicitação ("mensagem", neste caso).

O Thymeleaf também oferece outro atributo, th:each, que itera sobre uma coleção de
elementos, renderizando o HTML uma vez para cada item na coleção. Este atributo
será útil ao projetar sua visualização para listar ingredientes de taco do modelo. Por
exemplo, para renderizar apenas a lista de ingredientes "wrap", você pode usar o seguinte snippet de HTML:

```html
<h3>Designate your wrap:</h3>
<div th:each="ingredient : ${wrap}">
<input th:field="*{ingredients}" type="checkbox"
th:value="${ingredient.id}"/>
<span th:text="${ingredient.name}">INGREDIENT</span><br/>
</div>
```

Aqui, você usa o atributo th:each na tag < div > para repetir a renderização do < div >
uma vez para cada item na coleção encontrado no atributo de solicitação de wrap. Em cada iteração, o item ingrediente é vinculado a uma variável do Thymeleaf chamada ingrediente.
Dentro do elemento < div > há um elemento < input > de caixa de seleção e um elemento < span >
para fornecer um rótulo para a caixa de seleção. A caixa de seleção usa o th:value do Thymeleaf para definir
o atributo value do elemento < input > renderizado para o valor encontrado na propriedade
id do ingrediente. O atributo th:field define o atributo name do elemento < input > e é usado para lembrar se a caixa de seleção está marcada ou não. Quando adicionarmos
validação posteriormente, isso garantirá que a caixa de seleção mantenha seu estado caso o formulário precise ser exibido novamente após um erro de validação. O elemento < span > usa th:text para substituir o texto do placeholder "INGREDIENT" pelo valor da propriedade name do ingrediente. Quando renderizado com dados de modelo reais, uma iteração desse loop < div > pode ficar assim:

```html
<div>
<input name="ingredients" type="checkbox" value="FLTO" />
<span>Flour Tortilla</span><br/>
</div>
```

Por fim, o snippet Thymeleaf anterior é apenas parte de um formulário HTML maior
por meio do qual seus usuários artistas de taco enviarão suas criações saborosas. O modelo Thymeleaf completo, incluindo todos os tipos de ingredientes e o formulário, é mostrado na listagem a seguir.

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
xmlns:th="http://www.thymeleaf.org">
<head>
<title>Taco Cloud</title>
<link rel="stylesheet" th:href="@{/styles.css}" />
</head>
<body>
<h1>Design your taco!</h1>
<img th:src="@{/images/TacoCloud.png}"/>
<form method="POST" th:object="${taco}">
<div class="grid">
<div class="ingredient-group" id="wraps">
<h3>Designate your wrap:</h3>
<div th:each="ingredient : ${wrap}">
<input th:field="*{ingredients}" type="checkbox"
th:value="${ingredient.id}"/>
<span th:text="${ingredient.name}">INGREDIENT</span><br/>
</div>
</div>
<div class="ingredient-group" id="proteins">
<h3>Pick your protein:</h3>
<div th:each="ingredient : ${protein}">
<input th:field="*{ingredients}" type="checkbox"
th:value="${ingredient.id}"/>
<span th:text="${ingredient.name}">INGREDIENT</span><br/>
</div>
</div>
<div class="ingredient-group" id="cheeses">
<h3>Choose your cheese:</h3>
<div th:each="ingredient : ${cheese}">
<input th:field="*{ingredients}" type="checkbox"
th:value="${ingredient.id}"/>
<span th:text="${ingredient.name}">INGREDIENT</span><br/>
</div>
</div>
<div class="ingredient-group" id="veggies">
<h3>Determine your veggies:</h3>
<div th:each="ingredient : ${veggies}">
<input th:field="*{ingredients}" type="checkbox"
th:value="${ingredient.id}"/>
<span th:text="${ingredient.name}">INGREDIENT</span><br/>
</div>
</div>
<div class="ingredient-group" id="sauces">
<h3>Select your sauce:</h3>
<div th:each="ingredient : ${sauce}">
<input th:field="*{ingredients}" type="checkbox"
th:value="${ingredient.id}"/>
<span th:text="${ingredient.name}">INGREDIENT</span><br/>
</div>
</div>
</div>
<div>
<h3>Name your taco creation:</h3>
<input type="text" th:field="*{name}"/>
<br/>
<button>Submit Your Taco</button>
</div>
</form>
</body>
```

Como você pode ver, você repete o snippet < div > para cada um dos tipos de ingredientes e inclui um botão Enviar e um campo onde o usuário pode nomear sua criação.
Também vale a pena notar que o modelo completo inclui a imagem do logotipo do Taco Cloud e uma referência < link > a uma folha de estilo.2 Em ambos os casos, o operador @{} do Thymeleaf é usado para produzir um caminho relativo ao contexto para os artefatos estáticos que essas tags estão referenciando. Como você aprendeu no capítulo 1, o conteúdo estático em um aplicativo Spring Boot é servido do diretório /static na raiz do classpath.
Agora que seu controlador e visualização estão completos, você pode iniciar o aplicativo para ver os frutos do seu trabalho. Temos muitas maneiras de executar um aplicativo Spring Boot. No capítulo 1, mostrei como executar o aplicativo clicando no botão Iniciar no
Painel do Spring Boot. Não importa como você inicia o aplicativo Taco Cloud, uma vez
ele iniciado, aponte seu navegador para http://localhost:8080/design. Você deve ver uma página que se parece com a figura 2.3.
Está parecendo bom! Um artista de tacos que visita seu site recebe um formulário contendo
uma paleta de ingredientes de tacos a partir dos quais ele pode criar sua obra-prima. Mas o que acontece quando ele clica no botão Enviar seu Taco?
Seu DesignTacoController ainda não está pronto para aceitar criações de tacos. Se o formulário de design for enviado, o usuário verá um erro. (Especificamente, será um
erro HTTP 405: Método de solicitação “POST” não suportado.) Vamos consertar isso escrevendo mais algum código de controlador que lida com o envio do formulário.

### 2.2 Processando envio de formulário
Se você der outra olhada na tag < form > em sua visualização, verá que seu atributo method
está definido como POST. Além disso, o < form > não declara um atributo action. Isso
significa que quando o formulário é enviado, o navegador reunirá todos os dados no
formulário e os enviará ao servidor em uma solicitação HTTP POST para o mesmo caminho para o qual uma solicitação GET exibiu o formulário — o caminho /design.
Portanto, você precisa de um método manipulador de controlador na extremidade receptora dessa solicitação POST. Você precisa escrever um novo método manipulador em DesignTacoController que manipule uma solicitação POST para /design.

![[Captura de tela de 2025-03-21 08-25-05.png]]

Na listagem 2.4, você usou a anotação @GetMapping para especificar que o método showDesign-Form() deve manipular solicitações HTTP GET para /design. Assim como @GetMapping manipula solicitações GET, você pode usar @PostMapping para manipular solicitações POST. Para manipular envios de design de tacos, adicione o método processTaco() na listagem a seguir ao DesignTacoController.

Listagem 2.6 - Manipulando requisições POST com @PostMapping

```java
@PostMapping
public String processTaco(Taco taco,
@ModelAttribute TacoOrder tacoOrder) {
tacoOrder.addTaco(taco);
log.info("Processing taco: {}", taco);
return "redirect:/orders/current";
}
```

Conforme aplicado ao método processTaco(), @PostMapping coordena com o nível de classe @RequestMapping para indicar que processTaco() deve manipular solicitações POST para /design.
É exatamente disso que você precisa para processar as criações enviadas de um artista de tacos.
Quando o formulário é enviado, os campos no formulário são vinculados às propriedades de um objeto Taco (cuja classe é mostrada na próxima listagem) que é passado como um parâmetro para processTaco(). A partir daí, o método processTaco() pode fazer o que quiser
com o objeto Taco. Nesse caso, ele adiciona o Taco ao objeto TacoOrder passado como um
parâmetro para o método e então o registra. O @ModelAttribute aplicado ao parâmetro Taco-Order indica que ele deve usar o objeto TacoOrder que foi colocado
no modelo por meio do método order() anotado com @ModelAttribute mostrado anteriormente na listagem 2.4. Se você olhar novamente para o formulário na listagem 2.5, verá vários elementos de caixa de seleção, todos com o nome ingredientes e um elemento de entrada de texto chamado nome. Esses campos no formulário correspondem diretamente às propriedades ingredientes e nome da classe Taco. O campo nome no formulário precisa capturar apenas um valor textual simples. Portanto, a propriedade nome do Taco é do tipo String. As caixas de seleção ingredientes também têm valores textuais
, mas como zero ou muitos deles podem ser selecionados, a propriedade ingredientes
à qual eles estão vinculados é uma List< Ingredient > que capturará cada um dos
ingredientes escolhidos.
Mas espere. Se as caixas de seleção ingredientes tiverem valores textuais (por exemplo, String), mas o objeto Taco representa uma lista de ingredientes como List< Ingredient >, então não há uma incompatibilidade? Como uma lista textual como ["FLTO", "GRBF", "LETC"] pode ser vinculada a uma lista de objetos Ingredient que são objetos mais ricos contendo não apenas um ID, mas também um nome descritivo e tipo de ingrediente?
É aí que um conversor é útil. Um conversor é qualquer classe que implementa
a interface Converter do Spring e implementa seu método convert() para pegar um
valor e convertê-lo em outro. Para converter uma String em um Ingredient, usaremos o
IngredientByIdConverter da seguinte forma.

Listagem 2.7 - Convertendo strings em ingredientes

```java
package tacos.web;
import java.util.HashMap;
import java.util.Map;
import org.springframework.core.convert.converter.Converter;
import org.springframework.stereotype.Component;
import tacos.Ingredient;
import tacos.Ingredient.Type;
@Component
public class IngredientByIdConverter implements Converter<String, Ingredient> {
private Map<String, Ingredient> ingredientMap = new HashMap<>();
public IngredientByIdConverter() {
ingredientMap.put("FLTO",
new Ingredient("FLTO", "Flour Tortilla", Type.WRAP));
ingredientMap.put("COTO",
new Ingredient("COTO", "Corn Tortilla", Type.WRAP));
ingredientMap.put("GRBF",
new Ingredient("GRBF", "Ground Beef", Type.PROTEIN));
ingredientMap.put("CARN",
new Ingredient("CARN", "Carnitas", Type.PROTEIN));
ingredientMap.put("TMTO",
new Ingredient("TMTO", "Diced Tomatoes", Type.VEGGIES));
ingredientMap.put("LETC",
new Ingredient("LETC", "Lettuce", Type.VEGGIES));
ingredientMap.put("CHED",
new Ingredient("CHED", "Cheddar", Type.CHEESE));
ingredientMap.put("JACK",
new Ingredient("JACK", "Monterrey Jack", Type.CHEESE));
ingredientMap.put("SLSA",
new Ingredient("SLSA", "Salsa", Type.SAUCE));
ingredientMap.put("SRCR",
new Ingredient("SRCR", "Sour Cream", Type.SAUCE));
}
@Override
public Ingredient convert(String id) {
return ingredientMap.get(id);
}
}
```

Como ainda não temos um banco de dados do qual extrair objetos Ingredient, o construtor de IngredientByIdConverter cria um Map com chave em uma String que é o
ID do ingrediente e cujos valores são objetos Ingredient. No capítulo 3, adaptaremos esse
conversor para extrair os dados do ingrediente de um banco de dados em vez de ser codificado como isso. O método convert() então simplesmente pega uma String que é o ID do ingrediente e a usa para procurar o Ingredient no mapa. Observe que o IngredientByIdConverter é anotado com @Component para torná-lo descobrível como um bean no contexto do aplicativo Spring. A autoconfiguração do Spring Boot descobrirá isso e quaisquer outros beans Converter e os registrará automaticamente com o Spring MVC para serem usados ​​quando a conversão de parâmetros de solicitação para propriedades vinculadas for necessária. Por enquanto, o método processTaco() não faz nada com o objeto Taco. Na verdade, ele não faz muita coisa. Tudo bem. No capítulo 3, você adicionará alguma lógica de persistência que salvará o Taco enviado em um banco de dados.
Assim como no método showDesignForm(), processTaco() termina retornando um
valor String. E assim como showDesignForm(), o valor retornado indica uma visualização
que será mostrada ao usuário. Mas o que é diferente é que o valor retornado de
processTaco() é prefixado com "redirect:", indicando que esta é uma visualização de redirecionamento. Mais especificamente, ele indica que após processTaco() ser concluído, o navegador do usuário deve ser redirecionado para o caminho relativo /orders/current.
A ideia é que após criar um taco, o usuário seja redirecionado para um formulário de pedido
do qual ele pode fazer um pedido para que suas criações de taco sejam entregues. Mas você ainda não tem um controlador que lidará com uma solicitação para /orders/current.
Dado o que você sabe agora sobre @Controller, @RequestMapping e @Get-Mapping, você pode facilmente criar tal controlador. Pode parecer algo como a listagem a seguir.

Listagem 2.8 - Um controlador para apresentar um formulário de pedido de taco

```java
package tacos.web;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.SessionAttributes;
import org.springframework.web.bind.support.SessionStatus;
import lombok.extern.slf4j.Slf4j;
import tacos.TacoOrder;
@Slf4j
@Controller
@RequestMapping("/orders")
@SessionAttributes("tacoOrder")
public class OrderController {
@GetMapping("/current")
public String orderForm() {
return "orderForm";
}
}
```

Mais uma vez, você usa a anotação @Slf4j do Lombok para criar um objeto Logger SLF4J gratuito em tempo de compilação. Você usará este Logger em um momento para registrar os detalhes do pedido que foi enviado.
O @RequestMapping de nível de classe especifica que quaisquer métodos de tratamento de solicitação neste controlador manipularão solicitações cujo caminho começa com /orders. Quando combinado com o @GetMapping de nível de método, ele especifica que o método orderForm() manipulará solicitações HTTP GET para /orders/current.
Quanto ao método orderForm() em si, ele é extremamente básico, retornando apenas um
nome de visualização lógica de orderForm. Depois que você tiver uma maneira de persistir criações de tacos em um banco de dados no capítulo 3, você revisitará este método e o modificará para preencher o modelo com uma lista de objetos Taco a serem colocados no pedido.
A visualização orderForm é fornecida por um modelo Thymeleaf chamado orderForm.html,
que é mostrado a seguir.

Listagem 2.9 - Uma visualização do formulário de pedido de taco

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
xmlns:th="http://www.thymeleaf.org">
<head>
<title>Taco Cloud</title>
<link rel="stylesheet" th:href="@{/styles.css}" />
</head>
<body>
<form method="POST" th:action="@{/orders}" th:object="${tacoOrder}">
<h1>Order your taco creations!</h1>
<img th:src="@{/images/TacoCloud.png}"/>
<h3>Your tacos in this order:</h3>
<a th:href="@{/design}" id="another">Design another taco</a><br/>
<ul>
<li th:each="taco : ${tacoOrder.tacos}">
<span th:text="${taco.name}">taco name</span></li>
</ul>
<h3>Deliver my taco masterpieces to...</h3>
<label for="deliveryName">Name: </label>
<input type="text" th:field="*{deliveryName}"/>
<br/>
<label for="deliveryStreet">Street address: </label>
<input type="text" th:field="*{deliveryStreet}"/>
<br/>
<label for="deliveryCity">City: </label>
<input type="text" th:field="*{deliveryCity}"/>
<br/>
<label for="deliveryState">State: </label>
<input type="text" th:field="*{deliveryState}"/>
<br/>
<label for="deliveryZip">Zip code: </label>
<input type="text" th:field="*{deliveryZip}"/>
<br/>
<h3>Here's how I'll pay...</h3>
<label for="ccNumber">Credit Card #: </label>
<input type="text" th:field="*{ccNumber}"/>
<br/>
<label for="ccExpiration">Expiration: </label>
<input type="text" th:field="*{ccExpiration}"/>
<br/>
<label for="ccCVV">CVV: </label>
<input type="text" th:field="*{ccCVV}"/>
<br/>
<input type="submit" value="Submit Order"/>
</form>
</body>
</html>
```

Na maior parte, a visualização orderForm.html é um conteúdo típico HTML/Thymeleaf, com
muito pouco de nota. Ela começa listando os tacos que foram adicionados ao pedido. Ela usa th:each do Thymeleaf para percorrer a propriedade tacos do pedido conforme cria a lista.
Então, ela renderiza o formulário de pedido.
Mas observe que a tag < form > aqui é diferente da tag < form > usada na listagem 2.5, pois ela também especifica uma ação de formulário. Sem uma ação especificada, o formulário enviaria uma solicitação HTTP POST de volta para a mesma URL que apresentou o formulário.
Mas aqui, você especifica que o formulário deve ser POSTado para /orders (usando o operador @{...} do Thymeleaf para um caminho relativo ao contexto).
Portanto, você precisará adicionar outro método à sua classe OrderController
que lida com solicitações POST para /orders. Você não terá como persistir pedidos
até o próximo capítulo, então você manterá isso simples aqui — algo como o que você vê
na próxima listagem.

Listagem 2.10 - Manipulando um envio de pedido de taco

```java
@PostMapping
public String processOrder(TacoOrder order,
SessionStatus sessionStatus) {
log.info("Order submitted: {}", order);
sessionStatus.setComplete();
return "redirect:/";
}
```

Quando o método processOrder() é chamado para manipular um pedido enviado, ele recebe um objeto TacoOrder cujas propriedades são vinculadas aos campos do formulário enviados. Taco- Order, assim como Taco, é uma classe bastante direta que carrega informações do pedido.
No caso deste método processOrder(), o objeto TacoOrder é simplesmente registrado.
Veremos como persisti-lo em um banco de dados no próximo capítulo. Mas antes que process-Order() seja feito, ele também chama setComplete() no objeto SessionStatus passado como um parâmetro. O objeto TacoOrder foi inicialmente criado e colocado na sessão Quando o método processOrder() é chamado para manipular um pedido enviado, ele recebe um objeto TacoOrder cujas propriedades são vinculadas aos campos do formulário enviados. Taco-Order, assim como Taco, é uma classe bastante direta que carrega informações do pedido.
No caso deste método processOrder(), o objeto TacoOrder é simplesmente registrado.
Veremos como persisti-lo em um banco de dados no próximo capítulo. Mas antes que process-Order() seja feito, ele também chama setComplete() no objeto SessionStatus passado como um parâmetro. O objeto TacoOrder foi inicialmente criado e colocado na sessão quando o usuário criou seu primeiro taco. Ao chamar setComplete(), estamos garantindo que a sessão seja limpa e esteja pronta para um novo pedido na próxima vez que o usuário criar um taco.
Agora que você desenvolveu um OrderController e a visualização do formulário de pedido, você está pronto para experimentá-lo. Abra seu navegador em http://localhost:8080/design, selecione alguns ingredientes para seu taco e clique no botão Enviar seu taco. Você deverá ver um formulário semelhante ao mostrado na figura 2.4.

![[Captura de tela de 2025-03-21 08-40-32.png]]

Preencha alguns campos no formulário e pressione o botão Enviar pedido. Conforme você faz isso, fique de olho nos logs do aplicativo para ver as informações do seu pedido. Quando eu tentei, a entrada do log parecia algo assim (reformatada para caber na largura desta página):

Order submitted: TacoOrder(deliveryName=Craig Walls, deliveryStreet=1234 7th
Street, deliveryCity=Somewhere, deliveryState=Who knows?,
deliveryZip=zipzap, ccNumber=Who can guess?, ccExpiration=Some day,
ccCVV=See-vee-vee, tacos=[Taco(name=Awesome Sauce, ingredients=[
Ingredient(id=FLTO, name=Flour Tortilla, type=WRAP), Ingredient(id=GRBF,
name=Ground Beef, type=PROTEIN), Ingredient(id=CHED, name=Cheddar,
type=CHEESE), Ingredient(id=TMTO, name=Diced Tomatoes, type=VEGGIES),
Ingredient(id=SLSA, name=Salsa, type=SAUCE), Ingredient(id=SRCR,
name=Sour Cream, type=SAUCE)]), Taco(name=Quesoriffic, ingredients=
[Ingredient(id=FLTO, name=Flour Tortilla, type=WRAP), Ingredient(id=CHED,
name=Cheddar, type=CHEESE), Ingredient(id=JACK, name=Monterrey Jack,
type=CHEESE), Ingredient(id=TMTO, name=Diced Tomatoes, type=VEGGIES),
Ingredient(id=SRCR,name=Sour Cream, type=SAUCE)])])

Parece que o método processOrder() fez seu trabalho, manipulando o envio do formulário registrando detalhes sobre o pedido. Mas se você olhar cuidadosamente para a entrada de log do meu pedido de teste, você pode ver que ele deixou um pouco de informação ruim entrar. A maioria dos campos no formulário continha dados que não poderiam estar corretos. Vamos adicionar alguma validação para garantir que os dados fornecidos pelo menos se assemelhem ao tipo de informação necessária.

### 2.3 Validando a entrada do formulário
Ao projetar uma nova criação de taco, e se o usuário não selecionar nenhum ingrediente ou não especificar um nome para sua criação? Ao enviar o pedido, e se o usuário não
preencher os campos de endereço obrigatórios? Ou se ele inserir um valor no campo do cartão de crédito que nem mesmo é um número de cartão de crédito válido?
Do jeito que as coisas estão agora, nada impedirá o usuário de criar um taco sem
nenhum ingrediente ou com um endereço de entrega vazio, ou mesmo enviar a letra de sua
música favorita como o número do cartão de crédito. Isso porque você ainda não especificou como esses campos devem ser validados. Uma maneira de executar a validação de formulário é encher os métodos processTaco() e process-Order() com um monte de blocos if/then, verificando cada campo para garantir que ele atenda às regras de validação apropriadas. Mas isso seria trabalhoso e difícil de ler e depurar.
Felizmente, o Spring suporta a API de validação #JavaBean (também conhecida como JSR 303; https://jcp.org/en/jsr/detail?id=303). Isso facilita a declaração de regras de validação em oposição à escrita explícita da lógica de declaração no código do seu aplicativo.
Para aplicar a validação no Spring MVC, você precisa
- Adicionar o iniciador Spring Validation à compilação.
- Declarar regras de validação na classe que deve ser validada: especificamente, a classe Taco.
- Especifique que a validação deve ser realizada nos métodos do controlador que
requerem validação: especificamente, o método processTaco() do DesignTacoController
e o método processOrder() do OrderController.
- Modifique as visualizações do formulário para exibir erros de validação.
A API de validação oferece várias anotações que podem ser colocadas em propriedades de
objetos de domínio para declarar regras de validação. A implementação da API de validação do Hibernate adiciona ainda mais anotações de validação. Ambas podem ser adicionadas a um projeto adicionando o iniciador #Spring-Validation à compilação. A caixa de seleção Validation em I/O no assistente Spring Boot Starter fará o trabalho, mas se você preferir editar manualmente sua compilação, a seguinte entrada no arquivo pom.xml do Maven resolverá o problema:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

Com o iniciador de validação em vigor, vamos ver como você pode aplicar algumas anotações para validar um Taco ou TacoOrder enviado.

### 2.3.1 Declarando regras de validação

Para a classe Taco, você quer garantir que a propriedade name não esteja vazia ou nula e
que a lista de ingredientes selecionados tenha pelo menos um item. A listagem a seguir mostra uma classe Taco atualizada que usa @NotNull e @Size para declarar essas regras de validação.

Listagem 2.11 - Adicionando validação à classe de domínio Taco

```java
package tacos;
import java.util.List;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;
import lombok.Data;

@Data
public class Taco {

@NotNull
@Size(min=5, message="Name must be at least 5 characters long")
private String name;

@NotNull
@Size(min=1, message="You must choose at least 1 ingredient")
private List<Ingredient> ingredients;
}
```

Você notará que, além de exigir que a propriedade name não seja nula, você declara que ela deve ter um valor com pelo menos cinco caracteres de comprimento.
Quando se trata de declarar validação em pedidos de taco enviados, você deve aplicar
anotações à classe TacoOrder. Para as propriedades address, você quer ter certeza
de que o usuário não deixou nenhum campo em branco. Para isso, você usará a anotação @NotBlank A validação dos campos payment, no entanto, é um pouco mais exótica. Você precisa garantir não apenas que a propriedade ccNumber não esteja vazia, mas também que ela contenha um valor que possa ser um número de cartão de crédito válido. A propriedade ccExpiration deve estar em conformidade com um formato de MM/AA (mês e ano de dois dígitos), e a propriedade ccCVV precisa ser um número de três dígitos. Para atingir esse tipo de validação, você precisa usar algumas outras anotações da API de validação JavaBean e pegar emprestada uma anotação de validação da coleção de anotações do Hibernate Validator. A listagem a seguir mostra as alterações
necessárias para validar a classe TacoOrder.

Listagem 2.12 Validando campos de pedidos

```java
package tacos;
import javax.validation.constraints.Digits;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Pattern;
import org.hibernate.validator.constraints.CreditCardNumber;
import java.util.List;
import java.util.ArrayList;
import lombok.Data;

@Data
public class TacoOrder {
@NotBlank(message="Delivery name is required")
private String deliveryName;
@NotBlank(message="Street is required")
private String deliveryStreet;
@NotBlank(message="City is required")
private String deliveryCity;
@NotBlank(message="State is required")
private String deliveryState;
@NotBlank(message="Zip code is required")
private String deliveryZip;
@CreditCardNumber(message="Not a valid credit card number")
private String ccNumber;
@Pattern(regexp="^(0[1-9]|1[0-2])([\\/])([2-9][0-9])$",
message="Must be formatted MM/YY")
private String ccExpiration;
@Digits(integer=3, fraction=0, message="Invalid CVV")
private String ccCVV;
private List<Taco> tacos = new ArrayList<>();
public void addTaco(Taco taco) {
this.tacos.add(taco);
}
}

```

Como você pode ver, a propriedade ccNumber é anotada com @CreditCardNumber. Esta
anotação declara que o valor da propriedade deve ser um número de cartão de crédito válido que
passe na verificação do algoritmo Luhn (https://creditcardvalidator.org/articles/luhn-
algorithm). 
Isso evita erros do usuário e dados deliberadamente ruins, mas não garante que o número do cartão de crédito seja realmente atribuído a uma conta ou que a conta possa ser usada para cobrança. Infelizmente, não há nenhuma anotação pronta para validar o formato MM/AA da propriedade ccExpiration. Eu apliquei a anotação @Pattern, fornecendo a ela
uma expressão regular que garante que o valor da propriedade esteja de acordo com o formato desejado. Se você está se perguntando como decifrar a expressão regular, eu o encorajo a verificar os muitos guias de expressão regular online, incluindo http://www.regular-expressions.info/.
A sintaxe da expressão regular é uma arte obscura e certamente está fora do
escopo deste livro. Finalmente, anotamos a propriedade ccCVV com @Digits para garantir
que o valor contenha exatamente três dígitos numéricos. Todas as anotações de validação incluem um atributo de mensagem que define a mensagem que você exibirá ao usuário se as informações inseridas não atenderem aos requisitos das regras de validação declaradas.

### 2.3.2 Executando validação na vinculação de formulário

Agora que você declarou como um Taco e um TacoOrder devem ser validados, precisamos
revisitar cada um dos controladores, especificando que a validação deve ser executada quando os formulários forem POSTados para seus respectivos métodos manipuladores.
Para validar um Taco enviado, você precisa adicionar a anotação @Valid da JavaBean Validation API ao argumento Taco do método process-Taco() do DesignTacoController, conforme mostrado a seguir.

Listagem 2.13 - Validando um Taco Postado

```java
import javax.validation.Valid;
import org.springframework.validation.Errors;
...
@PostMapping
public String processTaco(
@Valid Taco taco, Errors errors,
@ModelAttribute TacoOrder tacoOrder) {
if (errors.hasErrors()) {
return "design";
}
tacoOrder.addTaco(taco);
log.info("Processing taco: {}", taco);
return "redirect:/orders/current";
}
```

A anotação @Valid diz ao #Spring-MVC para executar a validação no objeto Taco
enviado após ele ser vinculado aos dados do formulário enviado e antes do método processTaco() ser chamado. Se houver algum erro de validação, os detalhes desses erros serão capturados em um objeto Errors que é passado para processTaco(). As primeiras linhas de processTaco() consultam o objeto Errors, perguntando ao seu método hasErrors() se há algum erro de validação. Se houver, o método conclui sem processar o Taco
e retorna o nome da visualização "design" para que o formulário seja exibido novamente.
Para executar a validação em objetos TacoOrder enviados, alterações semelhantes também são necessárias no método processOrder() de OrderController, conforme mostrado na próxima listagem de código.

Listagem 2.14 - Validando um TacoOrder POSTado

```java
@PostMapping
public String processOrder(@Valid TacoOrder order, Errors errors,
SessionStatus sessionStatus) {
if (errors.hasErrors()) {
return "orderForm";
}
log.info("Order submitted: {}", order);
sessionStatus.setComplete();
return "redirect:/";
}
```

Em ambos os casos, o método poderá processar os dados enviados se não houver
erros de validação. Se houver erros de validação, a solicitação será encaminhada para a
visualização do formulário para dar ao usuário uma chance de corrigir seus erros.
Mas como o usuário saberá quais erros precisam de correção? A menos que você chame
os erros no formulário, o usuário ficará adivinhando sobre como enviar o formulário com sucesso.

### 2.3.3 Exibindo erros de validação
O Thymeleaf oferece acesso conveniente ao objeto Errors por meio da propriedade fields e
com seu atributo th:errors. Por exemplo, para exibir erros de validação no
campo de número do cartão de crédito, você pode adicionar um elemento < span > que usa essas referências de erro para o modelo de formulário de pedido, como a seguir.

Listagem 2.15 - Exibindo erros de validação
```html
<label for="ccNumber">Credit Card #: </label>
<input type="text" th:field="*{ccNumber}"/>
<span class="validationError"
th:if="${#fields.hasErrors('ccNumber')}"
th:errors="*{ccNumber}">CC Num Error</span>
```

Além de um atributo de classe que pode ser usado para estilizar o erro para que ele chame a atenção do usuário, o elemento < span > usa um atributo th:if para decidir se deve exibir o < span >. O método hasErrors() da propriedade fields verifica se há erros no campo ccNumber. Se houver, o < span > será renderizado. O atributo th:errors faz referência ao campo ccNumber e, supondo que existam erros para esse campo, ele substituirá o conteúdo do espaço reservado do elemento < span > pela mensagem de validação. Se você espalhasse tags < span > semelhantes ao redor do formulário de pedido para os outros campos, você poderia ver um formulário parecido com a figura 2.5 ao enviar informações inválidas. Os erros indicam que os campos de nome, cidade e código postal foram deixados em branco e que todos os campos de pagamento não atendem aos critérios de validação. Agora, seus controladores Taco Cloud não apenas exibem e capturam a entrada, mas eles também validam que as informações atendem a algumas regras básicas de validação. Vamos dar um passo para trás e reconsiderar o HomeController do capítulo 1, observando uma implementação
alternativa.

### 2.4 Trabalhando com controladores de visualização

Embora cada controlador sirva a um propósito distinto na funcionalidade do aplicativo, eles
todos eles praticamente aderem ao seguinte modelo de programação:
- Eles são todos anotados com @Controller para indicar que são classes de controlador que devem ser descobertas automaticamente pela varredura de componentes Spring e instanciadas como beans no contexto do aplicativo Spring
- Todos, exceto HomeController, são anotados com @RequestMapping no nível de classe
para definir um padrão de solicitação de linha de base que o controlador manipulará.

![[Captura de tela de 2025-03-21 09-14-34.png]]

- Todos eles têm um ou mais métodos que são anotados com @GetMapping ou
@PostMapping para fornecer detalhes sobre quais métodos devem lidar com quais
tipos de solicitações.
A maioria dos controladores que você escreverá seguirá esse padrão. Mas quando um controlador é simples o suficiente para não preencher um modelo ou entrada de processo — como é o caso com seu HomeController — há outra maneira de definir o controlador. Dê uma olhada na próxima listagem para ver como você pode declarar um controlador de visualização — um controlador que não faz nada além de encaminhar a solicitação para uma visualização.

- Listagem 2.16 -  Declarando um controlador de visualização

```java
package tacos.web;
import org.springframework.context.annotation.Configuration;
import
org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
@Configuration
public class WebConfig implements WebMvcConfigurer {
@Override
public void addViewControllers(ViewControllerRegistry registry) {
registry.addViewController("/").setViewName("home");
}
}
```

A coisa mais significativa a ser notada sobre o WebConfig é que ele implementa a interface Web-MvcConfigurer. O #WebMvcConfigurer define vários métodos para configurar o
Spring MVC. Embora seja uma interface, ele fornece implementações padrão de todos
os métodos, então você precisa substituir apenas os métodos que precisa. Neste caso, você substitui addViewControllers().
O método addViewControllers() recebe um ViewControllerRegistry que você
pode usar para registrar um ou mais controladores de visualização. Aqui, você chama addViewController() no registro, passando “/”, que é o caminho para o qual seu controlador de visualização irá lidar com solicitações GET. Esse método retorna um objeto ViewControllerRegistration, no qual você imediatamente chama setViewName() para especificar home como a visualização para a qual uma solicitação de “/” deve ser encaminhada. E assim, você conseguiu substituir HomeController por algumas linhas em uma classe de configuração. Agora você pode excluir o HomeController, e o aplicativo deve
ainda se comportar como antes. A única outra alteração necessária é revisitar o HomeControllerTest do capítulo 1, removendo a referência ao HomeController da
anotação @WebMvcTest, para que a classe de teste seja compilada sem erros.
Aqui, você criou uma nova classe de configuração WebConfig para abrigar a declaração do controlador de visualização. Mas qualquer classe de configuração pode implementar o WebMvcConfigurer e substituir o método addViewController. Por exemplo, você poderia ter adicionado a mesma declaração do controlador de visualização à classe bootstrap TacoCloudApplication assim:

```java
@SpringBootApplication
public class TacoCloudApplication implements WebMvcConfigurer {
public static void main(String[] args) {
SpringApplication.run(TacoCloudApplication.class, args);
}
@Override
public void addViewControllers(ViewControllerRegistry registry) {
registry.addViewController("/").setViewName("home");
}
}
```

Ao estender uma classe de configuração existente, você pode evitar criar uma nova classe de configuração, mantendo a contagem de artefatos do seu projeto baixa. Mas eu prefiro criar uma nova classe de configuração para cada tipo de configuração (web, dados, segurança e assim por diante),
mantendo a configuração de bootstrap do aplicativo limpa e simples.
Falando em controladores de visualização — e mais genericamente, as visualizações para as quais os controladores encaminham solicitações — até agora você tem usado o Thymeleaf para todas as suas visualizações. Eu gosto
muito do Thymeleaf, mas talvez você prefira um modelo de template diferente para as visualizações do seu aplicativo.
Vamos dar uma olhada nas muitas opções de visualização suportadas pelo Spring.
Ao estender uma classe de configuração existente, você pode evitar criar uma nova classe de configuração, mantendo a contagem de artefatos do seu projeto baixa. Mas eu prefiro criar uma nova classe de configuração para cada tipo de configuração (web, dados, segurança e assim por diante), mantendo a configuração de bootstrap do aplicativo limpa e simples. Falando em controladores de visualização — e mais genericamente, as visualizações para as quais os controladores encaminham solicitações — até agora você tem usado o Thymeleaf para todas as suas visualizações. Eu gosto muito do Thymeleaf, mas talvez você prefira um modelo de template diferente para as visualizações do seu aplicativo.
Vamos dar uma olhada nas muitas opções de visualização suportadas pelo Spring.

### 2.5 Escolhendo uma biblioteca de modelos de visualização

Na maior parte, sua escolha de uma biblioteca de modelos de visualização é uma questão de gosto pessoal. O Spring é flexível e suporta muitas opções comuns de modelos. Com apenas algumas pequenas exceções, a biblioteca de modelos que você escolher não terá ideia de que está funcionando com o Spring.3
A Tabela 2.2 cataloga as opções de modelos suportadas pela autoconfiguração do Spring Boot.


| Template               | Spring Boot starter dependency       |
| ---------------------- | ------------------------------------ |
| FreeMarker             | spring-boot-starter-freemarker       |
| Groovy templates       | spring-boot-starter-groovy-templates |
| JavaServer Pages (JSP) | None (provided by Tomcat or Jetty)   |
| Mustache               | spring-boot-starter-mustache         |
| Thymeleaf              | spring-boot-starter-thymeleaf        |
Em termos gerais, você seleciona a biblioteca de modelos de visualização que deseja, adiciona-a como uma dependência em sua compilação e começa a escrever modelos no diretório /templates (sob o diretório src/main/resources em um projeto Maven ou Gradle).
O Spring Boot detecta sua biblioteca de modelos escolhida e configura automaticamente os componentes necessários para que ela sirva visualizações para seus controladores Spring MVC.
Você já fez isso com o Thymeleaf para o aplicativo Taco Cloud. No capítulo 1, você selecionou a caixa de seleção Thymeleaf ao inicializar o projeto. Isso
resultou na inclusão do iniciador Thymeleaf do Spring Boot no arquivo pom.xml. Quando o aplicativo é inicializado, a autoconfiguração do Spring Boot detecta a presença do Thymeleaf e configura automaticamente os beans Thymeleaf para você. Tudo o que você tinha que fazer era começar a escrever modelos em /templates.
Se você preferir usar uma biblioteca de modelos diferente, basta selecioná-la na inicialização do projeto ou editar sua compilação de projeto existente para incluir a biblioteca de modelos recém-escolhida.
Por exemplo, digamos que você queira usar Mustache em vez de Thymeleaf. Sem problemas. Basta visitar o arquivo pom.xml do projeto e substituir este

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

por isso:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-mustache</artifactId>
</dependency>
```

Claro, você precisaria ter certeza de escrever todos os modelos com a sintaxe Mustache em vez das tags Thymeleaf. As especificidades de trabalhar com Mustache (ou qualquer uma das escolhas de linguagem de modelo) estão bem fora do escopo deste livro, mas para lhe dar umaideia do que esperar, aqui está um trecho de um modelo Mustache que renderizará um dos grupos de ingredientes no formulário de design de taco:

```html
<h3>Designate your wrap:</h3>
{{#wrap}}
<div>
<input name="ingredients" type="checkbox" value="{{id}}" />
<span>{{name}}</span><br/>
</div>
{{/wrap}}
```

Este é o equivalente do Mustache do snippet Thymeleaf na seção 2.1.3. O bloco
{{#wrap}} (que conclui com {{/wrap}}) itera por uma coleção no atributo de solicitação cuja chave é wrap e renderiza o HTML incorporado para cada item. As tags {{id}} e {{name}} fazem referência às propriedades id e name do item (que deve ser um Ingredient).
Você notará na tabela 2.2 que o JSP não requer nenhuma dependência especial na
construção. Isso ocorre porque o próprio contêiner de servlet (Tomcat por padrão) implementa a especificação JSP, não exigindo mais dependências. Mas há um porém se você escolher usar JSP. Acontece que os contêineres de servlet Java — incluindo contêineres Tomcat e Jetty incorporados — geralmente procuram por JSPs em algum lugar em /WEB-INF. Mas se você estiver construindo seu aplicativo como um arquivo JAR executável, não há como satisfazer esse requisito. Portanto, JSP é uma opção somente se
você estiver construindo seu aplicativo como um arquivo WAR e implantando-o em um contêiner de servlet tradicional. Se estiver construindo um arquivo JAR executável, você deve escolher Thymeleaf, Free-Marker ou uma das outras opções na tabela 2.2.

### 2.5.1 Cache de modelos

Por padrão, os modelos são analisados ​​apenas uma vez — quando são usados ​​pela primeira vez — e os resultados desse análise são armazenados em cache para uso subsequente. Esse é um ótimo recurso para produção, porque evita a análise redundante de modelos em cada solicitação e, portanto, melhora o desempenho.
No entanto, esse recurso não é tão incrível no momento do desenvolvimento. Digamos que você inicie seu aplicativo, acesse a página de design do taco e decida fazer algumas alterações nele. Ao atualizar seu navegador, você ainda verá a versão original.
A única maneira de ver suas alterações é reiniciar o aplicativo, o que é bastante
inconveniente. Felizmente, temos uma maneira de desabilitar o cache. Tudo o que precisamos fazer é definir uma propriedade de cache apropriada para o modelo como falsa. A Tabela 2.3 lista as propriedades de cache para cada das bibliotecas de modelos suportadas.


| Template         | Cache-enable property        |
| ---------------- | ---------------------------- |
| FreeMarker       | spring.freemarker.cache      |
| Groovy templates | spring.groovy.template.cache |
| Mustache         | spring.mustache.cache        |
| Thymeleaf        | spring.thymeleaf.cache       |
Por padrão, todas essas propriedades são definidas como true para habilitar o cache. Você pode desabilitar o cache para o mecanismo de modelo escolhido configurando sua propriedade cache como false. Por exemplo, para desabilitar o cache do Thymeleaf, adicione a seguinte linha em application.properties:
spring.thymeleaf.cache=false

O único problema é que você vai querer ter certeza de remover essa linha (ou defini-la como true) antes de implantar seu aplicativo para produção. Uma opção é definir a propriedade em um perfil. (Falaremos sobre perfis no capítulo 6.)
Uma opção muito mais simples é usar o DevTools do Spring Boot, como optamos por fazer no capítulo 1. Entre os muitos bits úteis de ajuda em tempo de desenvolvimento oferecidos pelo DevTools, ele desabilitará o cache para todas as bibliotecas de modelo, mas se desabilitará (e, portanto, reativará o cache de modelo) quando seu aplicativo for implantado.

### Sumário

- Spring oferece um framework web poderoso chamado Spring MVC que pode ser usado para desenvolver o frontend web para um aplicativo Spring.
- Spring MVC é baseado em anotações, permitindo a declaração de métodos de manipulação de solicitações com anotações como @RequestMapping, @GetMapping e @Post-Mapping.
- A maioria dos métodos de manipulação de solicitações conclui retornando o nome lógico de uma visão, como um modelo Thymeleaf, para o qual a solicitação (junto com quaisquer dados de modelo) é encaminhada.
- Spring MVC suporta validação por meio da API de validação JavaBean e implementações da API de validação, como o Hibernate Validator.
- Controladores de visualização podem ser registrados com addViewController em uma classe WebMvc-Configurer para manipular solicitações HTTP GET para as quais nenhum dado de modelo ou processamento é necessário.
- Além do Thymeleaf, Spring suporta uma variedade de opções de visualização, incluindo FreeMarker, modelos Groovy e Mustache.