
Este Capítulo abrange:
	Fundamentos do Spring e do Spring Boot
	Inicialização de um projeto Spring
	Uma visão geral do cenário Spring

Embora o filósofo grego Heráclito não fosse muito conhecido como desenvolvedor de software, ele parece ter tido um bom domínio do assunto. Ele foi citado como dizendo, "==A única constante é a mudança"==. Essa declaração captura uma verdade fundamental do desenvolvimento de software.
A maneira como desenvolvemos aplicativos hoje é diferente do que era há um ano, 5 anos atrás, 10 anos atrás e certamente 20 anos atrás, antes de uma forma inicial do Spring Framework ser introduzida no livro de Rod Johnson, Expert One-on-One J2EE Design
and Development.
Naquela época, os tipos mais comuns de aplicativos desenvolvidos eram aplicativos da web baseados em navegador, apoiados por bancos de dados relacionais. Embora esse tipo de desenvolvimento ainda seja relevante - e o #Spring esteja bem equipado para esses tipos de aplicativos - estamos agora também interessados em desenvolver aplicativos compostos de #microsserviços destinados à nuvem que persistem dados em uma variedade de bancos de dados. E um novo interesse em programação reativa que visa fornecer maior escalabilidade e desempenho aprimorado com operações não bloqueantes.
À medida que o desenvolvimento de software evolui, o Spring Framework também mudou para abordar preocupações de desenvolvimento moderno, incluindo microsserviços e programação reativa.
Os criadores do Spring também se propuseram a simplificar seu modelo de desenvolvimento introduzindo o Spring Boot. Esteja você desenvolvendo um aplicativo web simples com suporte de banco de dados ou construindo um aplicativo moderno construído em torno de microsserviços, o Spring é o framework que ajudará você a atingir seus objetivos. Este capítulo é seu primeiro passo em uma jornada pelo desenvolvimento de aplicativos modernos com o Spring.

## 1.1 O que é Spring ? 

Eu sei que você provavelmente está ansioso para começar a escrever um aplicativo Spring, e eu lhe asseguro que antes que este capítulo termine, você terá desenvolvido um simples. Mas primeiro, deixe-me preparar o cenário com alguns conceitos básicos do Spring que ajudarão você a entender o que faz o Spring funcionar.
Qualquer aplicativo não trivial compreende muitos componentes, cada um responsável por sua própria parte da funcionalidade geral do aplicativo, coordenando-se com os outros elementos do aplicativo para fazer o trabalho.
Quando o aplicativo é executado, esses componentes de alguma forma precisam ser criados e apresentados uns aos outros.
Em seu núcleo, o Spring oferece um contêiner, frequentemente chamado de #contexto do aplicativo Spring, que cria e gerencia componentes do aplicativo. Esses componentes, ou #beans, são conectados dentro do contexto do aplicativo Spring para fazer um aplicativo completo, assim como tijolos, argamassa, madeira, pregos, encanamento e fiação são unidos para fazer uma casa.
O ato de conectar beans é baseado em um padrão conhecido como #injeção de dependência (DI).
Em vez de ter componentes criando e mantendo o ciclo de vida de outros beans
dos quais eles dependem, um aplicativo com injeção de dependência depende de uma entidade separada (o contêiner) para criar e manter todos os componentes e injetá-los nos beans que precisam deles.
Isso é feito normalmente por meio de argumentos construtores ou métodos acessadores de propriedade.
Por exemplo, suponha que entre os muitos componentes de um aplicativo, você irá
abordar dois:
Um serviço de inventário (para buscar níveis de inventário) e um serviço de produto (para fornecer informações básicas do produto). O serviço de produto depende do serviço de inventário para poder fornecer um conjunto de informações sobre produtos. 
A figura 1.1 ilustra os relacionamentos entre esses #beans e o contexto do aplicativo Spring.
Além de seu contêiner principal, o Spring e um portfólio completo de bibliotecas relacionadas oferecem uma estrutura web, uma variedade de opções de persistência de dados, uma estrutura de segurança, integração com outros sistemas, monitoramento de tempo de execução, suporte a microsserviços, um modelo de programação reativa e muitos outros recursos necessários para o desenvolvimento de aplicativos modernos. Historicamente, a maneira como você guiaria o contexto do Spring para conectar os beans era com um ou mais arquivos XML que descreviam os componentes e seu relacionamento com outros componentes.

![[figure1.1.png]]

> Figura 1.1 Os componentes do aplicativo são gerenciados e injetados uns nos outros pelo contexto do aplicativo Spring


Por exemplo, o seguinte código XML declara dois beans, um bean InvetoryService e um bean ProductService, e conecta o bean InventoryService ao Product-Service por meio de um argumento construtor:

```java
<bean id="inventoryService"
class="com.example.InventoryService" />

<bean id="productService"
class="com.example.ProductService" />
<constructor-arg ref="inventoryService" />
</bean>
```

Em versões recentes do Spring, no entanto, uma configuração baseada em Java é mais comum.
A seguinte classe de configuração baseada em Java é equivalente á configuração XML:
```java
@Configuration
public class ServiceConfiguration {
@Bean
public InventoryService inventoryService() {
return new InventoryService();
}

@Bean
public ProductService productService() {
return new ProductService(inventoryService());
}

}
```

A anotação @Configuration indica ao Spring que esta é uma classe de configuração que fornecerá beans para o contexto do aplicativo Spring.
Os métodos de configuração são anotados com @Bean, indicando que os objetos que eles retornam devem ser adicionados como beans no contexto do aplicativo (onde, por padrão, seus respectivos IDs de bean serão os mesmo que os nomes dos métodos que os definem).
A configuração baseada em Java oferece vários benefícios sobre a configuração baseada em XML, incluindo maior segurança de tipo e refatorabilidade aprimorada. Mesmo assim, a configuração explícita com Java ou XML é necessária apenas se o Spring não puder configurar os componentes automaticamente.
A configuração automática tem suas raízes nas técnicas do Spring conhecidas como autowiring e varredura de componentes. Com a varredura de componentes, o Spring pode descobrir automaticamente componentes do classpath de um aplicativo e criá-los como beans no contexto do aplicativo Spring.
Com a autowiring, o Spring injeta automaticamente os componentes com os outros beans dos quais eles dependem. Mais recentemente, com a introdução do Spring Boot, a configuração automática foi muito além da varredura de componentes e da fiação automática. O Spring Boot é uma extensão do Spring Framework que oferece vários aprimoramentos de produtividade. O mais conhecido desses aprimoramentos é a autoconfiguração, onde o Spring Boot pode fazer suposições razoáveis ​​sobre quais componentes precisam ser configurados e conectados, com base em entradas no classpath, variáveis ​​de ambiente e outros fatores.
A autoconfiguração é muito parecida com o vento — você pode ver os efeitos dela, mas
não há código que eu possa mostrar a vocês e dizer "Olha! Aqui está um exemplo de autoconfiguração!" Coisas acontecem, componentes são habilitados e a funcionalidade é fornecida sem escrever código. É essa falta de código que é essencial para a autoconfiguração e o que a torna tão maravilhosa.
A autoconfiguração do Spring Boot reduziu drasticamente a quantidade de configuração explícita (seja com XML ou Java) necessária para construir um aplicativo. Na verdade,
quando você terminar o exemplo neste capítulo, você terá um aplicativo Spring funcionando que tem apenas uma única linha de código de configuração Spring! O Spring Boot aprimora tanto o desenvolvimento Spring que é difícil imaginar o desenvolvimento de aplicativos Spring sem ele. 
Por esse motivo, este livro trata Spring e Spring Boot como se fossem um e o mesmo. Usaremos o Spring Boot o máximo possível e a configuração explícita somente quando necessário. E, como a configuração XML do Spring é a maneira tradicional de trabalhar com o Spring, focaremos principalmente na configuração baseada em Java do Spring.
Mas chega de conversa fiada, conversa fiada e balela. O título deste livro inclui
a frase em ação, então vamos começar, para que você possa começar a escrever seu primeiro aplicativo com o Spring.

## 1.2 Inicializando um aplicativo Spring

Ao longo deste livro, você criará o Taco Cloud, um aplicativo online para pedir a comida mais maravilhosa criada pelo homem — tacos. Claro, você usará Spring, Spring Boot e uma variedade de bibliotecas e frameworks relacionados para atingir esse objetivo.
Você encontrará várias opções para inicializar um aplicativo Spring. Embora eu pudesse
orientá-lo nas etapas de criação manual de uma estrutura de diretório de projeto e definição de uma especificação de construção, isso é tempo perdido — tempo melhor gasto escrevendo código de aplicativo.
Portanto, você vai se apoiar no Spring Initializr para inicializar seu aplicativo.
O Spring Initializr é um aplicativo da web baseado em navegador e uma API REST,
que pode produzir uma estrutura de projeto Spring de esqueleto que você pode desenvolver com qualquer funcionalidade que desejar.
Seguem várias maneiras de usar o Spring Initializr:
- Do aplicativo da web: https://start.spring.io/
- Da linha de comando usando o comando curl
- Da linha de comando usando a interface de linha de comando do Spring Boot
- Ao criar um novo projeto com o Spring Tool Suite
- Ao criar um novo projeto com o IntelliJ IDEA
- Ao criar um novo projeto com o Apache NetBeans
Em vez de gastar várias páginas deste capítulo falando sobre cada uma dessas
opções, coletei esses detalhes no apêndice.
Neste capítulo, e ao longo deste livro, mostrarei como criar um novo projeto usando minha opção favorita: Suporte ao Spring Initializr no Spring Tool Suite.
Como o nome sugere, o Spring Tool Suite é um fantástico ambiente de desenvolvimento Spring que vem na forma de extensões para Eclipse, VS Code ou o Theia IDE. Você pode baixar binários prontos para execução do Spring Tool Suite em https://spring.io/tools.
O Spring Tool Suite oferece um recurso prático do Spring Boot Dashboard que
facilita iniciar, reiniciar e parar aplicativos Spring Boot do IDE.
Se você não é um usuário do Spring Tool Suite, tudo bem; ainda podemos ser amigos. Pule
para o apêndice e substitua a opção Initializr que melhor lhe convier pelas instruções nas seções a seguir. Mas saiba que ao longo deste livro, eu posso ocasionalmente fazer referência a recursos específicos do Spring Tool Suite, como o Spring Boot Dashboard. Se você não estiver usando o Spring Tool Suite, precisará adaptar essas instruções para se adequarem ao seu IDE.

### 1.2.1 Inicializando um projeto Spring com Spring Tool Suite
Para começar com um novo projeto Spring no Spring Tool Suite, vá ao menu Arquivo e
selecione Novo, e então
![[figure1.2.1.png]]

  
Depois de selecionar Spring Starter Project, uma nova caixa de diálogo do assistente de projeto (figura 1.3) aparece. A primeira página do assistente solicita algumas informações gerais do projeto, como o nome do projeto, descrição e outras informações essenciais. Se você estiver familiarizado com o conteúdo de um arquivo Maven pom.xml, reconhecerá a maioria dos campos como itens que terminam em uma especificação de build do Maven. Para o aplicativo Taco Cloud, preencha a caixa de diálogo conforme mostrado na figura 1.3 e clique em Avançar.
![[figure1.3.png]]

A próxima página do assistente permite que você selecione dependências para adicionar ao seu projeto (veja a figura 1.4). Observe que próximo ao topo da caixa de diálogo, você pode selecionar em qual versão do Spring Boot você quer basear seu projeto. O padrão é a versão mais atual disponível. Geralmente é uma boa ideia deixar como está, a menos que você precise direcionar uma versão diferente.
![[figure1.4.png]]

Quanto às dependências em si, você pode expandir as várias seções e
procurar as dependências desejadas manualmente ou procurá-las na caixa de pesquisa
no topo da lista Disponível. Para o aplicativo Taco Cloud, você começará com as
dependências mostradas na figura 1.4.
Neste ponto, você pode clicar em Concluir para gerar o projeto e adicioná-lo ao seu espaço de trabalho. Mas se você estiver se sentindo um pouco aventureiro, clique em Avançar mais uma vez para ver a página final do novo assistente de projeto inicial, conforme mostrado na figura 1.5.
![[figure1.5.png]]

Por padrão, o novo assistente de projeto faz uma chamada para o Spring Initializr em http://start.spring.io para gerar o projeto.
Geralmente, não há necessidade de substituir esse padrão, é por isso que você poderia ter clicado em Concluir na segunda página do assistente. Mas se por algum motivo você estiver hospedando seu próprio clone do Initializr (talvez uma cópia local em sua própria máquina ou um clone personalizado em execução dentro do firewall da sua empresa), então
você vai querer alterar o campo Base Url para apontar para sua instância do Initializr antes
de clicar em Concluir.
Depois de clicar em Concluir, o projeto é baixado do Initializr e carregado em seu espaço de trabalho. Aguarde alguns instantes para que ele carregue e construa, e então você estará pronto para começar a desenvolver a funcionalidade do aplicativo. Mas primeiro, vamos dar uma olhada no queo Initializr lhe deu.

### 1.2.2 Examinando a estrutura do projeto Spring

Após o projeto carregar no IDE, expanda-o para ver o que ele contém. A Figura 1.6 mostra
o projeto Taco Cloud expandido no Spring Tool Suite.

![[figure1.6.png]]

Você pode reconhecer isso como uma estrutura típica de projeto Maven ou Gradle, onde o código-fonte do aplicativo é colocado em src/main/java, o código de teste é colocado em src/test/java e os recursos não Java são colocados em src/main/resources.
Dentro dessa estrutura de projeto, você vai querer anotar os seguintes itens:

- mvnw e mvnw.cmd — Esses são scripts wrapper do Maven. Você pode usar esses scripts
para construir seu projeto, mesmo que não tenha o Maven instalado em sua máquina.
- pom.xml - Esta é a especificação de build do Maven. Vamos dar uma olhada mais aprofundada nisso em um momento.
- TacoCloudApplication.java — Esta é a classe principal do Spring Boot que inicializa o projeto. Daremos uma olhada mais de perto nesta classe em um momento.
- application.properties — Este arquivo está inicialmente vazio, mas oferece um lugar onde você pode especificar propriedades de configuração.
Vamos mexer um pouco neste arquivo neste
capítulo, mas vou adiar uma explicação detalhada das propriedades de configuração
para o capítulo 6.
- static - Esta pasta é onde você pode colocar qualquer conteúdo estático (imagens, folhas de estilo, JavaScript e assim por diante) que você deseja servir ao navegador. Inicialmente, ela está vazia.
- templates — Esta pasta é onde você colocará os arquivos de modelo que serão usados ​​para renderizar conteúdo para o navegador. Inicialmente, ela está vazia, mas você adicionará um modelo Thymeleaf em breve.
- TacoCloudApplicationTests.java — Esta é uma classe de teste simples que garante que
o contexto do aplicativo Spring seja carregado com sucesso.
Você adicionará mais testes à mistura conforme desenvolve o aplicativo.
À medida que o aplicativo Taco Cloud cresce, você preencherá essa estrutura básica do projeto com código Java, imagens, folhas de estilo, testes e outros materiais colaterais que tornarão seu projeto mais completo. Mas, enquanto isso, vamos nos aprofundar um pouco mais em alguns dos itens que o Spring Initializr forneceu.

### EXPLORANDO A ESPECIFICAÇÃO DE CONSTRUÇÃO
Quando você preencheu o formulário Initializr, especificou que seu projeto deveria ser construído com Maven. Portanto, o Spring Initializr forneceu a você um arquivo pom.xml já preenchido com as escolhas que você fez. A listagem a seguir mostra o arquivo pom.xml inteiro fornecido pelo Initializr.
![[figure1.7.png]]![[figure1.8.png]]
![[figure1.9.png]]

A primeira coisa a ser notada é o elemento < parent > e, mais especificamente, seu
< version > filho. Isso especifica que seu projeto tem spring-boot-starter-parent
como seu POM pai. Entre outras coisas, esse POM pai fornece gerenciamento de dependências para várias bibliotecas comumente usadas em projetos Spring. Para essas
bibliotecas cobertas pelo POM pai, você não precisará especificar uma versão, porque ela é
herdada do pai. A versão, 2.5.6, indica que você está usando o Spring Boot 2.5.6 e, portanto, herdará o gerenciamento de dependências conforme definido por essa versão do Spring Boot.
Entre outras coisas, o gerenciamento de dependências do Spring Boot para a versão 2.5.6 especifica que a versão subjacente do Spring Framework principal será 5.3.12.
Ao falarmos de dependências, observe que há quatro dependências declaradas no elemento < dependencies >. As três primeiras devem parecer um pouco familiares para você. Elas correspondem diretamente às dependências #Spring Web, #Thymeleaf e
Spring Boot DevTools que você selecionou antes de clicar no botão Finish no assistente de novo projeto do Spring Tool Suite. A outra dependência é uma que
fornece muitos recursos de teste úteis. Você não precisou marcar uma caixa para que ela fosse incluída porque o Spring Initializr assume (espero que corretamente) que você estará
escrevendo testes.
Você também pode notar que todas as dependências, exceto a dependência #DevTools
têm a palavra starter em seu ID de artefato. As dependências iniciais do Spring Boot são especiais porque normalmente não têm nenhum código de biblioteca, mas, em vez disso, trazem outras bibliotecas de forma transitória. Essas dependências iniciais oferecem os seguintes benefícios principais:

- Seu arquivo de construção será significativamente menor e mais fácil de gerenciar porque você não precisará declarar uma dependência em cada biblioteca que você pode precisar.
- Você é capaz de pensar em suas dependências em termos de quais recursos elas
fornecem, em vez de seus nomes de biblioteca. Se você estiver desenvolvendo um aplicativo da web,
você adicionará a dependência do web starter em vez de uma longa lista de bibliotecas
individuais que permitem que você escreva um aplicativo da web.
- Você está livre do fardo de se preocupar com versões de biblioteca. Você pode confiar
que as versões das bibliotecas trazidas transitivamente serão compatíveis para uma determinada versão do Spring Boot. Você precisa se preocupar apenas com qual versão do
Spring Boot você está usando.
Finalmente, a especificação de build termina com o plugin Spring Boot. Este plugin executa
algumas funções importantes, descritas a seguir:
- Ele fornece uma meta Maven que permite que você execute o aplicativo usando o Maven.
- Ele garante que todas as bibliotecas de dependência sejam incluídas no arquivo JAR executável e disponíveis no classpath do tempo de execução.
- Ele produz um arquivo manifesto no arquivo JAR que denota a classe bootstrap
(TacoCloudApplication, no seu caso) como a classe principal para o JAR executável.
Falando da classe bootstrap, vamos abri-la e dar uma olhada mais de perto.

### BOOTSTRAPPING DO APLICATIVO
Como você executará o aplicativo a partir de um JAR executável, é importante
ter uma classe principal que será executada quando esse arquivo JAR for executado. Você também precisará de pelo menos uma quantidade mínima de configuração Spring para inicializar o aplicativo. Isso é o que você encontrará na classe TacoCloudApplication, mostrada na listagem a seguir.
![[figura1.10.png]]

Embora haja pouco código em TacoCloudApplication, o que há lá tem um
impacto e tanto. Uma das linhas de código mais poderosas também é uma das mais curtas. A anotação @SpringBootApplication significa claramente que este é um aplicativo Spring Boot. Mas há mais em @SpringBootApplication do que aparenta.
@SpringBootApplication é uma anotação composta que combina as seguintes
três anotações:

- @SpringBootConfiguration — Designa esta classe como uma classe de configuração.
Embora ainda não haja muita configuração na classe, você pode adicionar a configuração do Spring Framework baseada em Java a esta classe se precisar. Esta anotação é, na verdade, uma forma especializada da anotação @Configuration.

- @EnableAutoConfiguration — Habilita a configuração automática do Spring Boot.
Falaremos mais sobre autoconfiguração mais tarde. Por enquanto, saiba que esta anotação diz ao Spring Boot para configurar automaticamente quaisquer componentes que ele acha que você precisará.

- @ComponentScan — Habilita a varredura de componentes. Isso permite que você declare outras classes com anotações como @Component, @Controller e @Service para que o Spring as descubra e registre automaticamente como componentes no contexto do aplicativo Spring.

A outra parte importante do TacoCloudApplication é o método main(). Este é o método que será executado quando o arquivo JAR for executado. Na maior parte, este método é um código boilerplate; cada aplicativo Spring Boot que você escrever terá um método similar ou idêntico a este (não obstante as diferenças de nome de classe).
O método main() chama um método run() estático na classe SpringApplication, que executa o bootstrapping real do aplicativo, criando o contexto do aplicativo Spring. 
Os dois parâmetros passados ​​para o método run() são uma classe de configuração e os argumentos da linha de comando. Embora não seja necessário que a classe de configuração passada para run() seja a mesma da classe bootstrap, esta é a escolha mais conveniente e típica.
Provavelmente você não precisará alterar nada na classe bootstrap. Para aplicativos
simples, você pode achar conveniente configurar um ou dois outros componentes
na classe bootstrap, mas para a maioria dos aplicativos, é melhor criar uma classe de
configuração separada para qualquer coisa que não seja autoconfigurada. Você definirá várias classes de configuração ao longo deste livro, então fique atento aos detalhes.

### TESTING THE APPLICATION
O teste é uma parte importante do desenvolvimento de software. Você sempre pode testar seu projeto manualmente, construindo-o e então executando-o a partir da linha de comando como esta:

```java
$ ./mvnw package
...
$ java -jar target/taco-cloud-0.0.1-SNAPSHOT.jar
```

Ou, porque estamos usando Spring Boot, o plugin Spring Boot Maven torna isso ainda mais fácil, como mostrado a seguir:

```java
$ ./mvnw spring-boot:run
```

Mas o teste manual implica que há um humano envolvido e, portanto, potencial para erro humano e testes inconsistentes. Testes automatizados são mais consistentes e repetíveis.
![[figure1.11.png]]
![[figure1.12.png]]

Não há muito para ser visto em TacoCloudApplicationTests: o único método de teste na
classe está vazio. Mesmo assim, esta classe de teste realiza uma verificação essencial para garantir que o contexto do aplicativo Spring possa ser carregado com sucesso. Se você fizer alguma alteração que impeça a criação do contexto do aplicativo Spring, este teste falhará, e você pode reagir corrigindo o problema. A anotação @SpringBootTest diz ao JUnit para inicializar o teste com os recursos do Spring Boot. Assim como @SpringBootApplication, @SpringBootTest é uma anotação composta, que é anotada com @ExtendWith(SpringExtension.class), para adicionar recursos de teste do Spring ao JUnit 5. Por enquanto, porém, basta pensar nisso como o equivalente da classe de teste de chamar SpringApplication.run() em um método main().
Ao longo deste livro, você verá @SpringBootTest várias vezes, e nós
descobriremos um pouco de seu poder.
Finalmente, há o método de teste em si. Embora @SpringBootTest tenha a tarefa de
carregar o contexto do aplicativo Spring para o teste, ele não terá nada a ver se
não houver métodos de teste. Mesmo sem nenhuma asserção ou código de qualquer tipo, este método de teste vazio solicitará que as duas anotações façam seu trabalho e carreguem o contexto do aplicativo Spring. Se houver algum problema ao fazer isso, o teste falhará.
Para executar esta e quaisquer classes de teste da linha de comando, você pode usar o seguinte encantamento Maven:

```java
$ ./mvnw test
```


Neste ponto, concluímos nossa revisão do código fornecido pelo Spring Initializr. Você viu parte da base boilerplate que pode usar para desenvolver um aplicativo Spring, mas ainda não escreveu uma única linha de código. Agora é hora de iniciar seu IDE, tirar a poeira do seu teclado e adicionar algum código personalizado ao aplicativo Taco Cloud.


### 1.3 Escrevendo um aplicativo Spring
Como você está apenas começando, começaremos com uma alteração relativamente pequena no aplicativo Taco Cloud, mas que demonstrará muito da qualidade do Spring.
Parece apropriado que, como você está apenas começando, o primeiro recurso que você adicionará ao aplicativo Taco Cloud seja uma página inicial. Ao adicionar a página inicial, você criará os dois artefatos de código a seguir:
- Uma classe controladora que manipula solicitações para a página inicial
- Um modelo de visualização que define a aparência da página inicial
E como os testes são importantes, você também escreverá uma classe de teste simples para testar a página inicial. Mas primeiro a coisa mais importante... Vamos escrever esse controlador.

### 1.3.1 Lidando com solicitações da web
O Spring vem com uma estrutura web poderosa conhecida como Spring MVC. No centro do
Spring MVC está o conceito de um controlador, uma classe que manipula solicitações e responde com informações de algum tipo. No caso de um aplicativo voltado para o navegador, um controlador responde opcionalmente preenchendo dados do modelo e passando a solicitação para uma visualização para produzir HTML que é retornado ao navegador.
Uma classe controladora simples que manipula solicitações para o caminho raiz (por exemplo, /) e encaminha essas solicitações para a visualização da página inicial sem preencher nenhum dado do modelo. A listagem a seguir mostra a classe controladora
simples.
![[figure1.13.png]]


Como você pode ver, esta classe é anotada com @Controller. Por si só, @Controller não faz muita coisa. Seu propósito principal é identificar esta classe como um componente para escaneamento de componentes. Como HomeController é anotado com @Controller, o escaneamento de componentes do Spring o descobre automaticamente e cria uma instância de HomeController como um bean no contexto do aplicativo Spring.
Na verdade, um punhado de outras anotações (incluindo @Component, @Service e @Repository) atendem a um propósito semelhante ao @Controller. Você poderia ter anotado HomeController com qualquer uma desses anotações, e ele ainda teria funcionado da mesma forma. A escolha de @Controller é, no entanto, mais descritiva da função deste componente no aplicativo.
O método home() é tão simples quanto os métodos do controlador.
Ele é anotado com @GetMapping para indicar que se uma solicitação HTTP GET for recebida para o caminho raiz /, então esse método deve lidar com essa solicitação. Ele faz isso fazendo nada mais do que retornar um valor String de home. Esse valor é interpretado como o nome lógico de uma visualização. Como essa visualização é implementada depende de alguns fatores, mas como o Thymeleaf está no seu classpath, você pode definir esse modelo com o Thymeleaf.
O nome do modelo é derivado do nome da visualização lógica prefixando-o com /templates/ e post fixando-o com .html. O caminho resultante para o modelo é /templates/home.html. Portanto, você precisará colocar o modelo em seu projeto em /src/main/resources/templates/home.html. Vamos criar esse modelo agora.

### 1.3.2 Definindo a view
No interesse de manter sua home page simples, ela não deve fazer nada mais do que
dar boas-vindas aos usuários no site. A próxima listagem mostra o modelo básico do Thymeleaf que define a home page do Taco Cloud.
![[figure1.14.png]]

Não há muito o que discutir em relação a este modelo. A única linha de código notável é aquela com a tag <img> para exibir o logotipo do Taco Cloud. Ele usa um atributo Thymeleaf th:src e uma expressão @{...} para referenciar a imagem com um caminho relativo ao contexto. Além disso, não é muito mais do que uma página Hello World.
Vamos falar um pouco mais sobre essa imagem. Vou deixar para você definir um logotipo do Taco Cloud que você goste. Mas você precisará ter certeza de colocá-lo no lugar certo dentro do projeto. A imagem é referenciada com o caminho relativo ao contexto /images/TacoCloud.png.
Como você deve se lembrar da nossa revisão da estrutura do projeto, o conteúdo estático, como imagens, é mantido na pasta /src/main/resources/static. Isso significa que a imagem do logotipo do Taco Cloud também deve residir no projeto em /src/main/resources/static/
images/TacoCloud.png.
Agora que você tem um controlador para lidar com solicitações para a página inicial e um modelo de visualização para renderizar a página inicial, você está quase pronto para iniciar o aplicativo e vê-lo em ação. Mas primeiro, vamos ver como você pode escrever um teste em relação ao controlador.

### 1.3.3 Testando o controlador
Testar aplicativos da web pode ser complicado ao fazer afirmações sobre o conteúdo de
uma página HTML. Felizmente, o Spring vem com um suporte de teste poderoso que torna o teste de um aplicativo da web fácil.
Para os propósitos da página inicial, você escreverá um teste que é comparável em complexidade à própria página inicial. Seu teste executará uma solicitação HTTP GET para o caminho raiz / e esperará um resultado bem-sucedido onde o nome da visualização é home e o conteúdo resultante contém a frase “Bem-vindo a...”. O código a seguir deve resolver o problema.
![[figure1.15.png]]

A primeira coisa que você pode notar sobre este teste é que ele difere um pouco da classe Taco-CloudApplicationTests com relação às anotações aplicadas a ele. Em vez da marcação @SpringBootTest, o HomeControllerTest é anotado com @WebMvcTest.
Esta é uma anotação de teste especial fornecida pelo Spring Boot que organiza a execução do teste no contexto de um aplicativo Spring MVC. Mais especificamente, neste caso, ele organiza o registro do HomeController no Spring MVC para que você possa enviar solicitações a ele.
@WebMvcTest também configura o suporte do Spring para testar o Spring MVC. Embora pudesse ser feito para iniciar um servidor, simular a mecânica do Spring MVC é suficiente para seus propósitos. A classe de teste é injetada com um objeto MockMvc para o teste conduzir o mockup.
O método testHomePage() define o teste que você deseja executar na página inicial. Ele começa com o objeto MockMvc para executar uma solicitação HTTP GET para /
(o caminho raiz). 
A partir dessa solicitação, ele define as seguintes expectativas:
- A resposta deve ter um status HTTP 200 (OK).
- A visualização deve ter um nome lógico de home.
- A visualização renderizada deve conter o texto “Bem-vindo a...”
Você pode executar o teste no IDE de sua escolha ou com o Maven assim:
```java
$ mvnw test
```


Se, após o objeto MockMvc executar a solicitação, qualquer uma dessas expectativas não for atendida, o teste falhará. Mas seu controlador e modelo de visualização são escritos para satisfazer essas expectativas, então o teste deve passar com louvor — ou pelo menos com algum tom de verde indicando um teste aprovado.
O controlador foi escrito, o modelo de visualização criado e você tem um
teste aprovado. Parece que você implementou a página inicial com sucesso. Mas mesmo que o teste passe, há algo um pouco mais satisfatório em ver os resultados em um
navegador. Afinal, é assim que os clientes do Taco Cloud vão ver. Vamos construir o aplicativo e executá-lo.

### Construindo e executando o aplicativo
Assim como temos várias maneiras de inicializar um aplicativo Spring, também temos várias maneiras de executar um. Se quiser, você pode ir até o apêndice para ler sobre algumas das
maneiras mais comuns de executar um aplicativo Spring Boot. Como você escolheu usar o Spring Tool Suite para inicializar e trabalhar no projeto,
você tem um recurso útil chamado Spring Boot Dashboard disponível para ajudar a executar seu aplicativo dentro do IDE.
O Spring Boot Dashboard aparece como uma aba, normalmente perto do canto inferior esquerdo da janela do IDE. A Figura 1.7 mostra uma captura de tela anotada do Spring Boot Dashboard.
![[figure1.16.png]]

Coisas a saber agora é como usá-lo para executar o aplicativo Taco Cloud.
Certifique-se de que o aplicativo taco-cloud esteja destacado na lista de projetos (é o único aplicativo mostrado na figura 1.7) e, em seguida, clique no botão iniciar (o botão mais à esquerda com um triângulo verde e um quadrado vermelho). O aplicativo deve iniciar imediatamente.
Conforme o aplicativo inicia, você verá algumas artes Spring ASCII voando no console, seguidas por algumas entradas de log descrevendo as etapas conforme o aplicativo inicia. Antes que o log pare, você verá uma entrada de log dizendo que o Tomcat iniciou na(s) porta(s): 8080 (http), o que significa que você está pronto para apontar seu navegador da web para a página inicial para ver os frutos do seu trabalho.
Espere um minuto. O Tomcat iniciou? Quando você implantou o aplicativo em um servidor web Tomcat?
Os aplicativos Spring Boot tendem a trazer tudo o que precisam com eles e não
precisam ser implantados em algum servidor de aplicativos. Você nunca implantou seu aplicativo no Tomcat — o Tomcat é parte do seu aplicativo! (Descreverei os detalhes de como o Tomcat se tornou parte do seu aplicativo na seção 1.3.6.)
Agora que o aplicativo foi iniciado, aponte seu navegador da web para http://local-
host:8080 (ou clique no botão de globo no Spring Boot Dashboard) e você deverá ver algo como a figura 1.8. Seus resultados podem ser diferentes se você projetou sua própria
imagem de logotipo, mas não deve variar muito do que você vê na figura 1.8.

![[figure1.17.png]]

Pode não ser muito para se olhar. Mas este não é exatamente um livro sobre design gráfico. A aparência humilde da página inicial é mais do que suficiente por enquanto. E fornece
a você um começo sólido para conhecer o Spring. Uma coisa que ignorei até agora é o DevTools. Você o selecionou como uma dependência ao inicializar seu projeto. Ele aparece como uma dependência no arquivo pom.xml gerado.
E o Spring Boot Dashboard até mostra que o projeto tem o DevTools habilitado.
Mas o que é o DevTools e o que ele faz por você? Vamos fazer uma rápida pesquisa sobre alguns dos recursos mais úteis do DevTools.

### 1.3.5 Conhecendo o Spring Boot DevTools
Como o nome sugere, o DevTools fornece aos desenvolvedores do Spring algumas ferramentas úteis para o tempo de desenvolvimento. Entre elas estão as seguintes:
- Reinicialização automática do aplicativo quando o código muda
- Atualização automática do navegador quando os recursos destinados ao navegador (como modelos, JavaScript, folhas de estilo e assim por diante) mudam
- Desativação automática de caches de modelo
- Construído no H2 Console, se o banco de dados H2 estiver em uso
É importante entender que o DevTools não é um plugin IDE, nem requer que você use um IDE específico. Ele funciona igualmente bem no Spring Tool Suite, IntelliJ IDEA e NetBeans.
Além disso, como ele é destinado apenas para fins de desenvolvimento, ele é
inteligente o suficiente para se desativar ao implantar em um ambiente de produção. Discutiremos como ele faz isso quando você começar a implantar seu aplicativo no capítulo 18.
Por enquanto, vamos nos concentrar nos recursos mais úteis do Spring Boot DevTools, começando com reinicialização automática do aplicativo.

### Reinício Automático do Aplicativo
Com o DevTools como parte do seu projeto, você poderá fazer alterações no código Java e
arquivos de propriedades no projeto e ver essas alterações aplicadas após um breve momento.
O DevTools monitora as alterações e, quando vê que algo mudou, ele reinicia o aplicativo automaticamente.
Mais precisamente, quando o DevTools está ativo, o aplicativo é carregado em dois carregadores de classe separados na máquina virtual Java (JVM). Um carregador de classe é carregado com seu código Java, arquivos de propriedade e praticamente qualquer coisa que esteja no caminho src/main/ do projeto.
Esses são itens que provavelmente mudarão com frequência. O outro carregador de classe é carregado com bibliotecas de dependência, que provavelmente não mudarão com tanta frequência. Quando uma alteração é detectada, o DevTools recarrega apenas o carregador de classe que contém o código do seu projeto e reinicia o contexto do aplicativo Spring, mas deixa o outro carregador de classe e a JVM intactos. Embora sutil, essa estratégia proporciona uma pequena redução no tempo que leva para iniciar o aplicativo.
A desvantagem dessa estratégia é que as alterações nas dependências não estarão disponíveis em reinicializações automáticas. Isso ocorre porque o carregador de classes que contém bibliotecas de dependência não é recarregado automaticamente. Sempre que você adicionar, alterar ou remover uma dependência em sua especificação de compilação, será necessário fazer uma reinicialização forçada do aplicativo para que essas alterações entrem em vigor.

### Atualização Automática do Navegador e Desativação do Cache de Modelo
Por padrão, opções de modelo como Thymeleaf e FreeMarker são configuradas para
armazenar em cache os resultados da análise de modelo para que os modelos não precisem ser analisados ​​novamente a cada solicitação que atendem.
Isso é ótimo em produção, porque compra um pouco de benefício de desempenho.
Os modelos em cache, no entanto, não são tão bons em tempo de desenvolvimento.
Eles tornam impossível fazer alterações nos modelos enquanto o aplicativo está em execução e ver os resultados após atualizar o navegador. Mesmo que você tenha feito alterações, o modelo em cache ainda estará em uso até que você reinicie o aplicativo.
O DevTools resolve esse problema desabilitando automaticamente todo o cache de modelos. Faça quantas alterações quiser em seus modelos e saiba que você está a apenas
uma atualização do navegador de distância de ver os resultados.
Mas se você for como eu, nem quer ser sobrecarregado com o esforço de clicar no botão de atualização do navegador. Seria muito melhor se você pudesse fazer as alterações e testemunhar os resultados no navegador imediatamente. Felizmente, o DevTools tem algo especial para aqueles de nós que são preguiçosos demais para clicar em um botão de atualização.
O DevTools habilita automaticamente um servidor LiveReload (http://livereload.com/) junto
com seu aplicativo. Por si só, o servidor LiveReload não é muito útil. Mas quando acoplado a um plug-in de navegador LiveReload correspondente, ele faz com que seu navegador seja atualizado automaticamente quando alterações são feitas em modelos, imagens, folhas de estilo, JavaScript e assim por diante — na verdade, quase tudo que acaba sendo servido ao seu navegador. O LiveReload tem plug-ins de navegador para os navegadores Google Chrome, Safari e Firefox.
(Desculpe, fãs do Internet Explorer e Edge.) Visite http://livereload.com/extensions/ para encontrar informações sobre como instalar o LiveReload para seu navegador.

### Console H2 Integrado
Embora seu projeto ainda não use um banco de dados, isso mudará no capítulo 3. Se você
escolher usar o banco de dados H2 para desenvolvimento, o DevTools também habilitará automaticamente um console H2 que você pode acessar do seu navegador da web. Você só precisa apontar seu navegador da web para http:/ /localhost:8080/h2-console para obter insights sobre os dados com os quais seu aplicativo está trabalhando.
Neste ponto, você escreveu um aplicativo Spring completo, embora simples. Você irá
expandi-lo ao longo do livro. Mas agora é um bom momento para dar um passo para trás
e revisar o que você realizou e como o Spring desempenhou um papel.

### 1.3.6 Review
Pense em como você chegou a esse ponto. Resumindo, você seguiu os seguintes passos para construir seu aplicativo Taco Cloud Spring:
- Você criou uma estrutura de projeto inicial usando o Spring Initializr.
- Você escreveu uma classe controladora para manipular a solicitação da página inicial.
- Você definiu um modelo de visualização para renderizar a página inicial.
- Você escreveu uma classe de teste simples para provar seu trabalho.
Parece bem direto, não é? Com ​​exceção do primeiro passo para inicializar o projeto, cada ação que você tomou foi intensamente focada em atingir o objetivo de produzir uma home page.
Na verdade, quase todas as linhas de código que você escreveu são voltadas para esse objetivo. Sem contar as instruções de importação do Java, conto apenas duas linhas de código na sua classe de controlador e nenhuma linha no modelo de visualização que são específicas do Spring. E embora a maior parte da classe de teste utilize o suporte de teste do Spring, parece um pouco menos invasivo no contexto de um teste.
Esse é um benefício importante do desenvolvimento com o Spring.
Você pode se concentrar no código que atende aos requisitos de um aplicativo, em vez de satisfazer as demandas de um framework. Embora você sem dúvida precise escrever algum código específico do framework de tempos em tempos, geralmente será apenas uma pequena fração da sua base de código. Como eu disse antes, o Spring (com Spring Boot) pode ser considerado o framework sem framework.
Como isso funciona? O que o Spring está fazendo nos bastidores para garantir
que as necessidades do seu aplicativo sejam atendidas? Para entender o que o Spring está fazendo, vamos começar olhando para a especificação de construção.
No arquivo pom.xml, você declarou uma dependência nos iniciadores Web e Thymeleaf. Essas duas dependências trouxeram transitivamente um punhado de outras dependências,
incluindo as seguintes:
- Framework MVC do Spring
- Tomcat incorporado
- Thymeleaf e o dialeto de layout do Thymeleaf
Ele também trouxe a biblioteca de autoconfiguração do Spring Boot para o passeio. Quando o aplicativo inicia, a autoconfiguração do Spring Boot detecta essas bibliotecas e executa automaticamente as seguintes tarefas:
- Configura os beans no contexto do aplicativo Spring para habilitar o Spring MVC
- Configura o servidor Tomcat incorporado no contexto do aplicativo Spring
- Configura um resolvedor de visualização Thymeleaf para renderizar visualizações Spring MVC com modelos Thymeleaf
Em resumo, a autoconfiguração faz todo o trabalho pesado, deixando você se concentrar em escrever código que implementa a funcionalidade do seu aplicativo. Esse é um arranjo muito bom, se você me perguntar!
Sua jornada Spring apenas começou. O aplicativo Taco Cloud tocou apenas em uma
pequena parte do que o Spring tem a oferecer. Antes de dar o próximo passo, vamos pesquisar o cenário do Spring e ver quais marcos você encontrará em sua jornada.

### 1.4 Analisando o cenário do Spring
Para ter uma ideia do cenário do Spring, não procure mais do que a enorme lista de
caixas de seleção na versão completa do formulário da web Spring Initializr. Ela lista mais de 100 opções de dependência, então não vou tentar listá-las todas aqui ou fornecer uma captura de tela. Mas eu o encorajo a dar uma olhada. Enquanto isso, mencionarei alguns dos destaques.

### 1.4.1 O núcleo do Spring Framework
Como você pode esperar, o núcleo do Spring Framework é a base de tudo
no universo Spring. Ele fornece o núcleo do contêiner e da
estrutura de injeção de dependência. Mas ele também fornece alguns outros recursos essenciais.
Entre eles está o Spring MVC, o framework web do Spring. Você já viu como
usar o Spring MVC para escrever uma classe de controlador para lidar com solicitações web. O que você ainda não
viu, no entanto, é que o Spring MVC também pode ser usado para criar APIs REST que produzem
saída não HTML. Vamos nos aprofundar mais no Spring MVC no capítulo 2 e, em seguida,
daremos outra olhada em como usá-lo para criar APIs REST no capítulo 7.
O núcleo do Spring Framework também oferece algum suporte de persistência de dados elementares,
especificamente, suporte JDBC baseado em modelo. Você verá como usar o JdbcTemplate no
capítulo 3.
O Spring inclui suporte para programação de estilo reativo, incluindo um novo
framework web reativo chamado Spring WebFlux que toma muito emprestado do Spring MVC. Você

verá o modelo de programação reativa do Spring na parte 3 e o Spring WebFlux especificamente no capítulo 12.

### 1.4.2 Spring Boot
Já vimos muitos dos benefícios do Spring Boot, incluindo dependências iniciais e autoconfiguração. Certifique-se de que usaremos o máximo possível do Spring Boot ao longo deste livro e evitaremos qualquer forma de configuração explícita, a menos que seja absolutamente necessário. Mas, além das dependências iniciais e autoconfiguração, o Spring Boot também oferece os seguintes outros recursos úteis:
- O Actuator fornece insights de tempo de execução sobre o funcionamento interno de um aplicativo, incluindo métricas, informações de despejo de thread, integridade do aplicativo e propriedades de ambiente disponíveis para o aplicativo.
- Especificação flexível de propriedades de ambiente.
- Suporte de teste adicional além da assistência de teste encontrada no framework principal.
Além disso, o Spring Boot oferece um modelo de programação alternativo baseado em scripts Groovy, chamado de Spring Boot CLI (interface de linha de comando). Com o Spring
Boot CLI, você pode escrever aplicativos inteiros como uma coleção de scripts Groovy e executá-los a partir da linha de comando. Não gastaremos muito tempo com o Spring Boot CLI, mas falaremos sobre ele ocasionalmente quando for adequado às nossas necessidades. O Spring Boot se tornou uma parte tão integral do desenvolvimento Spring que não consigo imaginar desenvolver um aplicativo Spring sem ele.
Consequentemente, este livro adota uma visão centrada no Spring Boot, e você pode me pegar usando a palavra Spring quando estou me referindo a algo que o Spring Boot está fazendo.

### 1.4.3 Spring Data
Embora o Spring Framework principal venha com suporte básico de persistência de dados,
o Spring Data fornece algo bastante surpreendente: a capacidade de definir os
repositórios de dados do seu aplicativo como interfaces Java simples, usando uma convenção de nomenclatura ao definir métodos para direcionar como os dados são armazenados e recuperados.
Além disso, o Spring Data é capaz de trabalhar com vários tipos diferentes de bancos de dados, incluindo relacional (via JDBC ou JPA), documento (Mongo), gráfico (Neo4j), e outros. Você usará o Spring Data para ajudar a criar repositórios para o aplicativo Taco Cloud no capítulo 3.

### 1.4.4 Spring Security
A segurança de aplicativos sempre foi um tópico importante e parece se tornar mais
importante a cada dia. Felizmente, o Spring tem uma estrutura de segurança robusta no SpringSecurity. O Spring Security aborda uma ampla gama de necessidades de segurança de aplicativos, incluindo autenticação, autorização e segurança de API. Embora o escopo do Spring Security seja muito grande para ser coberto adequadamente neste livro, abordaremos alguns dos casos de uso mais comuns nos capítulos 5 e 12.

### 1.4.5 Spring Integration e Spring Batch
Em algum momento, a maioria dos aplicativos precisará se integrar a outros aplicativos ou até mesmo a outros componentes do mesmo aplicativo. Vários padrões de integração de aplicativos surgiram para atender a essas necessidades. Spring Integration e Spring Batch
fornecem a implementação desses padrões para aplicativos Spring.
Spring Integration aborda a integração em tempo real, onde os dados são processados ​​à medida que são disponibilizados. Em contraste, Spring Batch aborda a integração em lote, onde os dados têm permissão para coletar por um tempo até que algum gatilho (talvez um gatilho de tempo) sinalize que é hora do lote de dados ser processado. Você explorará Spring Integration no capítulo 10.

### 1.4.6 Spring Cloud
O mundo do desenvolvimento de aplicativos está entrando em uma nova era em que não mais desenvolveremos nossos aplicativos como unidades monolíticas de implantação única e, em vez disso, comporemos aplicativos de várias unidades de implantação individuais conhecidas como microsserviços.
Os microsserviços são um tópico importante, abordando várias preocupações práticas de desenvolvimento e tempo de execução. Ao fazer isso, no entanto, eles trazem à tona seus próprios desafios. Esses desafios são enfrentados de frente pelo Spring Cloud, uma coleção de projetos para desenvolver aplicativos nativos da nuvem com Spring.
O Spring Cloud cobre muito terreno, e seria impossível cobrir tudo neste livro. Para uma discussão completa do Spring Cloud, sugiro dar uma olhada em Cloud Native Spring in Action de Thomas Vitale (Manning, 2020, www.manning.com/books/cloud-native-spring-in-action).

### 1.4.7 Spring Native
Um desenvolvimento relativamente novo no Spring é o projeto Spring Native. Este projeto experimental permite a compilação de projetos Spring Boot em executáveis ​​nativos usando o compilador de imagem nativa GraalVM, resultando em imagens que iniciam significativamente mais rápido e têm uma pegada mais leve.
Para mais informações sobre Spring Native, consulte https://github.com/spring-projects-experimental/spring-native.

Resumo
- O Spring visa facilitar os desafios do desenvolvedor, como criar aplicativos da web,
trabalhar com bancos de dados, proteger aplicativos e microsserviços.
- O Spring Boot é construído sobre o Spring para tornar o Spring ainda mais fácil com gerenciamento de dependências simplificado, configuração automática e insights de tempo de execução.
- Os aplicativos Spring podem ser inicializados usando o Spring Initializr, que é baseado na web e suportado nativamente na maioria dos ambientes de desenvolvimento Java.
- Os componentes, comumente chamados de beans, em um contexto de aplicativo Spring podem ser declarados explicitamente com Java ou XML, descobertos por arredura de componentes ou configurados automaticamente com autoconfigurações do Spring Boot.