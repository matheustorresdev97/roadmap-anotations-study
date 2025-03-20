### Começando com Spring

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
pág 43