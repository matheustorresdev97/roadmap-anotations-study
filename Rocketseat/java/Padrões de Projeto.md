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