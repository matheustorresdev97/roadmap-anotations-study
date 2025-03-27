GraphQL é uma linguagem de consulta para APIs e um runtime para executar essas consultas com seus dados existentes. Foi desenvolvido pelo Facebook em 2012 e lançado como código aberto em 2015. Diferente de REST, o GraphQL permite que os clientes solicitem exatamente os dados que precisam, evitando o problema de over-fetching ou under-fetching.

### Gerando esqueleto do servidor GraphQL
Para criar um servidor GraphQL, você precisa configurar a estrutura básica. Geralmente, isso envolve:
- Instalar as dependências necessárias (`graphql`, `apollo-server`, etc.)
- Configurar um servidor HTTP básico
- Integrar o middleware GraphQL ao servidor
- Definir a estrutura inicial para schemas e resolvers

### Criando Schema GraphQL
O schema é a parte central do GraphQL. Ele define:
- Os tipos de dados disponíveis
- As relações entre esses tipos
- As operações possíveis (queries, mutations, subscriptions)

Um schema básico pode incluir:

```graphql
type Category {
  id: ID!
  name: String!
  description: String
}

type Course {
  id: ID!
  name: String!
  description: String
  category: Category!
}

type Query {
  categories: [Category]
  category(id: ID!): Category
  courses: [Course]
  course(id: ID!): Course
}

type Mutation {
  createCategory(name: String!, description: String): Category
  createCourse(name: String!, description: String, categoryId: ID!): Course
}
```


### Gerando esqueleto de nossa aplicação

Esta fase envolve estruturar sua aplicação com:

- Organização de pastas (models, resolvers, schemas)
- Configuração de banco de dados
- Configuração da camada de apresentação/cliente

### Criando resolver para Category
Os resolvers são funções que determinam como os dados para um campo são recuperados:

```javascript
const resolvers = {
  Query: {
    categories: () => categoryDB.getAll(),
    category: (_, { id }) => categoryDB.getById(id)
  },
  Mutation: {
    createCategory: (_, { name, description }) => {
      return categoryDB.create({ name, description });
    }
  }
};
```

## Persistindo Category via Playground
O GraphQL Playground é uma interface gráfica que permite testar queries e mutations. Para persistir uma Category, você poderia executar:

```graphql
mutation {
  createCategory(name: "Backend", description: "Cursos de backend") {
    id
    name
    description
  }
}
```

### Fazendo queries de Category
Para consultar as categorias:

```graphql
query {
  categories {
    id
    name
    description
  }
}
```

Para uma categoria específica:
```graphql
query {
  category(id: "1") {
    id
    name
    description
  }
}
```

### Implementando CourseDB
CourseDB seria a camada de acesso a dados para entidades do tipo Course. Semelhante à CategoryDB, você implementaria métodos como:

- `getAll()` - Retorna todos os cursos
- `getById(id)` - Retorna um curso específico
- `create(courseData)` - Cria um novo curso
- `update(id, courseData)` - Atualiza um curso existente
- `delete(id)` - Remove um curso

### Criando resolver de CreateCourse
O resolver `createCourse` é responsável por criar novos cursos no seu sistema. Ele implementa a mutation definida no seu schema GraphQL. Veja como poderia ser implementado:
```javascript
const resolvers = {
  // ... outros resolvers
  Mutation: {
    // ... outros mutations
    createCourse: async (_, { name, description, categoryId }) => {
      // Verificar se a categoria existe
      const categoryExists = await categoryDB.getById(categoryId);
      if (!categoryExists) {
        throw new Error('Categoria não encontrada');
      }
      
      // Criar o curso
      const newCourse = {
        id: generateId(), // Função que gera um ID único
        name,
        description,
        categoryId
      };
      
      // Persistir no banco de dados
      return courseDB.create(newCourse);
    }
  }
};
```

Para usar essa mutation no GraphQL Playground:

```graphql
mutation {
  createCourse(
    name: "Introdução ao GraphQL"
    description: "Aprenda os fundamentos do GraphQL"
    categoryId: "1"
  ) {
    id
    name
    description
  }
}
```

### Implementando QueryCourses
A implementação de queries para cursos permite buscar todos os cursos ou um curso específico. Os resolvers seriam:

```javascript
const resolvers = {
  Query: {
    // ... outros resolvers
    courses: () => courseDB.getAll(),
    course: (_, { id }) => courseDB.getById(id)
  },
  // ... outros resolvers
};
```

No Playground, você poderia fazer consultas como:


```graphql
# Buscar todos os cursos
query {
  courses {
    id
    name
    description
  }
}

# Buscar um curso específico
query {
  course(id: "1") {
    id
    name
    description
  }
}
```

### Dados encadeados
Uma das grandes vantagens do GraphQL é a capacidade de encadear dados relacionados. Para implementar isso, você precisa adicionar resolvers para os campos que representam relações entre tipos.
Por exemplo, para permitir que um Course acesse sua Category:

```javascript
const resolvers = {
  // ... outros resolvers
  Course: {
    category: (parent) => {
      // parent contém os dados do curso atual
      return categoryDB.getById(parent.categoryId);
    }
  }
};
```

Isso permite consultas como:

```graphql
query {
  courses {
    id
    name
    category {
      id
      name
    }
  }
}
```

### Finalizando encadeamento de categorias

Para completar o encadeamento bidirecional, você pode implementar o resolver que permite que uma Category acesse todos os seus Courses:

```javascript
const resolvers = {
  // ... outros resolvers
  Category: {
    courses: (parent) => {
      // Buscar todos os cursos que pertencem a esta categoria
      return courseDB.getAllByCategoryId(parent.id);
    }
  }
};
```

Isso permite consultas como:

```graphql
query {
  categories {
    id
    name
    courses {
      id
      name
      description
    }
  }
}
```

O método `getAllByCategoryId` precisa ser implementado na sua classe CourseDB:

```javascript
class CourseDB {
  // ... outros métodos
  
  getAllByCategoryId(categoryId) {
    // Buscar no banco de dados todos os cursos com o categoryId específico
    return this.courses.filter(course => course.categoryId === categoryId);
  }
}
```

Após implementar esses resolvers, você terá um sistema GraphQL funcional que permite criar cursos, consultar cursos e categorias, e navegar pelas relações entre eles de forma bidirecional, aproveitando o poder do GraphQL para buscar exatamente os dados necessários em uma única requisição.