Um repositório comumente se refere a um local de armazenamento, geralmente considerado um local de segurança ou preservação dos itens nele armazenados. Quando você armazena algo em um repositório e depois retorna para recuperá-lo, você espera que ele esteja no mesmo estado que estava quando você o colocou lá. Em algum momento, você pode optar por remover o item armazenado do repositório. 
Esses objetos semelhantes a coleções são sobre persisência. Todo tipo Agregado persistente terá um Repositório. De um modo geral, existe uma relação um-para-um entre um tipo Agregado e um Repositório.
Um Repositório serve como uma fachada que fornece a ilusão de uma coleção em memória de todos os objetos de um determinado tipo Agregado. Ele encapsula a infraestrutura necessária para persistir e recuperar os agregados, isolando o domínio dessa complexidade.
Características importantes dos Repositórios:

1. **Interface orientada à coleção**: Oferece métodos como `add()`, `remove()`, `findById()`, e às vezes `findAll()` ou métodos de busca específicos do domínio.
2. **Relação 1:1 com Agregados**: Como você mencionou, geralmente existe um repositório para cada tipo de Agregado, não para cada Entidade.
3. **Encapsulamento da persistência**: Eles ocultam os detalhes de como os objetos são armazenados e recuperados (banco de dados, arquivo, serviço web, etc.).
4. **Manutenção da integridade**: Garantem que os agregados sejam armazenados e recuperados como unidades atômicas, preservando suas invariantes.
5. **Parte da camada de domínio**: Embora a implementação frequentemente esteja na infraestrutura, a interface do repositório pertence ao domínio.

Uma distinção importante é entre Repositórios e DAOs (Data Access Objects):

- **Repositórios** trabalham com Agregados completos e são parte do vocabulário do domínio
- **DAOs** tendem a ser mais focados em tabelas/collections e são um padrão mais técnico

No fluxo típico de uma aplicação DDD:

1. Um Repositório carrega um Agregado
2. O Agregado é modificado por um Serviço de Domínio ou diretamente pela aplicação
3. O Agregado é salvo de volta através do mesmo Repositório


```typescript
export default interface RepositoryInterface,T> {
	create(entity: T): Promise<void>;
	update(entity: T): Promise<void>;
	find(id: string) Promise<T>;
	findAll(): Promise<T[]>
}
```

