Na Engenharia de Software, um padrão de projeto (ou do inglês design pattern) é uma solução geral para um problema que ocorre com frequência dentro de um determinado contexto no projeto de software.
Um padrão de projeto não é um projeto finalizado que pode ser diretamente transformado em código fonte ou de máquina, ele é uma descrição ou modelo (template) de como resolver um problema que pode ser usado em muitas situações diferentes.

Podemos afirmar que padrões de projetos são melhores práticas formalizadas que o programador possa usar para resolver problemas comuns quando projetar uma aplicação ou sistema ? 
Depende de alguns fatores:

1- Escopo do projeto: Muitas das vezes o escopo do projeto é tão pequeno e objetivo que não se faz necessário;
2- Maturidade do time: Não podemos pensar em adotar padrões de projetos se não há uma compreensão de conceitos e diretrizes por todos os envolvidos;
3- Prazo e orçamento: Em alguns casos para estimativa de prazo e custo de desenvolvimento de um software não é inserido o valor e tempo de esforço em aplicar determinados padrões de projetos.

Após identificar a real necessidade e o impacto em adotar padrões de projetos, é necessário determinar as 04 principais características abaixo:

- Nome do padrão;
- Problema a ser resolvido;
- Solução dada pelo padrão;
- Consequências

Exemplo: Vamos imaginar que fomos contratados para desenvolver um serviço de notificação de cobrança onde o cliente possa escolher se será notificado por e-mail ou sms.


| CAMPO        | VALOR                                                                               |
| ------------ | ----------------------------------------------------------------------------------- |
| Nome         | Fabrica de Notificação                                                              |
| Problema     | Determinar o tipo de notificação de cobrança de acordo com a preferência do cliente |
| Solução      | Criar uma fábrica de Serviços de notificação                                        |
| Consequência | A cada novo meio de notificação, deverá ser criada uma nova implementação           |

#Padrões são melhores práticas formalizadas que o programador pode usar para resolver problemas comuns quando projetar uma aplicação ou sistema. Algumas linguagens de programação adotam o padrão de projeto da programação #orientada-a-objeto onde todo algoritmo é organizado por #classes com #atributos e #métodos

### Padrões GoF

De acordo com o livro "Padrões de Projeto: soluções reutilizáveis de software orientado a objetos", os padrões GoF ('Gang of Four') são divididos em 24 tipos. Em função dessa grande quantidade de padrões, foi necessário classificá-los de acordo com as suas finalidades.

São 3 as classificações/famílias:

1. Padrões de criação: fornecem mecanismos de criação de objetos que aumentam a flexibilidade e a reutilização de código.
2. Padrões estruturais: explicam como montar objetos e classes em estruturas maiores, enquanto ainda mantém as estruturas flexíveis e eficientes.
3. Padrões comportamentais: cuidam da comunicação eficiente e da assinalação de responsabilidades entre objetos.

Abaixo temos uma ilustração dos padrões de projetos distribuídos em
![[Captura de tela de 2025-04-02 19-55-43.png]]

Acreditamos que o maior desafio em implementar um padrão em nosso projeto é identificar um cenário e caso de uso apropriado, esta compreensão exige um certo tempo de interação e entendimento do problema apresentado e solução sugerida por todos os envolvidos.

### Caso de Uso
Abaixo iremos ilustrar alguns cenaŕios que se aplicariam alguns dos padrões mais utilizados em projetos corporativos.
Imagina que sua empresa foi contratada para desenvolver um software para gestão das correspondências de uma plataforma de ecommerce que inicialmente atende as demandas de compras de sua cidade realizando as entregas via motocicleta.

### Factory Method
É um padrão criacional que oferece uma estrutura para criar objetos em uma superclasse, permitindo que subclasses determinem o tipo específico de objeto a ser criado.
Após meses de operação e feedback dos clientes sobre a plataforma de e-commerce, a empresa optou por expandir seu catálogo de produtos. Para atender a essa demanda, surge a necessidade e incluir novas opções de transporte para entrega, como transporte terrestre por carro, e agora também transporte aéreo por avião, visando alcançar outros municípios e estados do país.

### Adapter
O Adapter é um padrão de projeto estrutural que permite objetos com interfaces incompatíveis colaborarem entre si.
Considerando que agora o nosso serviço de logística da empresa de ecommerce possui uma integração com os serviços das empresas aéreas do país, será necessária uma implementação de recursos que possibilitem a adaptação e comunicação entre os dois sistemas.
Estas alternativas podem ser diversas, desde uma definição de estruturação de arquivos txt sendo transmitidos via protocolo ftp até mesmo a implementação de protocolos #http enviando e recebendo arquivos xml e javaon.

### Strategy
É um padrão de projeto comportamental que permite que você defina uma família de algoritmos, coloque-os em classes separadas, e faça os objetos deles intercambiáveis.
Imagina que a empresa de ecommerce agora irá oferecer um plano de serviço que foi carinhosamente chamado de compre e retire com alguns pontos de drive thru espalhados pela cidade para assim agilizar a entrega e ampliar a publicidade de sua marca.
Agora o sistema de gestão logística deverá ter duas estratégias para as entregas dos produtos comprados no site diante da opção de recebimento escolhida pelo cliente.

### SOLID
É o acrônimo criado por Michael Feathers para representar os cinco princípios da programação orientada a objetos e design de códigos que tem por finalidade conduzir o programador para a criação de código com menos complexidade nas futuras manutenções.

| Sigla | Signficado                     | Tradução                              |
| ----- | ------------------------------ | ------------------------------------- |
| S     | Single Responsibility          | Princípio da Responsabilidade Única   |
| O     | Open/closed Principle          | Princípio do aberto/fechado           |
| L     | Liskov Substitution            | Princípio da Substituição de Liskov   |
| I     | Interface Segregation          | Princípio da Segregação de Interfaces |
| D     | Dependency Inversion Principle | Princípio da Inversão de Dependências |
Estes princípios separam responsabilidades diminuindo o acoplamento do código com intuito de facilitar na refatoração, melhorias e evolução do seu projeto. Por isso, antes de pensar em começar aplicando logo de cara todos estes princípios, repense no esforço que será necessário.

### Single Responsibility
Um dos princípios mais importante e acreditamos que o mais utilizados em todas as etapas de desenvolvimento do projeto, o Single Responsibility tem como finalidade definir e manter de forma bem distribuída e independente todo algoritmo e lógica de um software
Em resumo, o Princípio da Responsabilidade Única sugere que uma classe deva ter somente um assunto, uma finalidade e ou objetivo, isso não quer dizer um único atributo ou método.
É muito comum em exemplos acadêmicos e até mesmo em projetos reais nos deparamos com abordagens semelhantes a esta abaixo:
Imagina que a empresa de ecommerce diante de seu sucesso de atuação no seguimento, resolveu diversificar suas atividades proporcionando para alguns de seus clientes a possibilidade dos mesmos criarem uma conta digital capaz de depositar, sacar e o principal, comprar online com muito mais facilidade.

1. Implementação convencional:
```java
public class Conta{
	double saldo;

	void depositar(Double valorInserido) {
		saldo = saldo + valorInserido;
	}
}

	void sacar(Double valorSolicitado) {
		saldo = saldo - valorSolicitado;
	}

	double obterSaldo() {
		return saldo;
	}

	String imprimirExtrato() {
	//Logica que busca e apresenta todas as movimentações desta 
	return "";
	}
		}
	}
	```


Riscos
Precisamos começar a perceber os riscos de manutenibilidade e evolução do nosso software considerando alguns riscos conforme abaixo:
- A classe possui mais de uma responsabilidade (baixa coesão) ou faz praticamente tudo, sendo conhecida como a God Class ou Classe Deus;
- As responsabilidades estão misturadas e confusas dificultando uma interpretação diante da necessidade de uma nautenção;
- Aumento significativo do esforço necessário para implementar melhorias ou novas funcionalidades;
- Incertezas surgem quando confrontadas a lógica aplicada diante dos requisitos apresentados pelos usuários.


2. Implementação com SRP:
Nesta abordagem grande parte do código que existe somente em uma classe, poderá ser distribuído em outras classes em seu projeto.

```java
public class Conta {
	private double saldo;
	
public void atualizarSaldo(Operacao operacao, Double valor) {
	if(Operacao.DEPOSITO == operacao)
		saldo = saldo + valor
	else
		saldo = saldo - valor
	}
	public double obterSaldo() {
		return saldo;
	}
}
```

```java
public class CaixaEletronico {

	void depositar(Conta conta, Double valorInserido) {
		//receber as notas em um envelope
		//algoritmo de verificação de cédulas
		conta.atualizarSaldo(Operacao.DEPOSITO, valorInserido);
	}

	void sacar(Conta conta, Double valorSolicitado) {
		//consultar saldo disponivel
		//algoritmo de contagem das cédulas
		conta.atualizarSaldo();
	}

	double obterSaldo(Conta conta) {
		return conta.obterSaldo();
	}

	String imprimirExtrato(Conta conta) {
	//lógica que busca e apresenta todas as movimentações da conta
	//a conta não precisa saber quais foram suas movimentações
		return '';
	}
}
```


```java
public enum Operacao {
	DEPOSITO,
	SAQUE;
}
```


Diante de toda esta abordagem sobre SRP, agora somos capazes de compreender a importância de aplicar o princípio da responsabilidade única diante das classes de nosso projeto. Não esqueça: Uma classe deve ter um e somente um motivo para mudar.


### Open-Closed
Na programação orientada a objeto, o princípio Open/Closed ou aberto/fechado estabelece que "entidades de software (classes, módulos, funções, etc.) devem ser abertas para extensão, mas fechadas para modificação"; isto é, a entidade pode permitir que o seu comportamento seja estendido sem modificar seu código-fonte.
Muitas das vezes este princípio exige a criação de interfaces ou classes abstratas para proporcionar o mecanismo de extensão comumente utilizada em linguagens orientadas a objetos. Vejamos o nosso contexto de contas ilustrando no princípio da Responsabilidade Única.
Vamos imaginar que a empresa de ecommerce quer agora permitir que seus clientes realize o saque de sua conta considerando os cenários abaixo:
- Quando o saque for realizado em um caixa eletrônico em seus pontos de apoio, o mesmo receberá o dineiro em espécie;
- Novo recurso: Quando o saque for realizado pelo site, uma transação via pix deverá ser realizada de acordo com os dados cadastrais do correntista.

1. Implementação convencional:
```java
public class CaixaEletronico {
	void sacar(Conta conta, String tipoSaque, Double valorSolicitado) {
		conta.atualizarSaldo(Operacao.SAQUE, valorSolicitado);
		//novos comportamentos
		if(tipoSaque == 'ESPECIE')
		 //o dinheiro será liberado pelo caixa eletrônico
		else
		 //uma transação via pix será gerada creditando a conta pix do correntista
	}
}
```

2. Implementação Open/Closed:
```java
public interface Terminal {
	void sacar(Conta conta, Double valorSolicitado);

	//os métodos de uma interface ou de uma classe abstrata não possuem corpo.
}
```

```java
public class CaixaEletronico implements Terminal {
	void sacar(Conta conta, Double valorSolicitado) {
		conta.atualizarSaldo(Operacao.SAQUE, valorSolicitado);
		
	//o dinheiro será liberado pelo caixa eletrônico
	}
}
```

```java
public class CaixaVirtual implements Terminal {
	void sacar(Conta conta, Double valorSolicitado) {
		conta.atualizarSaldo(Operacao.SAQUE, valorSolicitado);

	//uma transação via pix será gerada creditando a conta pix
	}
}
```

Alterar classes já existentes para atender aos novos requisitos pode ampliar os riscos de introduzirmos bugs em nosso sistema. Então, lembre-se, aplicar a abstração via interfaces ou classes abstratas poderá deixar seu algoritmo mais flexível às nossas solicitações no projeto.

### Liskov
Na programação orientada a objetos, o princípio da substituição de Liskov ou Liskov Substitution é uma definição partiular para o conceito de subtipo, que diz que: Sua classe derivada deve ser substituída por sua classe base.
Considerando que nosso sistema precisará a partir de agora exibir o nome de seus clientes considerando que estes clientes poderão ser pessoas físicas ou empresas em seu módulo de cadastros, logo, vejamos o contexto abaixo:

```java
//o uso do conceito abstract é circunstâncial
	public abstract class Cliente {
		String getNome();
		}
	}
	```

```java
	public class ClientePessoaFisica extends Cliente{
		String nome;
		String getNome(){
			return nome;
			}
	}
	```

```java
public class ClientePessoaFisica extends Cliente{
		String nomeFanstasia;
		String getNome(){
			return nomeFantasia;
			}
	}
	```

```java
public class EmissorNotaFiscal {
	public void imprimirNotaFiscal(Cliente cliente) {

	print(cliente.getNome());
	}
}
```

```java
public class Sistema {

	public static void main() {

	Cliente pessoaFisica = new ClientePessoaFisica();
	Cliente empresa = new ClienteEmpresa();

	EmissorNotaFiscal emissor = new EmissorNotaFiscal();

	emissor.imprimirNotaFiscal(pessoaFisica);
	emissor.imprimirNotaFiscal(empresa);
	}
}
```

### Interface Segregation
No campo da engenharia de software, o princípio da segregação de Interface (Interface Segregation) afirma que nenhum cliente deve ser forçados a depender de métodos que não utiliza. ISP divide interfaces que são muito grandes em menores e mais específicas, para que os clientes só necessitem saber sobre os métodos que são de interesse para eles.

A nossa empresa de ecommerce solicitou para o nosso time de desenvolvedores que agora será possível transferir o saldo em conta para outras contas cadastradas em sua plataforma, porém, somente via online ou caixa virtual.

```java
	public interface Terminal {
		void sacar(Conta conta, Double valorSolicitado);
		void transferir(Conta contaOrigem);

		//Se este recurso for adicionado nesta interface, todas as classes devem implementar
	}
```

```java
	public interface TerminalTransferencia extends Terminal {
		// nova interface contendo os novos métodos adequadamente
		void transferir(Conta contaOrigem, Conta contaDestino, Double valorInformado);
	}
```

```java
	public class CaixaEletronico implements Terminal {
		void sacar(Conta conta, Double valorSolicitado){}
	}
```

```java
	public class CaixaVirtual implements Terminal, TerminalTransferencia {
	void sacar(Conta conta, Double valorSolicitado){}

	void transferir(Conta contaOrigem, Conta contaDestino, Double valorInformado) {

	//logica de transferencia realizada no CaixaVirtual
	}
	}
```

### Dependency Inversion
No paradigma de orientação a objetos, o Princípio da inversão de dependência ou Dependency Inversion refere-se a uma forma específica de desacoplamento de módulos de software que determina a inversão das relações de dependência.
O DIP reforça que é mais conveniente depender de abstrações e não de implementações com base em suas duas definições:

1. Módulos de alto nível não devem depender de módulos de baixo nível. Ambos devem depender de abstrações;
2. As abstrações não devem depender de detalhes. Os detalhes devem depender das abstrações

Inversão de dependências não é o mesmo que injeção de dependências, mesmo que ambos possuem a finalidade de desacoplar o código ou implementação real.
Vamos imaginar que a empresa de ecommerce iniciou uma ação de marketing para divulgar um novo produto e contratou três agências de marketing para cuidar da criação do material que será apresentado em forma de três comerciais ao longo de uma semana.

```java
public interface Agencia {
	void divulgarProduto(Produto produto);
}
```

```java
public class Agencia01 implements Agencia {
	void divulgarProduto(Produto produto) {
		print("conteúdo de apresentação da Agencia 01");
	}
}
```

```java
public class Agencia02 implements Agencia {
	void divulgarProduto(Produto produto) {
		print("conteúdo de apresentação da Agencia 02");
	}
}
```

```java
public class LancamentoProduto {

	Agencia agencia;

	LancamentoProduto(Agencia agenciaProgramada) {

	//agenciaProgramada corresponderá a ordem de apresentação das agências contratadas para divulgar o produto.

	agencia = agenciaProgramada;

	}

	void iniciar() {
		Produto produto = produto selecionado;

	//abstração sem a necessidade de conhecer a estratégia da agência
	agencia.divulgarProduto(produto);
	}
}
```


### Domain-Driven Design

É uma abordagem de modelagem de software. Seu propósito é focar no desenvolvimento na criação de um modelo de domínio. Esse modelo é baseado nas competências da organização, considerando a complexidade do negócio.
Não entenda design apenas como layout ou visual da sua aplicação.
O intuito principal do DDD é oferecer recursos e diretrizes diante de uma visão tática e estratégica e assim poder assegurar a entrega de um software com alta qualidade, mesmo em cenários com uma complexidade considerável no negócio. Também não podemos deixar de mencionar que é possível aplicar DDD independente da linguagem e tecnologia em seus projetos.
- design estratégico: nos ajuda a resolver problemas relacionados à modelagem de software;
- design tático: ocorre após a fase estratégica, se concentra no desenvolvimento do produto, focado nos detalhes de implementação.
Um dos principais motivos do fracasso em tentar implementar DDD em alguns projetos, é que as empresas e os stakeholders não consideram em seu planejamento de esforço, tempo e custo os requisitos necessários para adotar suas diretrizes. Resumindo, não existe uma estimativa relevante para uma abordagem estratégica previamente estruturada.

Para aplicar e seguir o DDD é necessário conhecer e compreender seus princípios que por sua vez eles concentram todos os seus esforços para expandir o conhecimento e a compreensão do negócio, visando atender às necessidades e expectativas dos usuários:

1. Discussão
2. Escuta
3. Compreensão


#### Por que utilizar DDD?

DDD é uma jornada, não o destino. A busca pela harmonia do domínio nunca termina. O domínio muda à medida que o negócio muda. Mantém o software alinhado com o negócio. Faz isso estratégica, tática e filosoficamente.
Praticar DDD, significa aproximar os especialistas do domínio (Domain experts) do time de desenvolvimento. Criando uma linguagem única. Uma comunicação sem ruídos, limpa.

### Domínio
O termo domínio refere-se ao propósito, negócio da sua empresa ou assunto do seu projeto que podemos reduzir ainda mais para o core-domain ou domínio central que é parte principal de um domínio.
A implementação de DDD em um projeto requer um time de desenvolvedores e especialistas do negócio, os domain experts. Por exemplo, ao desenvolver um software de gestão logística, os profissionais do centro de distribuição seriam esses experts

### Linguagem Ubíqua
Para que todos consigam aprender e compreender as especificações e diretrizes do projeto, é necessário que haja uma comunicação única e simples, esta abordagem é conhecida como linguagem ubíqua.
A linguagem ubíqua, é uma linguagem da equipe, compartilhada e principalmente entendida por todos e que também pode evoluir ao longo do projeto.

### Bounded Context
O outro pilar do DDD é a delimitação do contexto ou Bounded Context, que representa o limite conceitual do projeto. Em um projeto você pode se deparar com um dompinio relativamente extenso e complexo, o Bounded Context tem a intenção de delimitar ou criar sub-domínios baseando-se nas intenções de negócio.
Cada bounded context poderá ter a sua linguagem ubíqua, sua abordagem de arquitetura, de armazenamento de dados e até a própria equipe de desenvolvimento.

Seguindo o nosso contexto ilustrativo que apresentamos no tópico Design Pattern com base na proposta de um software para gestão de uma empresa de ecommerce, iremos ilustrar como é realizada a delimitação dos contextos de negócio que divide a complexidade de sua aplicação em sub-domínios mais simples para a interpretação de todos os envolvidos proporcionando assim uma linguagem ubíqua e melhor entendimento.
Em nosso exemplo de Design Pattern para um software de gestão de ecommerce, ilustramos a delimitação dos contextos de negócio. Essa delimitação divide a complexidade da aplicação em sub-domínios mais simples. Isso facilita a interpretação de todos os envolvidos, proporcionando uma linguagem ubíqua e melhor entendimento.

![[Captura de tela de 2025-04-07 19-21-55.png]]

Vale ressaltar que uma entidade de domínio poderá assumir identidades e papéis diferentes de acordo com o sub-domínio que a mesma esteja envolvida, exemplo: Cliente, Destinatário e Remetente, estas três entidades representam a pessoa que está adquirindo um produto através de uma compra pelo site onde também as mesmas poderão ter características distintas e similares.

### Context Map
O mapeamento de cenários é uma ferramenta que permite identificar a conexão entre ambientes definidos e a relação entre as equipes encarregadas por eles.
Os principais tipos de relacionamentos entre um contexto são:

- Shared Kernel: Quando vários bounded context compartilham de um mesmo domínio;
- Customer Supplier: Quando existe uma relação estilo cliente downstream e fornecedor upstream, onde a equipe upstream pode ter êxito independente da equipe downstream;
- Partnership: Quando equipes em dois contextos conseguem ou fracassam juntas, um relacionamento cooperativo precisa emergir.
- Anticorruption Layer: Essa abordagem geralmente é útil para integrar novos recursos a alguns softwares legados existentes.


### Evento de domínio
Um evento é um acontecimento passado. Uma ocorrência de domínio é algo que ocorreu em uma área específica que você deseja que outras partes do mesmo domínio (em processo) estejam cientes. As partes notificadas geralmente reagem aos eventos.
Um benefício importante dos eventos de domínio é que os efeitos colaterais podem ser expressos explicitamente.


![[Captura de tela de 2025-04-07 19-31-54.png]]


Quando o usuário inicia um pedido, a junção Order emite um acontecimento de domínio OrderStarted. O domínio de evento OrderStarted é manipulado pela agregação Comprador dispara a criação de um objeto Comprador para criar um objeto Buyer no microsserviço de pedidos, com base nas informações do usuário original, obtidas do microsserviço de identidade


### Implementação 
Considerando a afirmação de Vaughn Vernon que diz que "O design estratégico é como um rascunho antes de entrar nos detalhes da implementação." Chegou a hora de conhecer um pouco sobre a abordagem tática do domínio.

#### Domain Model Pattern
O Domain Model Pattern, ou padrão de modelo de domínio, é uma estratégia que orienta como compor as classes que irão representar as versões do mundo real e implementar os comportamentos do negócio.
A seguir, são mostrados alguns padrões de modelo usados frequentemente na modelagem tática em projetos de diversos segmentos de negócio.

- Entities: Classes que representam modelos de objetos que necessitam de um identificador;

- Value Objects: São objetos imutáveis e sem identidade que se agregam a outros objetos (Entities);

- Aggregate Object: Uma entidade que é a raiz agregadora de um processo do domínio que envolve mais de uma entidade;

- Factory: Classe responsável por construir adequadamente um objeto/entidade;

- Domain Service: Serviço do domínio lida com aspectos do negócio que não são naturalmente representados por entidades específicas. Ele coordena operações que envolvem múltiplas entidades, utiliza repositórios para persistência de dados e desempenha funções cruciais no domínio do problema;

- Repository: Uma classe que realiza a persistência das entidades se comunicando diretamente com o meio de acesso aos dados, é utilizado apenas um repositório por agregação;

- Application Service: Serviço de aplicação que coordena ações acionadas pela estrutura de apresentação e fornece DTOs para comunicação entre os demais níveis e para o uso da estrutura de apresentação;

- External Service: Serviço externo que realiza a consulta/persistência de informações por meios diversos.


![[Captura de tela de 2025-04-07 19-46-21.png]]

