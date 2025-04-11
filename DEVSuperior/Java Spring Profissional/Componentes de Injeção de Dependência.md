### Sistema e componentes
- Sistema é composto de componentes
- Componentes devem ser
	- Coesos (responsabilidade clara e única)
	- Desacoplados entre si
- Objetivos
	- Flexibilidade
	- Manutenção/substituição facilitada
	- Reaproveitamento

Exemplo didático
Fazer um programa ler os dados de um funcionário (nome, salário bruto) e depois mostrar o salário líquido, que consiste no valor bruto descontado impostos e previdência.
REGRAS:
1) Imposto é 20%
2) Previdência é 10%

Nome: Matheus
Salário bruto: 4000
Salário líquido= 2800

Solução "rápida"

```java
Locale.setDefault(Locale.US);
Scanner sc = new Scanner(System.in);

System.out.print("Nome: ");
String name = sc.nextLine();
System.out.print("Salario bruto: ");

double grossSalary = sc.nextDouble();
double netSalary = grossSalary * 0.7;
System.out.printf("Salario liquido = %.2f%n", netSalary);

sc.close();
```

Solução pensando em componentes
O sistema será composto por componentes, cada um com sua responsabilidade.

![[Captura de tela de 2025-04-09 08-06-52.png]]

### Inversão de controle
Analogia do carro: 
- No carro, o motor depende da bateria
- Porém, a base de encaixe da bateria é fora do motor
- Por que não colocar a base da bateria dentro do motor?
	- Por que se for preciso trocar a bateria, não é preciso abrir o motor.

Generalizando:
- Se um componente A depende de B, A não deve ser o controle sobre esta dependência (B).
- Por que? Porque se for preciso trocar a dependência (B), seria preciso também abrir o componente A.
- É preciso "inverter o controle", ou seja, o controle de dependência tem que ficar fora do componente A.

Se um componente A depende de B, A não deve ter o controle de quem será essa dependência

![[Captura de tela de 2025-04-09 08-11-16.png]]

### Injeção de dependência
- Uma vez que um sistema usa o princípio da inversão de controle, quando um componente A depende do componente B, esta dependência (B) precisa ser "injetada" em A. 
- Em sistemas de software, essa "injeção de dependência" pode ser feita de várias formas:
	- Construtor
	- Método set
	- Container de injeção de dependência (framework)


```java
package services;

  

import entities.Employee;

  

public class SalaryService {

  

final private TaxService taxService;

final private PensionService pensionService;

  

public SalaryService(TaxService taxService, PensionService pensionService) {

this.taxService = taxService;

this.pensionService = pensionService;

}

public double netSalary(Employee employee) {

return employee.getGrossSalary() - taxService.tax(employee.getGrossSalary())

- pensionService.discount(employee.getGrossSalary());

}

package services;

  

public class PensionService {

public double discount(double amount) {

return amount * 0.1;

}

}

package services;

  

public class TaxService {

public double tax(double amount) {

return amount * 0.2;

}

}

}
```



### Framework
- Tradução ao pé da letra: estrutura, armação, estrutura de trabalho
- É um conjunto de ferramentas que oferece uma infraestrutura para se desenvolver sistemas de forma produtiva
- Um framework gerencia: 
	-  Injeção de dependências
	- Transações (back end)
	- Ciclo de vida, escopo e reaproveitamento de componentes
	- Configurações
	- Integrações
	- Outros

Dois passos para trabalhar com injeção de dependência em frameworks
1. Registrar os componentes
2. Indicar quais componentes dependem de quais

Uma vez feito isso, o próprio framework se encarregará de:
- Instanciar componentes
- Resolver dependências
- Reaproveitar componentes
- Gerenciar escopo e ciclo de vida dos componentes




```java
package com.matheustorres.aula;

  

import org.springframework.beans.factory.annotation.Autowired;

import org.springframework.boot.CommandLineRunner;

import org.springframework.boot.SpringApplication;

import org.springframework.boot.autoconfigure.SpringBootApplication;

  

import com.matheustorres.aula.entities.Employee;

import com.matheustorres.aula.services.SalaryService;

  
  

@SpringBootApplication

public class AulaApplication implements CommandLineRunner {

  

@Autowired

private SalaryService salaryService;

  
  

public static void main(String[] args) {

SpringApplication.run(AulaApplication.class, args);

}

  

public AulaApplication(SalaryService salaryService) {

this.salaryService = salaryService;

}

  
  

@Override

public void run(String[] args) {

Employee employee = new Employee("Matheus", 4000.0);

System.out.println(salaryService.netSalary(employee));

}

}
```

```java
package com.matheustorres.aula.services;

  

import org.springframework.stereotype.Service;

  

@Service

public class TaxService {

public double tax(double amount) {

return amount * 0.2;

}

}



package com.matheustorres.aula.services;

  

import org.springframework.stereotype.Service;

  

@Service

public class PensionService {

public double discount(double amount) {

return amount * 0.1;

}

}


package com.matheustorres.aula.services;

  

import org.springframework.beans.factory.annotation.Autowired;

import org.springframework.stereotype.Service;

  

import com.matheustorres.aula.entities.Employee;

  
  

@Service

public class SalaryService {

  

@Autowired

private TaxService taxService;

  

@Autowired

private PensionService pensionService;

  

public double netSalary(Employee employee) {

return employee.getGrossSalary() - taxService.tax(employee.getGrossSalary())

- pensionService.discount(employee.getGrossSalary());

}

}
```
