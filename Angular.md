## 1. Что такое Angular

**Angular** — полноценный frontend-фреймворк от Google для разработки SPA.

В отличие от React, где ты сам выбираешь большую часть экосистемы, Angular изначально предоставляет:

- компоненты
- роутинг
- DI
- HTTP-клиент
- формы
- lifecycle
- RxJS
- инструменты CLI
- систему модулей/standalone components
- встроенные механизмы тестирования

Основной стек:

```
Angular
├── TypeScript
├── Components
├── Templates
├── Dependency Injection
├── Services
├── Router
├── Forms
├── HttpClient
└── RxJS
```

---

# 2. Component

Главная строительная единица Angular - **компонент**.

Пример:

```
@Component({
  selector: 'app-user',
  template: `
    <h1>{{ name }}</h1>
    <button (click)="changeName()">Change</button>
  `
})
export class UserComponent {
  name = 'Oleg';

  changeName() {
    this.name = 'Alex';
  }
}
```

В React примерно:

```
function User() {
  const [name, setName] = useState('Oleg');

  return (
    <>
      <h1>{name}</h1>
      <button onClick={() => setName('Alex')}>
        Change
      </button>
    </>
  );
}
```

То есть:

|React|Angular|
|---|---|
|Component|Component|
|JSX|Template|
|props|@Input|
|callback|@Output|
|hooks|lifecycle + DI + signals/RxJS|
|Context|DI / services|
|React Router|Angular Router|
|fetch/axios|HttpClient|
|Redux|Signals / RxJS / services / NgRx|

---

# 3. Template

Angular отделяет HTML-шаблон от TypeScript.

```
@Component({
  template: `
    <h1>{{ title }}</h1>

    <button (click)="increment()">
      {{ count }}
    </button>
  `
})
export class AppComponent {
  title = 'Counter';
  count = 0;

  increment() {
    this.count++;
  }
}
```

`{{ }}` - interpolation:

```
<h1>{{ title }}</h1>
```

---

# 4. Data Binding

В Angular есть несколько основных видов binding.

### Interpolation

```
<h1>{{ name }}</h1>
```

Данные из TS → HTML.

### Property binding

```
<img [src]="imageUrl">
```

Можно передавать свойства DOM.

### Event binding

```
<button (click)="save()">
  Save
</button>
```

HTML → TypeScript.

### Two-way binding

```
<input [(ngModel)]="name">
```

Изменение input автоматически меняет `name`.

Это называется:

**two-way data binding**.

---

# 5. @Input

Передача данных от родителя ребенку.

Родитель:

```
<app-user [user]="currentUser"></app-user>
```

Ребенок:

```
@Component({...})
export class UserComponent {
  @Input() user!: User;
}
```

В React:

```
<User user={currentUser} />
```

---

# 6. @Output

Передача события от ребенка родителю.

```
@Output() saved = new EventEmitter<User>();

save() {
  this.saved.emit(this.user);
}
```

Родитель:

```
<app-user (saved)="handleSave($event)">
</app-user>
```

Примерная аналогия:

```
<User onSave={handleSave} />
```

---

# 7. Directives

Directive позволяет изменять поведение или отображение элемента.

Например:

```
<div *ngIf="isVisible">
  Content
</div>
```

или:

```
<div [class.active]="isActive">
  User
</div>
```

Есть:

### Structural directives

Меняют структуру DOM:

```
*ngIf
*ngFor
```

В современных версиях Angular также есть новый control flow:

```
@if (isVisible) {
  <div>Content</div>
}

@for (user of users; track user.id) {
  <div>{{ user.name }}</div>
}
```

### Attribute directives

Меняют поведение или свойства:

```
<div [ngClass]="classes"></div>
```

---

# 8. Services

Сервис - класс для вынесения бизнес-логики, API, состояния и других общих функций.

```
@Injectable({
  providedIn: 'root'
})
export class UserService {

  getUsers() {
    return this.http.get<User[]>('/api/users');
  }

  constructor(private http: HttpClient) {}
}
```

Компонент:

```
constructor(private userService: UserService) {}
```

Это одна из ключевых концепций Angular.

---

# 9. Dependency Injection

Angular имеет встроенный **DI-контейнер**.

Например:

```
constructor(
  private userService: UserService,
  private router: Router
) {}
```

Angular сам создаёт и передаёт зависимости.

В React аналогичной встроенной системы нет.

Обычно приходится использовать:

- Context
- DI-библиотеки
- Redux
- собственные abstractions

В Angular DI является частью самого фреймворка.

---

# 10. Lifecycle

У Angular есть lifecycle hooks.

Самые известные:

```
ngOnInit()
ngOnChanges()
ngAfterViewInit()
ngOnDestroy()
```

Например:

```
export class UserComponent implements OnInit, OnDestroy {

  ngOnInit() {
    console.log('component created');
  }

  ngOnDestroy() {
    console.log('component destroyed');
  }
}
```

Аналогия с React:

| Angular         | React                      |
| --------------- | -------------------------- |
| ngOnInit        | useEffect(() => {}, [])    |
| ngOnDestroy     | cleanup в useEffect        |
| ngOnChanges     | реакция на изменение props |
| ngAfterViewInit | после рендера DOM          |

Но прямое соответствие не всегда корректно.

---

# 11. RxJS

Это очень важная часть Angular.

**RxJS** - библиотека для работы с Observable.

Например:

```
users$ = this.userService.getUsers();
```

В шаблоне:

```
<div *ngFor="let user of users$ | async">
  {{ user.name }}
</div>
```

Observable можно представить как поток данных:

```
Observable
   ↓
  data
   ↓
  data
   ↓
  data
   ↓
 complete
```

Основные операторы:

```
map()
filter()
switchMap()
mergeMap()
concatMap()
catchError()
debounceTime()
distinctUntilChanged()
combineLatest()
forkJoin()
```

Для React-разработчика особенно важно понимать:

```
switchMap()
```

Например, поиск:

```
пользователь вводит:

r
re
rea
reac
react
```

`switchMap` позволяет отменять/игнорировать предыдущие запросы и работать с актуальным.

---

# 12. Signals

В современных Angular есть **Signals** - реактивный примитив для состояния.

```
count = signal(0);
```

Получить значение:

```
count()
```

Изменить:

```
count.set(10);
```

или:

```
count.update(value => value + 1);
```

Computed:

```
double = computed(() => this.count() * 2);
```

Это уже ближе к привычной React-модели реактивного состояния.

Упрощённо:

```
signal
   ↓
изменение
   ↓
Angular понимает,
что нужно обновить
   ↓
зависимый UI
```

---

# 13. Forms

Angular имеет две основные системы форм.

### Template-driven

```
<input
  [(ngModel)]="name"
>
```

Простая форма.

### Reactive Forms

Для сложных приложений чаще используют Reactive Forms.

```
form = new FormGroup({
  name: new FormControl(''),
  email: new FormControl('')
});
```

HTML:

```
<form [formGroup]="form">
  <input formControlName="name">
  <input formControlName="email">
</form>
```

Можно добавить validators:

```
email: new FormControl('', [
  Validators.required,
  Validators.email
])
```

---

# 14. HttpClient

Angular имеет встроенный HTTP-клиент.

```
this.http.get<User[]>('/api/users');
```

POST:

```
this.http.post('/api/users', user);
```

Обычно результат - Observable:

```
getUsers(): Observable<User[]> {
  return this.http.get<User[]>('/api/users');
}
```

---

# 15. Interceptors

Очень часто спрашивают на собеседовании.

Interceptor позволяет перехватывать HTTP-запросы.

Например, добавить JWT:

```
Component
   ↓
Service
   ↓
HttpClient
   ↓
Interceptor
   ↓
API
```

Можно автоматически добавить:

```
Authorization: Bearer token
```

Также interceptor используют для:

- обработки ошибок
- логирования
- refresh token
- добавления headers
- loading state

---

# 16. Routing

Angular Router отвечает за SPA-навигацию.

```
const routes: Routes = [
  {
    path: 'users',
    component: UsersComponent
  },
  {
    path: 'users/:id',
    component: UserComponent
  }
];
```

В шаблоне:

```
<a routerLink="/users">
  Users
</a>

<router-outlet></router-outlet>
```

`router-outlet` - место, куда Angular вставляет текущий компонент маршрута.

---

# 17. Lazy Loading

Большое приложение не обязательно грузить целиком.

Можно лениво загружать route:

```
{
  path: 'admin',
  loadComponent: () =>
    import('./admin.component')
      .then(m => m.AdminComponent)
}
```

Получаем:

```
Initial bundle
     ↓
    app
     ↓
user открывает /admin
     ↓
загружается admin chunk
```

Это аналогично lazy loading в React:

```
const Admin = lazy(() => import('./Admin'));
```

---

# 18. Standalone Components

В старых Angular приложениях активно использовались `NgModule`.

Например:

```
@NgModule({
  declarations: [
    AppComponent,
    UserComponent
  ],
  imports: [
    BrowserModule
  ]
})
export class AppModule {}
```

В современных Angular можно использовать **standalone components**:

```
@Component({
  standalone: true,
  imports: [CommonModule],
  template: `...`
})
export class UserComponent {}
```

Идея похожа на более модульный подход:

```
Component
 ├── imports
 ├── dependencies
 └── template
```

без обязательного `NgModule`.

---

# 19. Change Detection

Очень важная тема для собеседования.

Angular должен понимать:

> Нужно ли обновить DOM после изменения данных?

За это отвечает **change detection**.

Есть стратегия:

```
ChangeDetectionStrategy.OnPush
```

Она позволяет сократить количество проверок компонента.

Пример:

```
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
```

В React похожая область оптимизации:

```
React
├── memo
├── useMemo
├── useCallback
└── reconciliation

Angular
├── OnPush
├── signals
└── change detection
```

Но механизмы отличаются.

---

# 20. Архитектура Angular-приложения

Условно:

```
App
│
├── Components
│
├── Services
│
├── Router
│
├── Forms
│
├── HttpClient
│
├── State
│
└── RxJS / Signals
```

Типичная структура:

```
src/
├── app/
│   ├── components/
│   ├── services/
│   ├── pages/
│   ├── guards/
│   ├── interceptors/
│   └── app.routes.ts
│
├── assets/
└── main.ts
```

---

# 21. Guards

Guards используются для контроля маршрутов.

Например:

```
/admin
   ↓
AuthGuard
   ↓
есть авторизация?
   ├── yes → Admin
   └── no  → Login
```

Современный вариант:

```
export const authGuard: CanActivateFn = () => {
  return authService.isAuthenticated();
};
```

---

# 22. Pipes

Pipe используется для преобразования данных прямо в template.

```
{{ user.name | uppercase }}
```

Другие:

```
{{ date | date }}
{{ price | currency }}
```

Можно создавать собственные:

```
@Pipe({
  name: 'shortName'
})
export class ShortNamePipe {
  transform(value: string) {
    return value.slice(0, 10);
  }
}
```

В React похожее обычно делают обычной функцией:

```
formatName(user.name)
```

---

# 23. Что важно знать React-разработчику

Если на собеседовании спрашивают **Angular**, я бы в первую очередь выучил вот этот набор:

```
1. Component
2. Template
3. @Input / @Output
4. Services
5. Dependency Injection
6. Lifecycle hooks
7. RxJS / Observable
8. Signals
9. Reactive Forms
10. HttpClient
11. Interceptors
12. Router
13. Guards
14. Lazy Loading
15. Change Detection
16. OnPush
17. Pipes
18. Directives
19. Standalone Components
20. NgModules - хотя бы понимать legacy-код
```

### Самая полезная ментальная модель

Если привык к React, можно держать в голове такую таблицу:

```
React                    Angular
------------------------------------------------
Component                Component
JSX                      Template
props                    @Input
callback props           @Output
Context                  DI / Service
useEffect                Lifecycle hooks
useState                 Signals
Redux                    NgRx / Signals / Services
React Router             Angular Router
fetch / axios            HttpClient
React Hook Form          Reactive Forms
React.memo               OnPush
Suspense/lazy            Lazy Loading
custom hooks             Services / RxJS
```

**Главное отличие:** React - это в первую очередь UI-библиотека, вокруг которой собирается архитектура приложения. Angular - полноценный opinionated framework, где архитектурные механизмы вроде DI, Router, Forms и HTTP-клиента уже являются частью экосистемы.