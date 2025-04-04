### Introdução aos objetos de valor
Os objetos de valor (Value Objects) são um conceito fundamental em Domain-Driven Design (DDD) que representam elementos cujo valor é o que importa, não sua identidade.

Quando você se preocupa apenas com os atributos de um elemento de um model, classifique isso como um Value Object
"Trate o Value Object como imutável"

 Características principais:

1. **Imutabilidade** - Uma vez criados, não devem ser modificados. Qualquer "alteração" deve criar um novo objeto.
2. **Igualdade por valor** - Dois objetos de valor são iguais se todos os seus atributos forem iguais, não por referência de memória.
3. **Ausência de identidade** - Diferente de entidades, não possuem um identificador único.
4. **Auto-validação** - Validam sua própria consistência durante a criação.

 Exemplo: Address (Endereço)
Um endereço é um perfeito exemplo de objeto de valor:

```java
public final class Address {
    private final String street;
    private final String city;
    private final String state;
    private final String zipCode;
    
    public Address(String street, String city, String state, String zipCode) {
        // Validação
        if (street == null || street.trim().isEmpty()) {
            throw new IllegalArgumentException("Street cannot be empty");
        }
        // Validações similares para outros campos...
        
        this.street = street;
        this.city = city;
        this.state = state;
        this.zipCode = zipCode;
    }
    
    // Apenas getters, sem setters (imutabilidade)
    public String getStreet() { return street; }
    public String getCity() { return city; }
    public String getState() { return state; }
    public String getZipCode() { return zipCode; }
    
    // Para criar um "novo" endereço com rua diferente
    public Address withStreet(String newStreet) {
        return new Address(newStreet, this.city, this.state, this.zipCode);
    }
    
    // Implementação correta de equals() e hashCode()
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        
        Address address = (Address) o;
        return street.equals(address.street) &&
               city.equals(address.city) &&
               state.equals(address.state) &&
               zipCode.equals(address.zipCode);
    }
    
    @Override
    public int hashCode() {
        // Implementação de hashCode considerando todos os campos
    }
}
```

Benefícios:

- **Segurança** - Por serem imutáveis, são thread-safe e previnem estados inconsistentes.
- **Expressividade** - Capturam conceitos do domínio de forma clara e coesa.
- **Simplicidade** - Facilitam o raciocínio sobre o código por não terem efeitos colaterais.

Use objetos de valor sempre que estiver modelando conceitos onde a identidade não importa, apenas os atributos e suas relações.



