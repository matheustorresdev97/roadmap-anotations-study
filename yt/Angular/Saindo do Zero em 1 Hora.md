O #Angular é um framework de desenvolvimento web front-end de código aberto mantido pela Google. Ele é usado para criar aplicações web dinâmicas e de página única ( #SPA - Single Page Applications).

Principais características do Angular:

- Baseado em TypeScript, uma linguagem que adiciona tipagem estática ao JavaScript
- Arquitetura baseada em componentes que permite criar interfaces de usuário modulares e reutilizáveis
- Sistema de binding bidirecional de dados que sincroniza automaticamente os dados entre o modelo e a visão
- Injeção de dependências embutida para melhor organização e testabilidade do código
- Ferramentas robustas para roteamento, formulários, comunicação HTTP e testes
- Programação Orientada a Objetos

O Angular teve uma grande reformulação com o lançamento do Angular 2 em 2016 (o Angular original passou a ser chamado de AngularJS), e desde então segue com versões numeradas sequencialmente (Angular 4, 5, 6, etc., até as versões mais recentes). Cada versão traz melhorias de desempenho, novos recursos e aprimoramentos.

É amplamente utilizado para construir aplicações empresariais complexas e escaláveis.

### Requisitos

- NodeJS v18 ou  mais recente
- Instalação do angular/cli
```shell
npm install -g @angular/cli
```

###  Criar um novo projeto
Primeiro, verifique se você já tem o Angular CLI instalado globalmente com:
`ng version`

Se não estiver instalado, instale o Angular CLI globalmente:
`npm install -g @angular/cli`

Após a instalação do Angular CLI, crie um novo projeto com:
```Shell
ng new meu-projeto-angular
```

Durante a criação, o CLI fará algumas perguntas:

- Se deseja adicionar o Angular routing (geralmente recomendado)
- Qual formato de folha de estilo você prefere usar (CSS, SCSS, SASS, Less, etc.)

### Estrutura base do projeto

![[Captura de tela de 2025-03-22 20-45-51.png]]

Diretórios Principais

- **public**: Contém arquivos estáticos acessíveis publicamente
    - **favicon.ico**: O ícone exibido na aba do navegador
- **src**: Diretório principal com o código-fonte da aplicação
    - **app**: Contém os componentes e lógica principal

 Arquivos do Componente Principal (em /app)

- **app.component.css**: Estilos CSS específicos para o componente principal
- **app.component.html**: Template HTML do componente principal
- **app.component.spec.ts**: Arquivo de testes unitários para o componente principal
- **app.component.ts**: Código TypeScript do componente principal (lógica, propriedades, métodos)
- **app.config.ts**: Configurações do aplicativo Angular (providers, importações)
- **app.routes.ts**: Definições de rotas para navegação entre componentes

 Arquivos na Raiz do Projeto

- **index.html**: Página HTML principal carregada pelo navegador
- **main.ts**: Ponto de entrada da aplicação, onde o módulo raiz é inicializado
- **styles.css**: Estilos globais aplicados a toda a aplicação

 Arquivos de Configuração

- **.editorconfig**: Configurações para manter consistência de código entre editores
- **.gitignore**: Lista de arquivos/diretórios ignorados pelo Git
- **angular.json**: Configuração principal do projeto Angular (build, serve, test)
- **package.json**: Lista de dependências e scripts npm
- **README.md**: Documentação do projeto
- **tsconfig.app.json**: Configuração TypeScript específica para a aplicação
- **tsconfig.json**: Configuração principal do TypeScript
- **tsconfig.spec.json**: Configuração TypeScript para testes


### Executando o projeto

```shell
ng server
```

### Criar um Componente no Angular

O comando básico para criar um componente é:
```shell
ng generate component nome-do-componente

ng generate component pasta/nome-do-componente
```

Ou a versão abreviada:
```shell
ng g c nome-do-componente
```


Exemplo Prático:
```shell
ng g c components/header
```

Este comando criará:
- `src/app/components/header/header.component.html` - Template HTML
- `src/app/components/header/header.component.ts` - Classe TypeScript
- `src/app/components/header/header.component.css` - Estilos CSS
- `src/app/components/header/header.component.spec.ts` - Arquivo de testes

### Estrutura dos Componentes

O arquivo `header.component.ts` terá uma estrutura como esta:

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-header',
  templateUrl: './header.component.html',
  styleUrls: ['./header.component.css']
})
export class HeaderComponent {
  // Propriedades e métodos do componente aqui
}
```

Opções Adicionais para Criação de Componentes
- Criar sem arquivo de teste: `ng g c nome-do-componente --skip-tests`
- Criar dentro de um módulo específico: `ng g c modulo/nome-do-componente`
- Criar com estilo inline: `ng g c nome-do-componente --inline-style`
- Criar com template inline: `ng g c nome-do-componente --inline-template`

### Utilizando um Componente
Depois de criar o componente, você pode usá-lo em qualquer template HTML de outro componente utilizando o seletor definido (por exemplo, `<app-header></app-header>`).
No arquivo `app.component.ts` existirá o decoretor @Component é um dos mais fundamentais no Angular. Ele marca uma classe como um componente Angular e fornece os metadados de configuração que determinam como o componente deve ser processado, instanciado e utilizado.
Vamos detalhar cada parte do decorador `@Component`: 

```typescript
@Component({
  selector: 'app-root',
  imports: [RouterOutlet, HeaderComponent],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css'
})
```

- **selector**: Define como o componente será referenciado no HTML. No seu caso, `'app-root'` significa que você pode usar `<app-root></app-root>` em templates HTML para incluir este componente.
- **imports**: Lista os componentes, diretivas ou pipes que são usados no template deste componente. No seu exemplo, você está importando:
    - `RouterOutlet`: Para permitir roteamento
    - `HeaderComponent`: Seu componente personalizado de cabeçalho
- **templateUrl**: Aponta para o arquivo HTML externo que define o template do componente.
- **styleUrl**: Aponta para o arquivo CSS que contém os estilos específicos deste componente.

```html
  

<h1>Hello World</h1>

<app-header></app-header>

<router-outlet />
```


### Criando uma Rota Simples no Angular
Configure o arquivo de rotas (app.routes.ts):
Você precisará definir suas rotas no arquivo `app.routes.ts`:
```typescript
import { Routes } from '@angular/router';
import { HomeComponent } from './components/home/home.component';
import { AboutComponent } from './components/about/about.component';

export const routes: Routes = [
  { path: '', component: HomeComponent },  // Rota padrão (página inicial)
  { path: 'about', component: AboutComponent },  // Rota para página "Sobre"
  { path: '**', redirectTo: '' }  // Redireciona qualquer rota não encontrada para a página inicial
];
```

Garanta que o Router Outlet está no seu template principal
No seu arquivo `app.component.html`, certifique-se de que você tem o `<router-outlet>` onde deseja que as páginas sejam carregadas:
```html
<app-header></app-header>
<main>
  <router-outlet></router-outlet>
</main>
```

Adicionando links de navegação:
No seu componente Header ou em qualquer outro lugar apropriado, adicione links para navegar entre as rotas:
```html
<!-- No header.component.html -->
<nav>
  <a routerLink="/">Home</a>
  <a routerLink="/about">About</a>
</nav>
```

Importe RouterLink no componente
No componente onde você está usando `routerLink` (como o HeaderComponent), você precisa importar RouterLink:
```typescript
import { Component } from '@angular/core';
import { RouterLink } from '@angular/router';

@Component({
  selector: 'app-header',
  standalone: true,
  imports: [RouterLink],
  templateUrl: './header.component.html',
  styleUrl: './header.component.css'
})
export class HeaderComponent {
  // ...
}
```

Com esses passos, você terá uma navegação básica funcionando em seu aplicativo Angular. Os usuários poderão navegar entre a página inicial e a página "About" usando os links ou digitando os URLs diretamente.

### Estados
Os estados no Angular são basicamente propriedades nas classes dos componentes:

```typescript
import { Component } from '@angular/core';

@Component({

selector: 'app-home',
imports: [],
templateUrl: './home.component.html',
styleUrl: './home.component.css'

})

export class HomeComponent {

// Estados (propriedades)

userName: string = 'Matheus';

isLoggedIn: boolean = true;

counter: number = 0;

users: string[] = ['Matheus', 'Matheuzinho', 'Matheusao'];

// Método para alterar estado 
incrementCounter() { this.counter++; }

}
```

### Renderização de Dados Dinâmicos
Para exibir dados dinâmicos no template, você usa a interpolação com chaves duplas `{{ }}`:

```html
<!-- home.component.html -->
<h1>Bem-vindo, {{ userName }}!</h1>
<p>Contador: {{ counter }}</p>

<!-- Renderização condicional com *ngIf -->
<div *ngIf="isLoggedIn">Conteúdo visível apenas para usuários logados</div>

<!-- Renderização de listas com *ngFor -->
<ul>
  <li *ngFor="let user of users">{{ user }}</li>
</ul>

<!-- Evento de clique para chamar método -->
<button (click)="incrementCounter()">Incrementar</button>
```

### Atributos Dinâmicos
Para vincular valores dinâmicos a atributos HTML, você usa colchetes `[ ]`:

```html
<!-- Vinculando propriedades a atributos -->
<img [src]="userProfileImage" [alt]="userName">

<!-- Vinculando a classes CSS dinamicamente -->
<div [class.active]="isActive" [class.disabled]="isDisabled">
  Este div terá classes dinâmicas
</div>

<!-- Vinculando a estilos inline -->
<p [style.color]="textColor" [style.font-size.px]="fontSize">
  Texto com estilo dinâmico
</p>

<!-- Desabilitando um botão com base em uma condição -->
<button [disabled]="!isFormValid">Enviar</button>
```

### Loops e Condicionais
No arquivo TypeScript de um componente Angular, você pode usar loops e condicionais da linguagem JavaScript/TypeScript padrão:

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-exemplo',
  templateUrl: './exemplo.component.html',
  styleUrl: './exemplo.component.scss'
})
export class ExemploComponent {
  items = ['Item 1', 'Item 2', 'Item 3', 'Item 4'];
  mostrarConteudo = true;
  
  // Exemplo de loop em TypeScript
  processarItems() {
    for (let i = 0; i < this.items.length; i++) {
      console.log(this.items[i]);
    }
    
    // Loop alternativo com forEach
    this.items.forEach(item => {
      console.log(item);
    });
  }
  
  // Exemplo de condicional em TypeScript
  verificarItem(item: string): string {
    if (item.includes('1')) {
      return 'Este item contém o número 1';
    } else if (item.includes('2')) {
      return 'Este item contém o número 2';
    } else {
      return 'Este item não contém 1 nem 2';
    }
  }
  
  // Condicional com operador ternário
  isItemImportante(item: string): boolean {
    return item.includes('1') ? true : false;
  }
}
```

No HTML (Template) , você usa diretivas estruturais para loops e condicionais no template HTML:

```html
<div>
  <!-- Condicional com *ngIf -->
  <div *ngIf="mostrarConteudo">
    Este conteúdo só aparece se mostrarConteudo for true
  </div>
  
  <!-- Condicional com else -->
  <div *ngIf="mostrarConteudo; else blocoElse">
    Conteúdo quando verdadeiro
  </div>
  <ng-template #blocoElse>
    Conteúdo alternativo quando falso
  </ng-template>
  
  <!-- Loop com *ngFor -->
  <ul>
    <li *ngFor="let item of items; let i = index">
      Item {{i}}: {{item}}
    </li>
  </ul>
  
  <!-- Combinando *ngFor com condicional -->
  <ul>
    <li *ngFor="let item of items">
      <span *ngIf="isItemImportante(item)" class="importante">
        {{item}} (Importante!)
      </span>
      <span *ngIf="!isItemImportante(item)">
        {{item}}
      </span>
    </li>
  </ul>
  
  <!-- Switch case com ngSwitch -->
  <div [ngSwitch]="items.length">
    <p *ngSwitchCase="0">Não há itens</p>
    <p *ngSwitchCase="1">Há apenas um item</p>
    <p *ngSwitchDefault>Há múltiplos itens: {{items.length}}</p>
  </div>
</div>
```

Principais recursos para loops e condicionais no template:

1. ***ngIf** - Para renderização condicional
2. ***ngFor** - Para loops/iterações
3. **ngSwitch**, ***ngSwitchCase**, ***ngSwitchDefault** - Para estruturas switch
4. **ng-template** - Para definir blocos de conteúdo reutilizáveis

### Services
Os Services no Angular são uma parte fundamental da arquitetura, permitindo compartilhar lógica, dados e funcionalidades entre diferentes componentes da aplicação. Vamos explorar como criar e utilizar services efetivamente:

Criação de um Service:

```typescript
// user.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { User } from './user.model';

@Injectable({
  providedIn: 'root'  // Disponibiliza o serviço globalmente na aplicação
})
export class UserService {
  private apiUrl = 'https://api.exemplo.com/users';

  constructor(private http: HttpClient) { }

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.apiUrl);
  }

  getUserById(id: number): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/${id}`);
  }

  createUser(user: User): Observable<User> {
    return this.http.post<User>(this.apiUrl, user);
  }

  updateUser(id: number, user: User): Observable<User> {
    return this.http.put<User>(`${this.apiUrl}/${id}`, user);
  }

  deleteUser(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}
```

Utilizando o Service em Componentes:

```typescript
// user-list.component.ts
import { Component, OnInit } from '@angular/core';
import { UserService } from '../services/user.service';
import { User } from '../models/user.model';

@Component({
  selector: 'app-user-list',
  templateUrl: './user-list.component.html'
})
export class UserListComponent implements OnInit {
  users: User[] = [];
  loading = false;
  error: string | null = null;

  constructor(private userService: UserService) { }

  ngOnInit(): void {
    this.loadUsers();
  }

  loadUsers(): void {
    this.loading = true;
    this.error = null;

    this.userService.getUsers().subscribe({
      next: (data) => {
        this.users = data;
        this.loading = false;
      },
      error: (err) => {
        this.error = 'Falha ao carregar usuários: ' + err.message;
        this.loading = false;
      }
    });
  }
}
```

Compartilhando Estado Entre Componentes:

```typescript
// shared-state.service.ts
import { Injectable } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class SharedStateService {
  private themeSubject = new BehaviorSubject<string>('light');
  private userSettingsSubject = new BehaviorSubject<any>({
    notificationsEnabled: true,
    language: 'pt-BR'
  });

  // Observables públicos que os componentes podem assinar
  theme$: Observable<string> = this.themeSubject.asObservable();
  userSettings$: Observable<any> = this.userSettingsSubject.asObservable();

  // Métodos para atualizar o estado
  setTheme(theme: string): void {
    this.themeSubject.next(theme);
  }

  updateUserSettings(settings: any): void {
    const currentSettings = this.userSettingsSubject.value;
    this.userSettingsSubject.next({ ...currentSettings, ...settings });
  }
}
```

Utilizando Signals no Angular 18

```typescript
// counter.service.ts
import { Injectable, signal, computed } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class CounterService {
  // Estado com signals
  private count = signal(0);
  private incrementAmount = signal(1);
  
  // Valores computed derivados
  readonly currentCount = this.count.asReadonly();
  readonly doubleCount = computed(() => this.count() * 2);
  readonly isPositive = computed(() => this.count() > 0);
  
  increment(): void {
    this.count.update(value => value + this.incrementAmount());
  }
  
  decrement(): void {
    this.count.update(value => value - this.incrementAmount());
  }
  
  reset(): void {
    this.count.set(0);
  }
  
  setIncrementAmount(amount: number): void {
    this.incrementAmount.set(amount);
  }
}
```

### Injeção de Dependência em Diferentes Níveis

**Nível Global**:
```typescript
@Injectable({
  providedIn: 'root'
})
```

**Nível de Módulo**:
```typescript
@NgModule({
  providers: [MeuService]
})
```

**Nível de Componente**:
```typescript
@Component({
  providers: [MeuService]
})
```

 Boas Práticas para Services

1. **Separar Responsabilidades**: Cada service deve ter um propósito único e bem definido
2. **Evitar Lógica nos Componentes**: Mantenha os componentes focados na apresentação
3. **Usar Interfaces**: Defina interfaces para os modelos de dados utilizados nos services
4. **Tratamento de Erros**: Implemente estratégias consistentes de tratamento de erros
5. **Cachear Resultados**: Para operações caras ou frequentes, considere implementar caching
6. **Testes Unitários**: Services são ideais para testes unitários por serem isolados da UI


### Comunicação entre componentes @Input e @Output

A comunicação entre componentes é fundamental para construir aplicações Angular modulares. O Angular oferece duas principais diretivas para facilitar essa comunicação: `@Input()` e `@Output()`.

Usando @Input para Passar Dados do Componente Pai para o Filho
O decorador `@Input()` permite que um componente pai envie dados para um componente filho.

Passo 1: componente pai (contador.component.ts)

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-contador',
  template: `
    <div style="margin: 20px; padding: 20px; border: 2px solid #333;">
      <h2>Componente Pai (Contador)</h2>
      <p>Contador: {{ valorContador }}</p>
      
      <!-- Usando o componente filho -->
      <app-botoes 
        [contador]="valorContador"
        (incrementado)="aumentarContador()"
        (decrementado)="diminuirContador()">
      </app-botoes>
    </div>
  `
})
export class ContadorComponent {
  valorContador: number = 0;
  
  aumentarContador() {
    this.valorContador++;
  }
  
  diminuirContador() {
    this.valorContador--;
  }
}

```


Usando @Output para Passar Dados do Componente Filho para o Pai
O decorador `@Output()` permite que um componente filho envie eventos para um componente pai.

```typescript

import { Component, Input, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-botoes',
  template: `
    <div style="margin: 20px; padding: 10px; border: 1px solid #ccc;">
      <h3>Componente Filho (Botões)</h3>
      <p>Valor atual: {{ contador }}</p>
      <button (click)="incrementar()">+1</button>
      <button (click)="decrementar()">-1</button>
    </div>
  `
})
export class BotoesComponent {
  // Recebe dados do componente pai
  @Input() contador: number = 0;
  
  // Envia eventos para o componente pai
  @Output() incrementado = new EventEmitter<void>();
  @Output() decrementado = new EventEmitter<void>();
  
  incrementar() {
    // Emite um evento para o pai
    this.incrementado.emit();
  }
  
  decrementar() {
    // Emite um evento para o pai
    this.decrementado.emit();
  }
}
```


