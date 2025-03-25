
- Represetational state of transfer
- Surgiu em 2000 por Roy Fielding em uma dissertação de doutorado
- Simplicidade
- Stateless
- Cacheável

### Níveis de maturidade (Richardson Maturity Model)
Níveis para criar uma API #RESTful 

- Nível 0: The swamp of POX
- Nível 1: Utilização de resources

| Verbo  | URI         | Operação |
| ------ | ----------- | -------- |
| GET    | /products/1 | BUSCAR   |
| POST   | /products   | INSERIR  |
| PUT    | /products/1 | ALTERAR  |
| DELETE | /products/1 | REMOVER  |

- Nível 2: Verbos HTTP


| Verbo  | Utilização           |
| ------ | -------------------- |
| GET    | Recuperar informação |
| POST   | Inserir              |
| PUT    | Alterar              |
| DELETE | Remover              |

- Nível 3: #HATEOAS : Hypermedia as the Engine of Application State

![[Captura de tela de 2025-03-25 08-40-29.png]]

### Uma boa API REST

- Utilizar URIs únicas para serviços e itens expostos para esses serviços
- Utiliza todos os verbos HTTP para realizar as operações em seus recursos, incluindo caching
- Provê links relacionais para os recursos exemplficando o que pode ser feito

###  HAL, Collection+JSON e Siren

- JSON não provê um padrão de hipermídia para realizar a linkagem
- HAL: Hypermedia Application Language
- Siren


Padrão HAL:

![[Captura de tela de 2025-03-25 08-46-13.png]]

### HTTP Method Negotiation
Http possui um outro método: OPTIONS. Esse método nos permite informar quais métodos são permitidos ou não em determinado recurso.

OPTIONS /api/product HTTP/1.1
Host: fullcycle.com.br

Resposta pode ser:

HTTP/1.1 200 OK
Allow: GET, POST

Caso envie a requisição em outro formato:

HTTP/1.1 405 Not Allowed
Allow: GET, POST


### Content Negotiation
O processo de content negotiation é baseado na requisição que o client está fazendo para o server. Nesse caso ele solicita o que e como ele quer a resposta. O server então retornará ou não a informação no formato desejado

Accept Negotiation
- Client solicita a informação e o tipo de retorno pelo server baseado no media type informado por ordem de prioridade.

GET / product
Accept: application/json

Resposta pode ser o retorno dos dados ou:

HTTP/1.1 406 Not Acceptable

Através de um content-type no header da request, o servidor consegue verificar se ele irá conseguir processar a informação para retornar a informação desejada.

POST / product HTTP/1.1
Accept: application/json
Content-Type: application/json

```json
{
"name": "Product 1"
}
```

Caso o servidor não aceite o content type, ele poderá retornar:

HTTP/1.1 415 Unsupported Media Type