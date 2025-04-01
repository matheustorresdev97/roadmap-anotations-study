#SOLID é um acrônimo que consolida 5 itens que são considerados como boas práticas no mundo do desenvolvimento orientado a objetos. 

- SRP = Single Responsibilty Principle
- OCP = Open-closed Principle
- LSP = Liskov Substitution Principle
- ISP = Interface Segregation Principle
- DIP = Dependency Inversion Principle


###  S - Princípio da Responsabilidade Única (Single Responsibility Principle)

> "Uma classe deve ter apenas um motivo para mudar."

Este princípio estabelece que uma classe deve ser responsável por apenas uma coisa. Se uma classe tem múltiplas responsabilidades, ela se torna acoplada. Uma mudança em uma responsabilidade pode afetar ou quebrar outras partes da classe.

Exemplo incorreto:

```java
class Usuário {
    private String nome;
    private String email;
    
    // Getters e setters
    
    public void salvarNoBancoDeDados() {
        // Código para salvar usuário no banco de dados
    }
    
    public void enviarEmail() {
        // Código para enviar email para o usuário
    }
    
    public String gerarRelatório() {
        // Código para gerar um relatório sobre o usuário
    }
}
```

Exemplo correto:

```java
class Usuário {
    private String nome;
    private String email;
    
    // Getters e setters
}

class RepositórioUsuário {
    public void salvar(Usuário usuário) {
        // Código para salvar usuário no banco de dados
    }
}

class ServiçoEmail {
    public void enviarEmail(Usuário usuário, String mensagem) {
        // Código para enviar email
    }
}

class GeradorRelatório {
    public String gerarRelatórioUsuário(Usuário usuário) {
        // Código para gerar relatório
    }
}
```

Neste exemplo correto, separamos as diferentes responsabilidades em classes distintas. A classe `Usuário` agora só mantém dados, enquanto outras classes lidam com persistência, comunicação e relatórios.

###  O - Princípio Aberto/Fechado (Open/Closed Principle)

> "Entidades de software (classes, módulos, funções, etc.) devem ser abertas para extensão, mas fechadas para modificação."

Este princípio sugere que o código deve ser escrito de forma que você possa adicionar novas funcionalidades sem alterar o código existente.

Exemplo incorreto:

```java
class CalculadoraÁrea {
    public double calcularÁrea(Object forma) {
        if (forma instanceof Retângulo) {
            Retângulo r = (Retângulo) forma;
            return r.largura * r.altura;
        } 
        else if (forma instanceof Círculo) {
            Círculo c = (Círculo) forma;
            return Math.PI * c.raio * c.raio;
        }
        return 0;
    }
}
```

Exemplo correto:

```java
interface Forma {
    double calcularÁrea();
}

class Retângulo implements Forma {
    public double largura;
    public double altura;
    
    @Override
    public double calcularÁrea() {
        return largura * altura;
    }
}

class Círculo implements Forma {
    public double raio;
    
    @Override
    public double calcularÁrea() {
        return Math.PI * raio * raio;
    }
}

class CalculadoraÁrea {
    public double calcularÁrea(Forma forma) {
        return forma.calcularÁrea();
    }
}
```

No exemplo correto, podemos adicionar novas formas (como Triângulo, Trapézio) sem modificar a classe `CalculadoraÁrea`. Cada nova forma implementa a interface `Forma` e fornece sua própria implementação do método `calcularÁrea()`.


###  L - Princípio da Substituição de Liskov (Liskov Substitution Principle)

> "Objetos de uma classe derivada devem poder substituir objetos da classe base sem afetar a corretude do programa."

Este princípio afirma que as classes derivadas devem ser substituíveis por suas classes base. Isso significa que um objeto de uma classe derivada deve comportar-se de maneira que não cause problemas quando usado em vez de um objeto da classe base.

Exemplo incorreto:

```java
class Retângulo {
    protected int largura;
    protected int altura;
    
    public void setLargura(int largura) {
        this.largura = largura;
    }
    
    public void setAltura(int altura) {
        this.altura = altura;
    }
    
    public int getÁrea() {
        return largura * altura;
    }
}

class Quadrado extends Retângulo {
    @Override
    public void setLargura(int largura) {
        this.largura = largura;
        this.altura = largura;  // Um quadrado tem lados iguais
    }
    
    @Override
    public void setAltura(int altura) {
        this.altura = altura;
        this.largura = altura;  // Um quadrado tem lados iguais
    }
}
```

O problema aqui é que se tivermos uma função que trabalha com `Retângulo`:

```java
void verificarÁrea(Retângulo r) {
    r.setLargura(5);
    r.setAltura(4);
    assert r.getÁrea() == 20;  // Esta asserção falhará se r for um Quadrado
}
```

Exemplo correto:

```java
interface FormaQuadrilátera {
    int getÁrea();
}

class Retângulo implements FormaQuadrilátera {
    private int largura;
    private int altura;
    
    public Retângulo(int largura, int altura) {
        this.largura = largura;
        this.altura = altura;
    }
    
    public int getÁrea() {
        return largura * altura;
    }
}

class Quadrado implements FormaQuadrilátera {
    private int lado;
    
    public Quadrado(int lado) {
        this.lado = lado;
    }
    
    public int getÁrea() {
        return lado * lado;
    }
}
```

Neste caso, não há herança entre Quadrado e Retângulo, evitando o problema de substituição.


### I - Princípio da Segregação de Interface (Interface Segregation Principle)

> "Clientes não devem ser forçados a depender de interfaces que não utilizam."


Este princípio sugere que é melhor ter várias interfaces específicas do que uma interface geral.

Exemplo incorreto:

```java
interface Trabalhador {
    void trabalhar();
    void comer();
    void dormir();
}

class Humano implements Trabalhador {
    public void trabalhar() { /* implementação */ }
    public void comer() { /* implementação */ }
    public void dormir() { /* implementação */ }
}

class Robô implements Trabalhador {
    public void trabalhar() { /* implementação */ }
    public void comer() {
        // Robôs não comem, mas são forçados a implementar este método
        throw new UnsupportedOperationException();
    }
    public void dormir() {
        // Robôs não dormem, mas são forçados a implementar este método
        throw new UnsupportedOperationException();
    }
}
```

Exemplo correto:

```java
interface Trabalhável {
    void trabalhar();
}

interface Comível {
    void comer();
}

interface Dormível {
    void dormir();
}

class Humano implements Trabalhável, Comível, Dormível {
    public void trabalhar() { /* implementação */ }
    public void comer() { /* implementação */ }
    public void dormir() { /* implementação */ }
}

class Robô implements Trabalhável {
    public void trabalhar() { /* implementação */ }
    // Não precisa implementar métodos que não usa
}
```

No exemplo correto, separamos a interface grande em interfaces menores e mais específicas. Dessa forma, o `Robô` só precisa implementar o que realmente faz sentido para ele.


### D - Princípio da Inversão de Dependência (Dependency Inversion Principle)

> "Módulos de alto nível não devem depender de módulos de baixo nível. Ambos devem depender de abstrações. Abstrações não devem depender de detalhes. Detalhes devem depender de abstrações."


Este princípio sugere que devemos depender de abstrações, não de implementações concretas.

Exemplo incorreto:

```java
class ServiçoNotificação {
    private EmailSender emailSender;
    
    public ServiçoNotificação() {
        this.emailSender = new EmailSender();
    }
    
    public void notificar(String mensagem) {
        this.emailSender.enviarEmail(mensagem);
    }
}

class EmailSender {
    public void enviarEmail(String mensagem) {
        // Lógica para enviar email
    }
}
```

Exemplo correto:

```java
interface EnviadorMensagem {
    void enviar(String mensagem);
}

class EmailSender implements EnviadorMensagem {
    public void enviar(String mensagem) {
        // Lógica para enviar email
    }
}

class SMSSender implements EnviadorMensagem {
    public void enviar(String mensagem) {
        // Lógica para enviar SMS
    }
}

class ServiçoNotificação {
    private EnviadorMensagem enviador;
    
    // Injeção de dependência
    public ServiçoNotificação(EnviadorMensagem enviador) {
        this.enviador = enviador;
    }
    
    public void notificar(String mensagem) {
        this.enviador.enviar(mensagem);
    }
}
```

No exemplo correto, `ServiçoNotificação` depende da abstração `EnviadorMensagem` e não de uma implementação concreta. Isso permite maior flexibilidade e facilita testes unitários, pois podemos passar qualquer implementação de `EnviadorMensagem` para o serviço.

## Aplicação Prática dos Princípios SOLID

Na prática, os princípios SOLID são aplicados em conjunto para criar código mais:

1. **Manutenível**: Mais fácil de entender e modificar
2. **Extensível**: Pode ser expandido com novas funcionalidades sem alterar código existente
3. **Testável**: Mais fácil de escrever testes unitários
4. **Reutilizável**: Componentes podem ser reaproveitados em diferentes partes do sistema
5. **Acoplamento reduzido**: Mudanças em uma parte do sistema afetam menos outras partes