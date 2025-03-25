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


| Expressão de segurança                                                           | O que ele avalia                                                                                                                |
| -------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| authentication                                                                   | O objeto de autenticação do usuário                                                                                             |
| denyAll                                                                          | Sempre avalia como falso                                                                                                        |
| hasAnyAuthority(String...<br>authorities)                                        | verdadeiro se o usuário recebeu alguma das autoridades fornecidas                                                               |
| hasAnyRole(String... roles)                                                      | verdadeiro se o usuário tiver qualquer uma das funções fornecidas                                                               |
| hasAuthority(String<br>authority)                                                | verdadeiro se o usuário recebeu a autoridade especificada                                                                       |
| hasPermission(Object target,<br>Object permission)                               | verdadeiro se o usuário tiver acesso ao objeto de destino especificado para a permissão fornecida                               |
| hasPermission(Serializable<br>targetId, String targetType,<br>Object permission) | verdadeiro se o usuário tiver acesso ao objeto especificado por targetId e o targetType especificado para a permissão fornecida |
| hasRole(String role)                                                             | verdadeiro se o usuário tiver a função fornecida                                                                                |
| hasIpAddress(String<br>ipAddress)                                                | verdadeiro se a solicitação vier do endereço IP fornecido                                                                       |
| isAnonymous()                                                                    | verdadeiro se o usuário for anônimo                                                                                             |
| isAuthenticated()                                                                | verdadeiro se o usuário estiver autenticado                                                                                     |
| isFullyAuthenticated()                                                           | verdadeiro se o usuário estiver totalmente autenticado (não autenticado com lembrar-de-mim                                      |
| isRememberMe()                                                                   | verdadeiro se o usuário for autenticado via lembrar-me                                                                          |
| permitAll                                                                        | Sempre avalia como verdadeiro                                                                                                   |
| principal                                                                        | O objeto principal do usuário                                                                                                   |

Como você pode ver, a maioria das extensões de expressão de segurança na tabela 5.2 correspondem a métodos semelhantes na tabela 5.1. Na verdade, usando o método access() junto com as expressões has-Role() e permitAll, você pode reescrever a configuração SecurityFilterChain da seguinte forma.

Listagem 5.7 Usando expressões Spring para definir regras de autorização

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
return http
.authorizeRequests()
.antMatchers("/design", "/orders").access("hasRole('USER')")
.antMatchers("/", "/**").access("permitAll()")
.and()
.build();
}
```

Isso pode não parecer grande coisa a princípio. Afinal, essas expressões apenas espelham o que você já fez com chamadas de método. Mas as expressões podem ser muito mais flexíveis. Por exemplo, suponha que (por algum motivo maluco) você queira permitir que apenas usuários com autoridade ROLE_USER criem novos tacos às terças-feiras (por exemplo, na terça-feira de tacos); você pode reescrever a expressão conforme mostrado nesta versão modificada do método do bean SecurityFilterChain:

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
return http
.authorizeRequests()
.antMatchers("/design", "/orders")
.access("hasRole('USER') && " +
"T(java.util.Calendar).getInstance().get("+
"T(java.util.Calendar).DAY_OF_WEEK) == " +
"T(java.util.Calendar).TUESDAY")
.antMatchers("/", "/**").access("permitAll")
.and()
.build();
}
```

Com restrições de segurança SpEL, as possibilidades são virtualmente infinitas. Aposto que você já está sonhando com restrições de segurança interessantes baseadas em SpEL. As necessidades de autorização para o aplicativo Taco Cloud são atendidas pelo uso simples de access() e das expressões SpEL. Agora vamos ver como personalizar a página de login para se ajustar à aparência do aplicativo Taco Cloud.

  
### 5.3.2 Criando uma página de login personalizada

A página de login padrão é muito melhor do que a caixa de diálogo básica HTTP desajeitada com a qual você começou, mas ainda é bastante simples e não combina muito com a aparência do resto do aplicativo Taco Cloud.
Para substituir a página de login integrada, primeiro você precisa informar ao Spring Security em qual caminho sua página de login personalizada estará. Isso pode ser feito chamando formLogin() no objeto HttpSecurity, conforme mostrado a seguir:

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
return http
.authorizeRequests()
.antMatchers("/design", "/orders").access("hasRole('USER')")
.antMatchers("/", "/**").access("permitAll()")
.and()
.formLogin()
.loginPage("/login")
.and()
.build();
}
```

Observe que antes de chamar formLogin(), você faz a ponte entre esta seção de configuração e a seção anterior com uma chamada para and(). O método and() significa que você terminou a configuração de autorização e está pronto para aplicar alguma configuração HTTP adicional. Você usará and() várias vezes ao começar novas seções de configuração.
Após a ponte, você chama formLogin() para começar a configurar seu formulário de login personalizado. A chamada para loginPage() depois disso designa o caminho onde sua página de login personalizada será fornecida. Quando o Spring Security determina que o usuário não está autenticado e precisa fazer login, ele o redirecionará para este caminho.
Agora você precisa fornecer um controlador que manipule solicitações naquele caminho. Como sua página de login será bastante simples — nada além de uma visualização — é fácil declará-la como um controlador de visualização no WebConfig. O método addViewControllers() a seguir
configura o controlador de visualização da página de login junto com o controlador de visualização que mapeia “/” para o controlador home:

```java
@Override
public void addViewControllers(ViewControllerRegistry registry) {
registry.addViewController("/").setViewName("home");
registry.addViewController("/login");
}
```

Por fim, você precisa definir a própria visualização da página de login. Como você está usando o Thymeleaf como seu mecanismo de template, o seguinte template do Thymeleaf deve funcionar bem:

```html
<!DOCTYPE html>
<html xmlns="http:/ /www.w3.org/1999/xhtml"
xmlns:th="http:/ /www.thymeleaf.org">
<head>
<title>Taco Cloud</title>
</head>
<body>
<h1>Login</h1>
<img th:src="@{/images/TacoCloud.png}"/>
<div th:if="${error}">
Unable to login. Check your username and password.
</div>
<p>New here? Click
<a th:href="@{/register}">here</a> to register.</p>
<form method="POST" th:action="@{/login}" id="loginForm">
<label for="username">Username: </label>
<input type="text" name="username" id="username" /><br/>
<label for="password">Password: </label>
<input type="password" name="password" id="password" /><br/>
<input type="submit" value="Login"/>
</form>
</body>
</html>
```

As principais coisas a serem observadas sobre esta página de login são o caminho para o qual ela publica e os nomes dos campos de nome de usuário e senha. Por padrão, o Spring Security escuta solicitações de login em /login e espera que os campos de nome de usuário e senha sejam nomeados username e password. No entanto, isso é configurável. Por exemplo, a seguinte configuração personaliza os nomes de caminho e campo:

```java
.and()
.formLogin()
.loginPage("/login")
.loginProcessingUrl("/authenticate")
.usernameParameter("user")
.passwordParameter("pwd")
```

Aqui, você especifica que o Spring Security deve escutar solicitações para /authenticate para lidar com envios de login. Além disso, os campos username e password agora devem ser nomeados user e pwd. Por padrão, um login bem-sucedido levará o usuário diretamente para a página para a qual ele estava navegando quando o Spring Security determinou que ele precisava fazer login. Se o usuário fosse navegar diretamente para a página de login, um login bem-sucedido o levaria para o caminho raiz (por exemplo, a página inicial). Mas você pode mudar isso especificando uma página de sucesso padrão, conforme mostrado a seguir:

```java
.and()
.formLogin()
.loginPage("/login")
.defaultSuccessUrl("/design")
```

Conforme configurado aqui, se o usuário fizer login com sucesso após ir diretamente para a página de login, ele será direcionado para a página /design.
Opcionalmente, você pode forçar o usuário para a página de design após o login, mesmo se ele estiver navegando em outro lugar antes de fazer login, passando true como um segundo parâmetro para defaultSuccessUrl da seguinte forma:

```java
.and()
.formLogin()
.loginPage("/login")
.defaultSuccessUrl("/design", true)
```

Entrar com um nome de usuário e senha é a maneira mais comum de autenticar em um aplicativo da web. Mas vamos dar uma olhada em outra maneira de autenticar um usuário que usa a página de login de outra pessoa.

### 5.3.3 Habilitando autenticação de terceiros

Você pode ter visto links ou botões no seu site favorito que dizem “Entrar com o Facebook”, “Entrar com o Twitter” ou algo semelhante. Em vez de pedir para um usuário inserir suas credenciais em uma página de login específica para o site, eles oferecem uma maneira de entrar por meio de outro site como o Facebook no qual ele já pode estar conectado. Esse tipo de autenticação é baseado em OAuth2 ou OpenID Connect (OIDC).
Embora OAuth2 seja uma especificação de autorização, e falaremos mais sobre como usá-lo para proteger APIs REST no capítulo 8, ele também pode ser usado para executar autenticação por meio de um site de terceiros. OpenID Connect é outra especificação de segurança que é baseada em OAuth2 para formalizar a interação que ocorre durante uma autenticação de terceiros.
Para empregar esse tipo de autenticação em seu aplicativo Spring, você precisará adicionar o iniciador do cliente OAuth2 à compilação da seguinte forma:

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

Então, no mínimo, você precisará configurar detalhes sobre um ou mais servidores OAuth2 ou OpenID Connect nos quais você deseja poder se autenticar. O Spring
Security oferece suporte para login com Facebook, Google, GitHub e Okta prontos para uso, mas você pode configurar outros clientes especificando algumas propriedades extras. O conjunto geral de propriedades que você precisará definir para que seu aplicativo atue como um cliente OAuth2/OpenID Connect é o seguinte:

```yaml
spring:
	security:
		oauth2:
			client:
				registration:
				<oauth2 or openid provider name>:
				clientId: <client id>
				clientSecret: <client secret>
				scope: <comma-separated list of requested scopes>
```


Por exemplo, suponha que para o Taco Cloud, queremos que os usuários possam fazer login usando o Facebook. A seguinte configuração em application.yml configurará o cliente OAuth2:

```yaml
spring:
security:
oauth2:
client:
registration:
facebook:
clientId: <facebook client id>
clientSecret: <facebook client secret>
scope: email, public_profile
```

O ID do cliente e o segredo são as credenciais que identificam seu aplicativo no Facebook. Você pode obter um ID do cliente e um segredo criando uma nova entrada de aplicativo em https://developers.facebook.com/. A propriedade scope especifica o acesso que o aplicativo receberá. Nesse caso, o aplicativo terá acesso ao
endereço de e-mail do usuário e às informações essenciais do seu perfil público do Facebook. Em um aplicativo muito simples, isso é tudo o que você precisa. Quando o usuário tenta acessar uma página que requer autenticação, seu navegador redirecionará para o Facebook. Se eles ainda não estiverem conectados ao Facebook, serão recebidos com a página de login do Facebook. Depois de fazer login no Facebook, eles serão solicitados a autorizar seu aplicativo e conceder o escopo solicitado. Finalmente, eles serão redirecionados de volta para seu aplicativo, onde terão sido autenticados.
Se, no entanto, você personalizou a segurança declarando um bean SecurityFilterChain, então você precisará habilitar o login #OAuth2 junto com o restante da configuração de segurança da seguinte forma:

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
return http
.authorizeRequests()
.mvcMatchers("/design", "/orders").hasRole("USER")
.anyRequest().permitAll()
.and()
.formLogin()
.loginPage("/login")
.and()
.oauth2Login()
...
.and()
.build();
}
```

Você também pode querer oferecer um login tradicional de nome de usuário-senha e um login de terceiros.
Nesse caso, você pode especificar a página de login na configuração assim:

```java
.and()
.oauth2Login()
.loginPage("/login")
```

Isso fará com que o aplicativo sempre leve o usuário para a página de login fornecida pelo aplicativo, onde ele pode escolher fazer login com seu nome de usuário e senha
como de costume. Mas você também pode fornecer um link na mesma página de login que oferece a eles a oportunidade de fazer login com o Facebook. Esse link pode ter a seguinte aparência no modelo HTML da página de login:

```html
<a th:href="/oauth2/authorization/facebook">Sign in with Facebook</a>
```

Agora que você lidou com o login, vamos virar para o outro lado da moeda da autenticação e ver como você pode habilitar um usuário a efetuar logout. Tão importante quanto efetuar login em um aplicativo é efetuar logout. Para habilitar o logout, você simplesmente precisa chamar logout no objeto HttpSecurity da seguinte forma:

```java
.and()
.logout()
```

Isso configura um filtro de segurança que intercepta solicitações POST para /logout. Portanto, para fornecer capacidade de logout, você só precisa adicionar um formulário e um botão de logout às visualizações em seu aplicativo, conforme mostrado a seguir:

```html
<form method="POST" th:action="@{/logout}">
<input type="submit" value="Logout"/>
</form>
```

Quando o usuário clica no botão, sua sessão será limpa e ele será desconectado do aplicativo. Por padrão, ele será redirecionado para a página de login, onde poderá
fazer login novamente. Mas se você preferir que ele seja enviado para uma página diferente, você pode chamar logoutSuccessUrl() para especificar uma landing page pós-logout diferente, como mostrado aqui:

```java
.and()
.logout()
.logoutSuccessUrl("/")
```

Nesse caso, os usuários serão enviados para a página inicial após o logout.

### 5.3.4 Prevenção de falsificação de solicitação entre sites

A falsificação de solicitação entre sites (CSRF) é um ataque de segurança comum. Envolve submeter um usuário a um código em uma página da web projetada de forma maliciosa que automaticamente (e geralmente secretamente) envia um formulário para outro aplicativo em nome de um usuário que geralmente é a
vítima do ataque. Por exemplo, um usuário pode receber um formulário no site de um invasor que publica automaticamente em uma URL no site bancário do usuário
(que provavelmente é mal projetado e vulnerável a esse tipo de ataque) para transferir dinheiro. O usuário pode nem saber que o ataque aconteceu até perceber
dinheiro faltando em sua conta.
Para se proteger contra esses ataques, os aplicativos podem gerar um token #CSRF ao exibir um formulário, colocar esse token em um campo oculto e armazená-lo para uso posterior no servidor. Quando o formulário é enviado, o token é enviado de volta ao servidor junto com o resto dos dados do formulário. A solicitação é então interceptada pelo servidor e comparada com o token que foi gerado originalmente. Se o token corresponder, a solicitação é permitida a prosseguir. Caso contrário, o formulário deve ter sido renderizado por um site maligno sem conhecimento do token gerado pelo servidor.
Felizmente, o Spring Security tem proteção CSRF integrada. Ainda mais afortunado é que ele é habilitado por padrão e você não precisa configurá-lo explicitamente. Você só precisa se certificar de que todos os formulários enviados pelo seu aplicativo incluam um campo chamado _csrf que contenha o token CSRF._
O Spring Security até facilita isso colocando o token CSRF em um atributo de solicitação com o nome _csrf. Portanto, você pode renderizar o token CSRF em um_ 
campo oculto com o seguinte em um modelo Thymeleaf:

```html
<input type="hidden" name="_csrf" th:value="${_csrf.token}"/>
```

Se você estiver usando a biblioteca de tags JSP do Spring MVC ou o Thymeleaf com o dialeto Spring Security, você nem precisa se preocupar em incluir explicitamente um campo oculto. O campo oculto será

renderizado automaticamente para você.
No Thymeleaf, você só precisa ter certeza de que um dos atributos do elemento < form >
é prefixado como um atributo do Thymeleaf. Isso geralmente não é uma preocupação, porque é
bastante comum deixar o Thymeleaf renderizar o caminho como relativo ao contexto. Por exemplo, o
atributo th:action mostrado a seguir é tudo o que você precisa para o Thymeleaf renderizar o
campo oculto para você:

```html
<form method="POST" th:action="@{/login}" id="loginForm">
```

É possível desabilitar o suporte a CSRF, mas estou hesitante em mostrar como. A proteção CSRF é importante e facilmente manipulada em formulários, então há pouca razão para desabilitá-la. Mas

se você insistir em desabilitá-la, você pode fazer isso chamando disable() assim:

```java
.and()
.csrf()
.disable()
```

Novamente, eu o alerto para não desabilitar a proteção CSRF, especialmente para aplicações
de produção.
Toda a sua segurança da camada web agora está configurada para o Taco Cloud. Entre outras
coisas, agora você tem uma página de login personalizada e a capacidade de autenticar usuários em

um repositório de usuários JPA. Agora vamos ver como você pode obter informações sobre o usuário
logado.

### 5.4 Aplicando segurança em nível de método

Embora seja fácil pensar sobre segurança no nível de solicitação da web, nem sempre é onde as restrições de segurança são melhor aplicadas. Às vezes, é melhor verificar se o usuário está autenticado e recebeu autoridade adequada no ponto em que a ação protegida será executada.
Por exemplo, digamos que, para fins administrativos, temos uma classe de serviço que inclui um método para limpar todos os pedidos do banco de dados. Usando um
OrderRepository injetado, esse método pode ser parecido com isto:

```java
public void deleteAllOrders() {
orderRepository.deleteAll();
}
```

Agora, suponha que temos um controlador que chama o método deleteAllOrders() como o resultado de uma solicitação POST, conforme mostrado aqui:

```java
@Controller
@RequestMapping("/admin")
public class AdminController {
	private OrderAdminService adminService;

	public AdminController(OrderAdminService adminService) {
		this.adminService = adminService;
	}
	
	@PostMapping("/deleteOrders")
	public String deleteAllOrders() {
	adminService.deleteAllOrders();
	return "redirect:/admin";
	}
}
```

Seria fácil ajustar o SecurityConfig da seguinte forma para garantir que apenas usuários autorizados tenham permissão para executar essa solicitação POST:

```java
.authorizeRequests()
...
.antMatchers(HttpMethod.POST, "/admin/**")
.access("hasRole('ADMIN')")
....
```

Isso é ótimo e evitaria que qualquer usuário não autorizado fizesse uma solicitação POST para /admin/deleteOrders que resultaria no desaparecimento de todos os pedidos do banco de dados. Mas suponha que algum outro método do controlador também chame deleteAllOrders().
Você precisaria adicionar mais matchers para proteger as solicitações para os outros controladores que precisarão ser protegidos.
Em vez disso, podemos aplicar a segurança diretamente no método deleteAllOrders() assim:

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteAllOrders() {
orderRepository.deleteAll();
}
```

A anotação @PreAuthorize pega uma expressão SpEL e, se a expressão for avaliada como falsa, o método não será invocado. Por outro lado, se a expressão for avaliada como verdadeira, o método será permitido. Neste caso, @PreAuthorize está verificando se o usuário tem o privilégio ROLE_ADMIN. Se sim, o método será chamado e todos os pedidos serão excluídos. Caso contrário, ele será interrompido.
No caso de @PreAuthorize bloquear a chamada, a Access-DeniedException do Spring Security será lançada. Esta é uma exceção não verificada, então você não precisa para pegá-lo, a menos que você queira aplicar algum comportamento personalizado em torno do tratamento de exceções. Se não for capturado, ele irá aparecer e eventualmente será capturado pelos filtros do Spring Security e tratado adequadamente, seja com uma página HTTP 403 ou talvez redirecionando para a página de login se o usuário não estiver autenticado.
Para que @PreAuthorize funcione, você precisará habilitar a segurança do método global. Para isso, você precisará anotar a classe de configuração de segurança com @EnableGlobalMethodSecurity da seguinte forma:

```java
@Configuration
@EnableGlobalMethodSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
...
}
```

Você descobrirá que @PreAuthorize é uma anotação útil para a maioria das necessidades de segurança de nível de método. Mas saiba que ela tem uma contraparte pós-invocação um pouco menos útil em @PostAuthorize. A anotação @PostAuthorize funciona quase da mesma forma que a anotação @PreAuthorize, exceto que sua expressão não será avaliada até depois que o método de destino seja invocado e retorne. Isso permite que a expressão considere o valor de retorno do método ao decidir se deve permitir a invocação do método.
Por exemplo, suponha que temos um método que busca um pedido por seu ID. Se você quiser restringi-lo de ser usado, exceto por administradores ou pelo usuário a quem o pedido pertence, você pode usar @PostAuthorize assim:

```java
@PostAuthorize("hasRole('ADMIN') || " +
"returnObject.user.username == authentication.name")
public TacoOrder getOrder(long id) {
...
}
```

Neste caso, o returnObject no TacoOrder retornou do método. Se sua propriedade user tiver um username que seja igual à propriedade name da autenticação, então ele
será permitido. Para saber disso, no entanto, o método precisará ser executado para que ele possa retornar o objeto TacoOrder para consideração. Mas espere! Como você pode proteger um método de ser invocado se a condição para aplicar a segurança depende do valor de retorno da invocação do método? Esse enigma do ovo e da galinha é resolvido permitindo que o método seja invocado, então lançando uma AccessDeniedException se a expressão retornar false.

### 5.5 Conhecendo seu usuário

Muitas vezes, não basta saber simplesmente que o usuário fez login e quais permissões ele recebeu. Geralmente é importante saber também quem ele é, para que você possa personalizar sua experiência.
Por exemplo, no OrderController, quando você cria inicialmente o objeto TacoOrder que está vinculado ao formulário de pedido, seria bom se você pudesse preencher previamente o TacoOrder com o nome e endereço do usuário, para que ele não precise inseri-lo novamente para cada pedido. Talvez ainda mais importante, quando você salva o pedido, você deve associar a entidade TacoOrder ao Usuário que criou o pedido.
Para obter a conexão desejada entre uma entidade TacoOrder e uma entidade Usuário, você precisa adicionar a seguinte nova propriedade à classe TacoOrder:

```java
@Data
@Entity
@Table(name="Taco_Order")
public class TacoOrder implements Serializable {
...
@ManyToOne
private User user;
...
}
```

A anotação @ManyToOne nesta propriedade indica que um pedido pertence a um único usuário e, inversamente, que um usuário pode ter muitos pedidos. (Como você está usando Lombok, não precisará definir explicitamente métodos de acesso para a propriedade.)
No OrderController, o método processOrder() é responsável por salvar um pedido. Ele precisará ser modificado para determinar quem é o usuário autenticado e para
chamar setUser() no objeto TacoOrder para conectar o pedido ao usuário.
Temos várias maneiras de determinar quem é o usuário. Algumas das maneiras mais comuns seguem:
- Injete um objeto java.security.Principal no método do controlador.
- Injete um objeto org.springframework.security.core.Authentication no método do controlador.
- Use org.springframework.security.core.context.SecurityContextHolder para obter o contexto de segurança.
- Injete um parâmetro de método anotado @AuthenticationPrincipal. (@AuthenticationPrincipal é do pacote org.springframework.security.core.annotation do Spring Security.)
Por exemplo, você pode modificar processOrder() para aceitar um java.security.Principal como um parâmetro. Você pode então usar o nome principal para procurar o usuário em um UserRepository como segue:

```java
@PostMapping
public String processOrder(@Valid TacoOrder order, Errors errors,
SessionStatus sessionStatus,
Principal principal) {
User user = userRepository.findByUsername(
principal.getName());
order.setUser(user);
...
}
```

Isso funciona bem, mas ele espalha código que não é relacionado à segurança com código de segurança. Você pode reduzir parte do código específico de segurança modificando processOrder() para aceitar um objeto Authentication como parâmetro em vez de um Principal, conforme mostrado a seguir:

```java
@PostMapping
public String processOrder(@Valid TacoOrder order, Errors errors,
SessionStatus sessionStatus,
Authentication authentication) {
...
User user = (User) authentication.getPrincipal();
order.setUser(user);
...
}
```

Com a Autenticação em mãos, você pode chamar getPrincipal() para obter o objeto principal que, neste caso, é um Usuário. Observe que getPrincipal() retorna um java.util.Object, então você precisa convertê-lo para Usuário. Talvez a solução mais limpa de todas, no entanto, seja simplesmente aceitar um objeto Usuário em processOrder(), mas anotá-lo com @AuthenticationPrincipal para que ele seja o principal da autenticação, como segue:

```java
@PostMapping
public String processOrder(@Valid TacoOrder order, Errors errors,
SessionStatus sessionStatus,
@AuthenticationPrincipal User user) {
if (errors.hasErrors()) {
return "orderForm";
}
order.setUser(user);
orderRepo.save(order);
sessionStatus.setComplete();
return "redirect:/";
}
```

O que é legal sobre @AuthenticationPrincipal é que ele não requer um cast (como com Authentication), e limita o código específico de segurança à própria anotação. Quando
você obtém o objeto User em processOrder(), ele está pronto para ser usado e atribuído ao TacoOrder.
Há uma outra maneira de identificar quem é o usuário autenticado, embora seja um pouco confuso no sentido de que é muito pesado com código específico de segurança. Você pode obter um objeto Authentication do contexto de segurança e então solicitar seu principal assim:

```java
Authentication authentication =
SecurityContextHolder.getContext().getAuthentication();
User user = (User) authentication.getPrincipal();
```

Embora esse snippet seja espesso com código específico de segurança, ele tem uma vantagem sobre as outras abordagens descritas: ele pode ser usado em qualquer lugar do aplicativo, não apenas em métodos de manipulador do controlador. Isso o torna adequado para uso em níveis mais baixos do código.

Resumo
- A autoconfiguração do Spring Security é uma ótima maneira de começar com a segurança, mas a maioria dos aplicativos precisará configurar explicitamente a segurança para atender aos seus requisitos de segurança exclusivos.
- Os detalhes do usuário podem ser gerenciados em armazenamentos de usuários apoiados por bancos de dados relacionais, LDAP ou implementações completamente personalizadas.
- O Spring Security protege automaticamente contra ataques CSRF.
- Informações sobre o usuário autenticado podem ser obtidas por meio do objeto SecurityContext (retornado de SecurityContextHolder.getContext()) ou injetadas em controladores usando @AuthenticationPrincipal.