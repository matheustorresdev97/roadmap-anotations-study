### JPA e gerenciamento de entidades
- A JPA faz o gerenciamento das entidades do sistema durante a sessão JPA.
- Uma sessão JPA corresponde ao contexto em que a JPA está fazendo operações com entidades durante uma conexão com o banco de dados.
	- EntityManager: objeto da JPA que encapsula uma conexão com o banco de dados e que gerencia as entidades durante uma sessão JPA.



![[Captura de tela de 2025-04-11 10-24-13.png]]

Estados de uma entidade

![[Captura de tela de 2025-04-11 10-24-50.png]]


Salvar entidades associadas para-um

![[Captura de tela de 2025-04-11 10-26-10.png]]

```java

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    
    @OneToMany(mappedBy = "department")
    private List<Person> people = new ArrayList<>();
    public List<Person> getPeople() {
        return people;
    }
```


```java
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private Double salary;

    @ManyToOne
    @JoinColumn(name = "department_id")
    private Department department;
```

Salvar entidades associadas para-muitos

![[Captura de tela de 2025-04-11 10-31-16.png]]

```java
 @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @ManyToMany(mappedBy = "categories")
    private Set<Product> products = new HashSet<>();
    
    public Set<Product> getProducts() {
        return products;
    }

```

```java
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private Double price;

    @ManyToMany
    @JoinTable(name = "tb_product_category",
            joinColumns = @JoinColumn(name = "product_id"),
            inverseJoinColumns = @JoinColumn(name = "category_id"))
    private Set<Category> categories = new HashSet<>();

    public Set<Category> getCategories() {
        return categories;
    }
```


### Evitando degradação de performance
Carregando lazy, tratativas, Transactional

Grande vilão da JPA: idas e vindas desnecessárias ao banco de dados.
Causa comum: entidades associadas lazy carregando sob demanda.

### Carregamento lazy de entidades associadas
Carregamento padrão de entidades associadas:
- Para-um: EAGER
- Para-muitos: LAZY

Devido ao comportamento lazy, enquanto a sessão JPA estiver ativa, o acesso a um objeto associado pode provocar uma nova consulta ao banco de dados.
Formas de alterar o comportamento padrão
- Atributo fetch no relacionamento da entidade (NÃO RECOMANDADO)
- Cláusula JOIN FETCH
- Consultas customizadas (IDEAL)

### Atributo fetch do relacionamento
Cuidado: a mudança desse atributo pode gerar efeitos indesejados! Evite esta abordagem!
Exemplo:
@ManyToOne(fetch = FetchType.LAZY)

### Cláusula JOIN FETCH
Nota: esta cláusula não funciona para buscas paginadas do Spring.

Exemplo:
```java
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
@Query("SELECT obj FROM Employee obj JOIN FETCH obj.department")
List<Employee> findEmployeesWithDepartments();
}
```

### Cache em memória das entidades
- A JPA mantém um "cache" das entidades gerenciadas na mesma sessão JPA (mapa de identidade).
- Se você trouxer entidades para a memória, a JPA não volta ao banco se vocÊ precisar novamente delas durante a mesma seção JPA.

### Transactional e open-in-view no Spring

- A annotation @Transactional assegura:
1. resolução da transação com o banco de dados.
2. resolução de todas pendências "lazy" com o banco de dados

- A propriedade spring;jpa.open-in-view=false faz com que a sessão JPA seja encerrada antes de voltar para a camada controller (camada web)

![[Captura de tela de 2025-04-11 10-41-18.png]]


### Consultas customizadas
Query methods, SQL, JPQL

### Query methods
- No JpaRepository do Spring Data JPA, é possível fazer uma consulta customizada apenas pelo nome do método
https://docs.spring.io/spring-data/jpa/reference/#repositories.query-methods

Vale a pena usar ? 
Para consultas simples: ok
Para consultas mais complexas: melhor escrever a consulta


### JPQL - linguagem de consulta de JPA
Geralmente toda ferramente de ORM possui uma linguagem e/ou ferramentes próprias para realização de consultas ao banco de dados.
A JPQL é parecida com a SQL, porém adaptada para o modelo de acesso a dados JPA.

EXEMPLO 1 :
```SQL

SELECT *
FROM tb_employee
WHERE UPPER(name) LIKE 'MARIA%'
```

```JPQL
SELECT obj
FROM Employee obj
WHERE UPPER(obj.name) LIKE 'MARIA%'
```

EXEMPLO 2:
```SQL
SELECT tb_employee.*
FROM tb_employee
INNER JOIN tb_department ON tb_department.id = tb_employee.department_id
WHERE tb_department.name = 'Financeiro'
```

```JQPL
SELECT obj
FROM Employee obj
WHERE obj.department.name = 'Financeiro'
```


vale a pena se especializar na JPQL?
Vantagens da JPQL: 
• Algumas consultas podem ficar mais simples na JPQL
• Usufrui melhor do Spring Data JPA (paginação, JpaRepository) 
• Os objetos resultantes são entidades gerenciadas pela JPA

Desvantagens da JPQL:
• Consultas complexas podem ficar difíceis de escrever e validar (mais fácil escrever e testar a consulta SQL diretamente na ferramenta de banco de dados) 
• Não tem união (JPA 2.x) 
• Curva de aprendizado para uma tecnologia específica