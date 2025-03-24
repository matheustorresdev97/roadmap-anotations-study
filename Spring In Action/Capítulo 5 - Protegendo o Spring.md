Este capítulo abrange:
	Autoconfiguração do Spring Security
	Definição de armazenamento de usuário personalizado
	Personalização da página de login
	Proteção contra ataques CSRF
	Conhecendo seu usuário

Você já notou que a maioria das pessoas em seriados de TV não trancam suas portas? Na época de Leave It to Beaver, não era tão incomum que as pessoas deixassem suas portas destrancadas. Mas parece loucura que em uma época em que estamos preocupados com privacidade e segurança, vemos personagens de televisão permitindo acesso irrestrito a seus apartamentos e casas. A informação é provavelmente o item mais valioso que temos agora; bandidos estão procurando maneiras de roubar nossos dados e identidades entrando furtivamente em aplicativos não seguros. Como desenvolvedores de software, devemos tomar medidas para proteger as informações que residem em
nossos aplicativos. Seja uma conta de e-mail protegida com um par de nome de usuário e senha ou uma conta de corretora protegida com um PIN de negociação, a segurança é um aspecto crucial da maioria dos aplicativos.
### 5.1 Habilitando o Spring Security
O primeiro passo para proteger seu aplicativo Spring é adicionar a dependência inicial de segurança Spring Boot à sua compilação. No arquivo pom.xml do projeto, adicione a seguinte entrada < dependency >:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

Se você estiver usando o Spring Tool Suite, isso é ainda mais fácil. Clique com o botão direito do mouse no arquivo pom.xml e selecione Edit Starters no menu de contexto do Spring. Na caixa de diálogo de dependências do starter, selecione a entrada Spring Security na categoria Security, conforme mostrado na figura 5.1. Acredite ou não, essa dependência é a única coisa necessária para proteger um aplicativo. Quando o aplicativo for iniciado, a autoconfiguração detectará que o #Spring-Security está no classpath e definirá algumas configurações básicas de segurança. Se quiser experimentar, inicie o aplicativo e tente visitar a página inicial (ou qualquer página, nesse caso). Você será solicitado a autenticar com uma página de login bastante simples que se parece com a figura 5.2
DICA Ficar anônimo: você pode achar útil definir seu navegador para o modo privado ou anônimo ao testar a segurança manualmente. Isso garantirá que você tenha uma nova sessão sempre que abrir uma janela privada/anônima. Você terá que entrar no aplicativo todas as vezes, mas pode ter certeza de que todas as alterações que você fez na segurança serão aplicadas e que não há resquícios de uma sessão mais antiga impedindo que você veja suas alterações. Para passar pela página de login, você precisará fornecer um nome de usuário e uma senha. O nome de usuário é user. Quanto à senha, ela é gerada aleatoriamente e gravada no arquivo de log do aplicativo. A entrada de log será parecida com esta: 
Usando a senha de segurança gerada: 087cfc6a-027d-44bc-95d7-cbb3a798a1ea
Supondo que você insira o nome de usuário e a senha corretamente, você terá acesso ao aplicativo.
Parece que proteger aplicativos Spring é um trabalho bem fácil.  Com o aplicativo Taco Cloud protegido, acho que poderia terminar este capítulo agora e passar para o próximo tópico. Mas antes de nos adiantarmos, vamos considerar que tipo de segurança a autoconfiguração forneceu.

Ao fazer nada mais do que adicionar o iniciador de segurança à construção do projeto, você obtém os seguintes recursos de segurança:
- Todos os caminhos de solicitação HTTP exigem autenticação.
- Nenhuma função ou autoridade específica é necessária.
- A autenticação é solicitada com uma página de login simples.
- Há apenas um usuário, o nome de usuário é user.

![[Captura de tela de 2025-03-24 12-00-01.png]]
![[Captura de tela de 2025-03-24 12-00-10.png]]

Este é um bom começo, mas acho que as necessidades de segurança da maioria dos aplicativos (incluindo o Taco Cloud) serão bem diferentes desses recursos rudimentares de segurança.
Você tem mais trabalho a fazer se quiser proteger adequadamente o aplicativo Taco Cloud. Você precisará pelo menos configurar o Spring Security para fazer o seguinte:
- Fornecer uma página de login projetada para corresponder ao site.
- Fornecer vários usuários e habilitar uma página de registro para que novos clientes do Taco Cloud possam se inscrever.
- Aplicar regras de segurança diferentes para diferentes caminhos de solicitação. A página inicial e as páginas de registro, por exemplo, não devem exigir autenticação alguma.

Para atender às suas necessidades de segurança para o Taco Cloud, você terá que escrever alguma configuração explícita, substituindo o que a autoconfiguração lhe deu. Você começará configurando um armazenamento de usuário adequado para que você possa ter mais de um usuário.

### 5.2 Configurando a autenticação
Ao longo dos anos, várias maneiras de configurar o Spring Security existiram, incluindo configuração XML longa. Felizmente, várias versões recentes do Spring Security
deram suporte à configuração Java, que é muito mais fácil de ler e escrever.
Antes de terminar este capítulo, você terá configurado todas as suas necessidades de segurança do Taco Cloud em uma configuração Java para o Spring Security. Mas, para começar, você vai facilitar escrevendo a classe de configuração mostrada na listagem a seguir.

Listagem 5.1 Uma classe de configuração barebones para Spring Security

```java
package tacos.security;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
public class SecurityConfig {

@Bean
public PasswordEncoder passwordEncoder() {
	return new BCryptPasswordEncoder();
}

}
```

O que essa configuração de segurança básica faz por você? Não muito, na verdade. A principal coisa que ela faz é declarar um #bean PasswordEncoder, que usaremos ao criar novos usuários e ao autenticar usuários no login. Nesse caso, estamos usando BCryptPasswordEncoder, um dos vários codificadores de senha fornecidos pela Spring Security, incluindo os seguintes:
- BCryptPasswordEncoder - Aplica criptografia de hash forte bcrypt
- NoOpPasswordEncoder - Não aplica codificação
- Pbkdf2PasswordEncoder - Aplica criptografia PBKDF2
- SCryptPasswordEncoder - Aplica criptografia de hash Scrypt
- StandardPasswordEncoder - Aplica criptografia de hash SHA-256

Não importa qual codificador de senha você use, é importante entender que a senha no banco de dados nunca é decodificada.
Em vez disso, a senha que o usuário insere no login é codificada usando o mesmo algoritmo e é então comparada com a senha codificada no banco de dados.
Essa comparação é realizada no método matches() do PasswordEncoder.
Qual codificador de senha você deve usar?
Nem todos os codificadores de senha são criados iguais. No final das contas, você precisará pesar cada algoritmo do codificador de senha em relação às suas metas de segurança e decidir por si mesmo. Mas você deve evitar alguns codificadores de senha para aplicativos de produção.
O NoOpPasswordEncoder não aplica nenhuma criptografia. Portanto, embora possa ser útil para testes, ele não é adequado para uso em produção. E o StandardPassword-
Encoder não é considerado seguro o suficiente para criptografia de senha e, de fato, foi descontinuado.
Em vez disso, considere um dos outros codificadores de senha, qualquer um dos quais é mais seguro.
Usaremos o BCryptPasswordEncoder para os exemplos neste livro.
Além do codificador de senha, preencheremos esta classe de configuração com mais beans para definir as especificidades de segurança para nosso aplicativo. Começaremos configurando um armazenamento de usuário que pode manipular mais de um usuário.
Para configurar um armazenamento de usuário para fins de autenticação, você precisará declarar um bean UserDetailsService é relativamente simples, incluindo apenas um método que deve ser implementado. Aqui está a aparência do UserDetailsService:

```java
public interface UserDetailsService {
UserDetails loadUserByUsername(String username) throws
UsernameNotFoundException;
}
```

O método loadUserByUsername() aceita um nome de usuário e o usa para procurar um objeto UserDetails. Se nenhum usuário puder ser encontrado para o nome de usuário fornecido, ele lançará uma UsernameNotFoundException. Como se vê, o Spring Security oferece várias implementações prontas para uso de UserDetailsService, incluindo o seguinte:
- Um armazenamento de usuário na memória
- Um armazenamento de usuário JDBC
- Um armazenamento de usuário LDAP

Ou você também pode criar sua própria implementação para atender às necessidades específicas de segurança do seu aplicativo.
Para começar, vamos experimentar a implementação na memória de UserDetailsService.

### 5.2.1 Serviço de detalhes do usuário na memória
Um lugar onde as informações do usuário podem ser mantidas é na memória. Suponha que você tenha apenas um punhado de usuários, nenhum dos quais provavelmente mudará. Nesse caso, pode ser simples o suficiente definir esses usuários como parte da configuração de segurança.
O método de bean a seguir mostra como criar um InMemoryUserDetailsManager com dois usuários, “buzz” e “woody”, para esse propósito.

Listagem 5.2 Declarando usuários em um bean de serviço de detalhes do usuário na memória

```java
@Bean
public UserDetailsService userDetailsService(PasswordEncoder encoder) {
List<UserDetails> usersList = new ArrayList<>();
usersList.add(new User(
"buzz", encoder.encode("password"),
Arrays.asList(new SimpleGrantedAuthority("ROLE_USER"))));
usersList.add(new User(
"woody", encoder.encode("password"),
Arrays.asList(new SimpleGrantedAuthority("ROLE_USER"))));
return new InMemoryUserDetailsManager(usersList);
}
```

Aqui, uma lista de objetos de usuário do Spring Security é criada, cada um com um nome de usuário, senha e uma lista de uma ou mais autoridades. Então, um InMemoryUserDetailsManager é criado usando essa lista. Se você experimentar o aplicativo agora, você deve conseguir fazer login como “woody” ou
“buzz”, usando password como senha. O serviço de detalhes do usuário na memória é conveniente para fins de teste ou para aplicativos muito simples, mas não permite edição fácil de usuários. Se você precisar adicionar, remover ou alterar um usuário, você terá que fazer as alterações necessárias e então reconstruir
e reimplantar o aplicativo. Para o aplicativo Taco Cloud, você quer que os clientes consigam se registrar no aplicativo e gerenciar suas próprias contas de usuário. Isso não se encaixa com as limitações do serviço de detalhes do usuário na memória. Então, vamos dar uma olhada em como criar nossa própria implementação do UserDetailsService que permite um banco de dados de armazenamento de usuário.

### 5.2.2 Personalizando a autenticação do usuário
No capítulo anterior, você decidiu usar o Spring Data JPA como sua opção de persistência para todos os dados de tacos, ingredientes e pedidos. Portanto, faria sentido persistir os dados do usuário da mesma forma. Se você fizer isso, os dados residirão em um banco de dados relacional, então você pode usar a autenticação JDBC. Mas seria ainda melhor aproveitar o repositório Spring Data JPA usado para armazenar usuários.
Primeiras coisas primeiro. Vamos criar o objeto de domínio e a interface do repositório que representa e persiste as informações do usuário.

DEFININDO O DOMÍNIO DO USUÁRIO E A PERSISTÊNCIA

Quando os clientes do Taco Cloud se registram no aplicativo, eles precisam fornecer mais do que apenas um nome de usuário e senha. Eles também fornecerão seu nome completo, endereço e número de telefone. Essas informações podem ser usadas para uma variedade de propósitos, incluindo o pré-preenchimento do formulário de pedido (sem mencionar potenciais oportunidades de marketing).
Para capturar todas essas informações, você criará uma classe User, da seguinte forma.

Listagem 5.3 Definindo uma entidade de usuário

```java
package tacos;
import java.util.Arrays;
import java.util.Collection;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.
SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import lombok.AccessLevel;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.RequiredArgsConstructor;

@Entity
@Data
@NoArgsConstructor(access=AccessLevel.PRIVATE, force=true)
@RequiredArgsConstructor
public class User implements UserDetails {
private static final long serialVersionUID = 1L;
@Id
@GeneratedValue(strategy=GenerationType.AUTO)
private Long id;
private final String username;
private final String password;
private final String fullname;
private final String street;
private final String city;
private final String state;
private final String zip;
private final String phoneNumber;
@Override
public Collection<? extends GrantedAuthority> getAuthorities() {
return Arrays.asList(new SimpleGrantedAuthority("ROLE_USER"));
}
@Override
public boolean isAccountNonExpired() {
return true;
}
@Override
public boolean isAccountNonLocked() {
return true;
}
@Override
public boolean isCredentialsNonExpired() {
return true;
}
@Override
public boolean isEnabled() {
return true;
}
}
```


A primeira coisa a notar sobre esse tipo de usuário é que ele não é o mesmo que a classe User que usamos ao criar o serviço de detalhes do usuário na memória. Este tem mais detalhes sobre o usuário que precisaremos para atender pedidos de taco, incluindo o endereço do usuário e informações de contato.
Você provavelmente também notou que a classe User é um pouco mais envolvida do que qualquer uma das outras entidades definidas no capítulo 3. Além de definir um punhado de propriedades, User também implementa a interface UserDetails do Spring Security.
As implementações de UserDetails fornecerão algumas informações essenciais do usuário para a estrutura, como quais autoridades são concedidas ao usuário e se a
conta do usuário está habilitada.
O método getAuthorities() deve retornar uma coleção de autoridades concedidas ao usuário. Os vários métodos is* retornam um booleano para indicar se a
conta do usuário está habilitada, bloqueada ou expirada.
Para sua entidade User, o método getAuthorities() simplesmente retorna uma coleção indicando que todos os usuários terão recebido a autoridade ROLE_USER. E, pelo menos por agora, o Taco Cloud não precisa desabilitar usuários, então todos os métodos is* retornam true para indicar que os usuários estão ativos.
Com a entidade User definida, agora você pode definir a interface do repositório da seguinte forma:

```java
package tacos.data;
import org.springframework.data.repository.CrudRepository;
import tacos.User;
public interface UserRepository extends CrudRepository<User, Long> {
	User findByUsername(String username);
}
```

Além das operações CRUD fornecidas pela extensão do CrudRepository, o UserRepository define um método findByUsername() que você usará no serviço de detalhes do usuário para procurar um usuário pelo nome de usuário.
Como você aprendeu no capítulo 3, o Spring Data JPA gera automaticamente a implementação dessa interface em tempo de execução. Portanto, agora você está pronto para escrever um serviço de detalhes do usuário personalizado que usa esse repositório.

CRIANDO UM SERVIÇO DE DETALHES DO USUÁRIO
Como você deve se lembrar, a interface UserDetailsService define apenas um único método loadUserByUsername(). Isso significa que é uma interface funcional e pode ser implementada como um #lambda em vez de uma classe de implementação completa. Como tudo o que realmente precisamos
é que nosso UserDetailsService personalizado delegue ao UserRepository, ele pode ser
simplesmente declarado como um bean usando o seguinte método de configuração.

Listagem 5.4 Definindo um bean de serviço de detalhes do usuário personalizado

```java
@Bean
public UserDetailsService userDetailsService(UserRepository userRepo) {
return username -> {
User user = userRepo.findByUsername(username);
if (user != null) return user;
throw new UsernameNotFoundException("User '" + username + "' not found");
};
}
```

O método userDetailsService() recebe um UserRepository como parâmetro. Para criar o bean, ele retorna um #lambda que pega um parâmetro username e o usa para chamar
findByUsername() no UserRepository fornecido.
O método loadByUsername() tem uma regra simples: ele nunca deve retornar nulo.
Portanto, se a chamada para findByUsername() retornar nulo, o lambda lançará uma UsernameNotFoundException (que é definida pelo Spring Security). Caso contrário, o
Usuário que foi encontrado será retornado.
Agora que você tem um serviço de detalhes de usuário personalizado que lê informações do usuário por meio de um repositório JPA, você só precisa de uma maneira de obter usuários no banco de dados em primeiro lugar. Você precisa criar uma página de registro para que os clientes do Taco Cloud se registrem no aplicativo.

REGISTRO DE USUÁRIOS
Embora o Spring Security lide com muitos aspectos da segurança, ele realmente não está diretamente envolvido no processo de registro de usuários, então você vai depender um pouco do Spring MVC para lidar com essa tarefa. A classe RegistrationController na listagem a seguir apresenta e processa formulários de registro.

Listagem 5.5 Um controlador de registro de usuário

```java
package tacos.security;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import tacos.data.UserRepository;

@Controller
@RequestMapping("/register")
public class RegistrationController {
private UserRepository userRepo;
private PasswordEncoder passwordEncoder;
public RegistrationController(
UserRepository userRepo, PasswordEncoder passwordEncoder) {
this.userRepo = userRepo;
this.passwordEncoder = passwordEncoder;
}

@GetMapping
public String registerForm() {
return "registration";
}

@PostMapping
public String processRegistration(RegistrationForm form) {
userRepo.save(form.toUser(passwordEncoder));
return "redirect:/login";
}
}
```

Como qualquer controlador Spring MVC típico, RegistrationController é anotado com @Controller para designá-lo como um controlador e marcá-lo para varredura de componentes.
Ele também é anotado com @RequestMapping para que ele manipule solicitações cujo caminho é /register.
Mais especificamente, uma solicitação GET para /register será manipulada pelo método register-Form(), que simplesmente retorna um nome de exibição lógica de registration. A listagem a seguir mostra um modelo Thymeleaf que define a exibição de registration.

Listagem 5.6 Uma visualização do formulário de registro Thymeleaf

```html
<!DOCTYPE html>
<html xmlns="http:/ /www.w3.org/1999/xhtml"
xmlns:th="http:/ /www.thymeleaf.org">
<head>
<title>Taco Cloud</title>
</head>
<body>
<h1>Register</h1>
<img th:src="@{/images/TacoCloud.png}"/>
<form method="POST" th:action="@{/register}" id="registerForm">
<label for="username">Username: </label>
<input type="text" name="username"/><br/>
<label for="password">Password: </label>
<input type="password" name="password"/><br/>
<label for="confirm">Confirm password: </label>
<input type="password" name="confirm"/><br/>
<label for="fullname">Full name: </label>
<input type="text" name="fullname"/><br/>
<label for="street">Street: </label>
<input type="text" name="street"/><br/>
<label for="city">City: </label>
<input type="text" name="city"/><br/>
<label for="state">State: </label>
<input type="text" name="state"/><br/>
<label for="zip">Zip: </label>
<input type="text" name="zip"/><br/>
<label for="phone">Phone: </label>
<input type="text" name="phone"/><br/>
<input type="submit" value="Register"/>
</form>
</body>
</html>
```

Quando o formulário é enviado, o método processRegistration() manipula a solicitação HTTPS POST. Os campos do formulário serão vinculados a um objeto RegistrationForm pelo Spring MVC e passados ​​para o método processRegistration() para processamento.
RegistrationForm é definido na seguinte classe:

```java
package tacos.security;
import org.springframework.security.crypto.password.PasswordEncoder;
import lombok.Data;
import tacos.User;
@Data
public class RegistrationForm {
private String username;
private String password;
private String fullname;
private String street;
private String city;
private String state;
private String zip;
private String phone;
public User toUser(PasswordEncoder passwordEncoder) {
return new User(
username, passwordEncoder.encode(password),
fullname, street, city, state, zip, phone);
}
}
```

Na maior parte, RegistrationForm é apenas uma classe Lombok básica com um punhado de propriedades. Mas o método toUser() usa essas propriedades para criar um novo objeto User, que é o que processRegistration() salvará, usando o UserRepository injetado.
Você sem dúvida notou que RegistrationController é injetado com um Password-Encoder. Este é exatamente o mesmo bean PasswordEncoder que você declarou anteriormente. Ao processar um envio de formulário, RegistrationController o passa para o método toUser(), que o usa para codificar a senha antes de salvá-la no banco de dados. Dessa maneira, a senha enviada é escrita em um formulário codificado, e o serviço de detalhes do usuário será capaz de autenticar contra essa senha codificada.
Agora, o aplicativo Taco Cloud tem suporte completo para registro e autenticação de usuário. Mas se você iniciá-lo neste ponto, você notará que não consegue nem chegar à
página de registro sem ser solicitado a fazer login. Isso porque, por padrão, todas as solicitações exigem autenticação. Vamos dar uma olhada em como as solicitações da web são interceptadas e protegidas para que você possa consertar essa estranha situação do ovo e da galinha.

### 5.3 Protegendo solicitações da web
Os requisitos de segurança para o Taco Cloud devem exigir que um usuário seja autenticado antes de projetar tacos ou fazer pedidos. Mas a página inicial, a página de login e a página de registro devem estar disponíveis para usuários não autenticados.
Para configurar essas regras de segurança, precisaremos declarar um SecurityFilterChain bean. O método @Bean a seguir mostra uma declaração mínima (mas não útil) do SecurityFilter-Chain bean:

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
return http.build();
}
```

O método filterChain() aceita um objeto HttpSecurity, que atua como um construtor que pode ser usado para configurar como a segurança é tratada no nível da web. Uma vez que a configuração de segurança é definida por meio do objeto HttpSecurity, uma chamada para build() criará um SecurityFilterChain que é retornado do método bean.
A seguir estão algumas das muitas coisas que você pode configurar com HttpSecurity:
- Exigir que certas condições de segurança sejam atendidas antes de permitir que uma solicitação seja atendida
- Configurar uma página de login personalizada
- Permitir que os usuários saiam do aplicativo
- Configurar proteção contra falsificação de solicitação entre sites
Interceptar solicitações para garantir que o usuário tenha autoridade adequada é uma das coisas mais comuns que você configurará o HttpSecurity para fazer. Vamos garantir que seus clientes do Taco
Cloud atendam a esses requisitos.


### 5.3.1 Protegendo solicitações
Você precisa garantir que as solicitações para /design e /orders estejam disponíveis apenas para usuários autenticados; todas as outras solicitações devem ser permitidas para todos os usuários. A configuração a seguir faz exatamente isso: 

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
return http
.authorizeRequests()
.antMatchers("/design", "/orders").hasRole("USER")
.antMatchers("/", "/**").permitAll()
.and()
.build();
}
```

A chamada para authorizeRequests() retorna um objeto (ExpressionUrlAuthorization-Configurer.ExpressionInterceptUrlRegistry) no qual você pode especificar caminhos de URL e padrões e os requisitos de segurança para esses caminhos. Neste caso, você especifica as duas regras de segurança a seguir:
- Solicitações para /design e /orders devem ser para usuários com uma autoridade concedida de ROLE_USER. Não inclua o prefixo ROLE_ em funções passadas para hasRole(); ele será assumido por hasRole().
- Todas as solicitações devem ser permitidas para todos os usuários.

A ordem dessas regras é importante. As regras de segurança declaradas primeiro têm precedência sobre aquelas declaradas abaixo. Se você trocasse a ordem dessas duas regras de segurança, todas as solicitações teriam permitAll() aplicado a elas; a regra para solicitações /design e /orders não teria efeito.
Os métodos hasRole() e permitAll() são apenas alguns dos métodos para declarar requisitos de segurança para caminhos de solicitação. A Tabela 5.1 descreve todos os
métodos disponíveis.
Tabela 5.1 Métodos de configuração para definir como um caminho deve ser protegido


| Method                     | What it does                                                                                 |
| -------------------------- | -------------------------------------------------------------------------------------------- |
| access(String)             | Permite acesso se a expressão Spring Expression Language (SpEL) for avaliada como verdadeira |
| anonymous()                | Permite acesso a usuários anônimos                                                           |
| authenticated()            | Permite acesso a usuários autenticados                                                       |
| denyAll()                  | Nega acesso incondicionalmente                                                               |
| fullyAuthenticated()       | Permite acesso se o usuário estiver totalmente autenticado (não lembrado)                    |
| hasAnyAuthority(String...) | Permite acesso se o usuário tiver qualquer uma das autoridades fornecidas                    |
| hasAnyRole(String...)      | Permite acesso se o usuário tiver qualquer uma das funções fornecidas                        |
| hasAuthority(String)       | Permite acesso se o usuário tiver a autoridade fornecida                                     |
| hasIpAddress(String)       | Permite acesso se a solicitação vier do endereço IP fornecido                                |
| hasRole(String)            | Permite acesso se o usuário tiver a função fornecida                                         |
| not()                      | Nega o efeito de qualquer um dos outros métodos de acesso                                    |
| permitAll()                | Permite acesso incondicionalmente                                                            |
| rememberMe()               | Permite acesso para usuários que são autenticados via lembre-se de mim                       |

A maioria dos métodos na tabela 5.1 fornece regras de segurança essenciais para o tratamento de solicitações, mas eles são autolimitados, habilitando regras de segurança somente conforme definido por esses métodos.
Como alternativa, você pode usar o método access() para fornecer uma expressão SpEL para declarar regras de segurança mais ricas. O Spring Security estende o SpEL para incluir vários valores e funções específicos de segurança, conforme listado na tabela 5.2.

