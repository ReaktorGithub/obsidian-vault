В NestJS архитектура основана на идеях **MVC** и **Dependency Injection**. Три ключевых понятия — **модули, контроллеры и сервисы** — отвечают за разные уровни ответственности.

Модули организуют приложение, DI связывает зависимости, контроллеры работают с HTTP, providers содержат логику, а middleware/guards/pipes/interceptors/filters позволяют вмешиваться в жизненный цикл запроса.

## 1. Что такое NestJS

**NestJS** - backend-фреймворк для Node.js, построенный вокруг TypeScript и модульной архитектуры.

Основные идеи:

- модули
- контроллеры
- providers
- Dependency Injection
- декораторы
- middleware
- guards
- interceptors
- pipes
- exception filters

NestJS по умолчанию работает поверх **Express**, но может использовать **Fastify**.

---

# 2. Архитектура NestJS

Типичная структура:

```
src/
├── app.module.ts
├── main.ts
│
├── users/
│   ├── users.module.ts
│   ├── users.controller.ts
│   ├── users.service.ts
│   └── dto/
│       └── create-user.dto.ts
│
└── auth/
    ├── auth.module.ts
    ├── auth.controller.ts
    └── auth.service.ts
```

Основная цепочка:

```
HTTP request
     ↓
Middleware
     ↓
Guards
     ↓
Interceptors
     ↓
Pipes
     ↓
Controller
     ↓
Service / Provider
     ↓
Response
```

При этом exception filters используются для обработки исключений.

---

# 3. `main.ts`

Точка входа приложения.

```
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  await app.listen(3000);
}

bootstrap();
```

`NestFactory.create()` создает экземпляр приложения.

Можно настроить глобальные middleware, pipes, guards и т.д.

```
const app = await NestFactory.create(AppModule);

app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,
  }),
);

await app.listen(3000);
```

---

# 4. Module

**Module** объединяет связанные части приложения.

```
@Module({
  controllers: [UserController],
  providers: [UserService],
})
export class UserModule {}
```

У модуля есть основные свойства:

```
@Module({
  imports: [],
  controllers: [],
  providers: [],
  exports: [],
})
```

### `controllers`

Контроллеры модуля.

```
controllers: [UserController]
```

### `providers`

Сервисы и другие зависимости, которыми управляет DI-контейнер NestJS.

```
providers: [UserService]
```

### `imports`

Другие модули, чьи экспортируемые providers нужны этому модулю.

```
imports: [AuthModule]
```

### `exports`

Делает provider доступным для модулей, которые импортируют текущий модуль.

```
@Module({
  providers: [UserService],
  exports: [UserService],
})
export class UserModule {}
```

---

# 5. Controller

Контроллер отвечает за HTTP-запросы.

```
@Controller('users')
export class UserController {
  @Get()
  findAll() {
    return [];
  }
}
```

Получится:

```
GET /users
```

---

## HTTP методы

```
@Get()
findAll() {}

@Get(':id')
findOne() {}

@Post()
create() {}

@Patch(':id')
update() {}

@Delete(':id')
remove() {}
```

---

# 6. Параметры запроса

## URL params

```
GET /users/123
```

```
@Get(':id')
findOne(@Param('id') id: string) {
  return id;
}
```

---

## Query params

```
GET /users?page=2&limit=10
```

```
@Get()
findAll(
  @Query('page') page: string,
  @Query('limit') limit: string,
) {
  return { page, limit };
}
```

Или весь query:

```
@Get()
findAll(@Query() query: Record<string, string>) {
  return query;
}
```

---

## Body

```
POST /users

{
  "name": "Alex",
  "email": "alex@test.com"
}
```

```
@Post()
create(@Body() body: CreateUserDto) {
  return body;
}
```

---

## Headers

```
@Get()
getData(@Headers('authorization') authorization: string) {
  return authorization;
}
```

---

# 7. Service

Бизнес-логика обычно находится в service.

```
@Injectable()
export class UserService {
  findAll() {
    return [];
  }

  findOne(id: string) {
    return { id };
  }
}
```

Контроллер использует service:

```
@Controller('users')
export class UserController {
  constructor(
    private readonly userService: UserService,
  ) {}

  @Get()
  findAll() {
    return this.userService.findAll();
  }
}
```

Идея:

```
Controller
    ↓
Service
    ↓
Business logic
```

Контроллер лучше держать тонким, а бизнес-логику выносить в providers.

---

# 8. Dependency Injection

NestJS использует **Dependency Injection**.

Вместо ручного создания:

```
const userService = new UserService();
```

NestJS сам создает зависимость:

```
constructor(
  private readonly userService: UserService,
) {}
```

Nest видит, что `UserController` зависит от `UserService`, и передает экземпляр автоматически.

---

# 9. `@Injectable()`

```
@Injectable()
export class UserService {}
```

`@Injectable()` сообщает NestJS, что класс может управляться DI-контейнером.

Например:

```
@Injectable()
export class UserService {
  constructor(
    private readonly authService: AuthService,
  ) {}
}
```

Nest автоматически внедрит `AuthService`, если он доступен в текущем модуле.

---

# 10. Providers

Provider - это зависимость, которую Nest может создать и внедрить.

Обычно:

```
providers: [UserService]
```

Но provider может быть зарегистрирован более явно:

```
providers: [
  {
    provide: 'CONFIG',
    useValue: {
      apiUrl: 'https://example.com',
    },
  },
]
```

Использование:

```
constructor(
  @Inject('CONFIG')
  private readonly config: {
    apiUrl: string;
  },
) {}
```

---

# 11. `useValue`

Позволяет передать готовое значение.

```
providers: [
  {
    provide: 'APP_CONFIG',
    useValue: {
      port: 3000,
      environment: 'development',
    },
  },
]
```

Полезно для конфигурации и тестов.

---

# 12. `useClass`

Можно указать класс, который должен использоваться как provider.

```
providers: [
  {
    provide: 'LOGGER',
    useClass: CustomLogger,
  },
]
```

---

# 13. `useFactory`

Provider может создаваться функцией.

```
providers: [
  {
    provide: 'CONFIG',
    useFactory: () => ({
      apiUrl: process.env.API_URL,
    }),
  },
]
```

Можно использовать зависимости:

```
{
  provide: 'CONFIG',
  inject: [ConfigService],
  useFactory: (config: ConfigService) => ({
    apiUrl: config.get('API_URL'),
  }),
}
```

---

# 14. DTO

**DTO (Data Transfer Object)** описывает данные, которые передаются через границу приложения.

```
export class CreateUserDto {
  name: string;
  email: string;
}
```

Использование:

```
@Post()
create(@Body() dto: CreateUserDto) {
  return this.userService.create(dto);
}
```

DTO не обязательно должен быть Entity или моделью базы.

---

# 15. ValidationPipe

Для валидации DTO обычно используют `class-validator`.

```
export class CreateUserDto {
  @IsString()
  name: string;

  @IsEmail()
  email: string;
}
```

Подключение:

```
app.useGlobalPipes(
  new ValidationPipe(),
);
```

Теперь Nest проверяет входные данные.

---

# 16. `whitelist`

```
new ValidationPipe({
  whitelist: true,
})
```

Удаляет свойства, которых нет в DTO.

Например:

```
{
  "name": "Alex",
  "email": "a@test.com",
  "isAdmin": true
}
```

Если `isAdmin` нет в DTO, при `whitelist: true` оно будет удалено.

---

# 17. `forbidNonWhitelisted`

```
new ValidationPipe({
  whitelist: true,
  forbidNonWhitelisted: true,
})
```

Теперь вместо удаления неизвестного поля Nest вернет ошибку.

---

# 18. Pipes

Pipe используется для:

- валидации
- преобразования данных

Например:

```
@Get(':id')
findOne(
  @Param('id', ParseIntPipe) id: number,
) {
  return id;
}
```

Запрос:

```
GET /users/123
```

`id` будет преобразован из строки `"123"` в число `123`.

---

# 19. Custom Pipe

```
@Injectable()
export class ParsePositiveIntPipe
  implements PipeTransform
{
  transform(value: string) {
    const number = Number(value);

    if (!Number.isInteger(number) || number <= 0) {
      throw new BadRequestException();
    }

    return number;
  }
}
```

Использование:

```
@Get(':id')
findOne(
  @Param('id', ParsePositiveIntPipe) id: number,
) {}
```

---

# 20. Middleware

Middleware выполняется до обработки запроса контроллером.

```
@Injectable()
export class LoggerMiddleware
  implements NestMiddleware {

  use(req: Request, res: Response, next: NextFunction) {
    console.log(req.method, req.url);

    next();
  }
}
```

Подключение:

```
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes('*');
  }
}
```

Middleware хорошо подходит для:

- логирования
- работы с request
- добавления данных в request
- технических проверок

---

# 21. Guards

Guard решает, **может ли запрос продолжить выполнение**.

Типичный пример - авторизация.

```
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();

    return Boolean(request.user);
  }
}
```

Использование:

```
@UseGuards(AuthGuard)
@Get('profile')
getProfile() {
  return {};
}
```

Если guard возвращает `false`, обработка запроса прекращается.

---

# 22. Middleware vs Guard

### Middleware

Работает раньше и в первую очередь занимается обработкой request.

```
Request
  ↓
Middleware
```

### Guard

Отвечает за доступ к endpoint.

```
Request
  ↓
Middleware
  ↓
Guard
  ↓
Controller
```

**На собеседовании:** authentication/authorization обычно логичнее реализовывать через Guard, а не Middleware.

---

# 23. Interceptors

Interceptor позволяет выполнить код **до и после** выполнения handler.

```
@Injectable()
export class LoggingInterceptor
  implements NestInterceptor {

  intercept(
    context: ExecutionContext,
    next: CallHandler,
  ) {
    console.log('before');

    return next.handle().pipe(
      tap(() => {
        console.log('after');
      }),
    );
  }
}
```

Использование:

```
@UseInterceptors(LoggingInterceptor)
@Get()
findAll() {
  return [];
}
```

---

# 24. Для чего нужны Interceptors

Типичные применения:

- логирование
- измерение времени выполнения
- изменение response
- кеширование
- преобразование данных
- обработка результата

Например, измерение времени:

```
const start = Date.now();

return next.handle().pipe(
  tap(() => {
    console.log(Date.now() - start);
  }),
);
```

---

# 25. Exception Filters

Filter отвечает за обработку исключений.

Например:

```
throw new NotFoundException('User not found');
```

Можно создать собственный filter:

```
@Catch(HttpException)
export class HttpExceptionFilter
  implements ExceptionFilter {

  catch(
    exception: HttpException,
    host: ArgumentsHost,
  ) {
    const response = host
      .switchToHttp()
      .getResponse();

    response.status(exception.getStatus()).json({
      message: exception.message,
    });
  }
}
```

---

# 26. Встроенные HTTP exceptions

NestJS предоставляет:

```
BadRequestException
UnauthorizedException
ForbiddenException
NotFoundException
ConflictException
InternalServerErrorException
```

Например:

```
if (!user) {
  throw new NotFoundException('User not found');
}
```

Nest автоматически вернет HTTP 404.

---

# 27. Request lifecycle

Один из популярных вопросов на собеседовании.

Упрощенно:

```
Request
   ↓
Middleware
   ↓
Guards
   ↓
Interceptors
   ↓
Pipes
   ↓
Controller
   ↓
Service
   ↓
Response
```

Более точно:

```
Middleware
   ↓
Guards
   ↓
Interceptors - до
   ↓
Pipes
   ↓
Controller
   ↓
Service
   ↓
Interceptors - после
   ↓
Response
```

Exception filters обрабатывают возникшие исключения.

---

# 28. Decorators

NestJS активно использует TypeScript decorators.

Например:

```
@Controller('users')
```

```
@Get()
```

```
@Injectable()
```

```
@Module({})
```

```
@Body()
```

```
@Param()
```

Декораторы добавляют метаданные, которые Nest использует во время построения приложения.

---

# 29. Custom Decorator

Можно создавать собственные декораторы.

Например:

```
export const CurrentUser = createParamDecorator(
  (_data, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();

    return request.user;
  },
);
```

Использование:

```
@Get('profile')
getProfile(@CurrentUser() user: User) {
  return user;
}
```

Это позволяет не писать каждый раз:

```
const request = context.switchToHttp().getRequest();
```

---

# 30. Global Guard

Guard можно сделать глобальным:

```
app.useGlobalGuards(new AuthGuard());
```

Но если Guard зависит от других providers, чаще используют:

```
providers: [
  {
    provide: APP_GUARD,
    useClass: AuthGuard,
  },
]
```

---

# 31. Global Pipe

```
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,
  }),
);
```

Теперь pipe применяется ко всем endpoint.

---

# 32. Global Interceptor

```
app.useGlobalInterceptors(
  new LoggingInterceptor(),
);
```

Применяется ко всему приложению.

---

# 33. Global Filter

```
app.useGlobalFilters(
  new HttpExceptionFilter(),
);
```

Обрабатывает исключения глобально.

---

# 34. Module imports / exports

Очень важная тема.

Есть:

```
@Module({
  providers: [UserService],
  exports: [UserService],
})
export class UserModule {}
```

Другой модуль:

```
@Module({
  imports: [UserModule],
  providers: [AuthService],
})
export class AuthModule {}
```

Теперь `AuthService` может получить `UserService`:

```
@Injectable()
export class AuthService {
  constructor(
    private readonly userService: UserService,
  ) {}
}
```

Логика:

```
UserModule
   │
   ├── UserService
   │
   └── exports UserService
             ↓
        AuthModule
             ↓
       imports UserModule
             ↓
       AuthService
```

---

# 35. Shared Module

Если один provider нужен многим модулям, его можно вынести в отдельный модуль.

```
@Module({
  providers: [LoggerService],
  exports: [LoggerService],
})
export class LoggerModule {}
```

Другие модули:

```
@Module({
  imports: [LoggerModule],
})
export class UserModule {}
```

---

# 36. Global Module

Модуль можно сделать глобальным:

```
@Global()
@Module({
  providers: [ConfigService],
  exports: [ConfigService],
})
export class ConfigModule {}
```

После этого его provider доступен в других модулях без явного `imports`.

Использовать глобальные модули стоит умеренно, чтобы зависимости оставались явными.

---

# 37. Lifecycle Hooks

NestJS предоставляет lifecycle hooks.

Например:

```
@Injectable()
export class AppService
  implements OnModuleInit {

  onModuleInit() {
    console.log('Module initialized');
  }
}
```

Основные hooks:

```
onModuleInit
onApplicationBootstrap
onModuleDestroy
beforeApplicationShutdown
onApplicationShutdown
```

Используются для:

- инициализации ресурсов
- подключения к внешним сервисам
- graceful shutdown
- освобождения ресурсов

---

# 38. Scopes

По умолчанию provider имеет **singleton scope**.

То есть Nest создает один экземпляр на приложение.

```
@Injectable()
export class UserService {}
```

Можно изменить scope:

```
@Injectable({
  scope: Scope.REQUEST,
})
export class UserService {}
```

Основные варианты:

```
DEFAULT
REQUEST
TRANSIENT
```

### DEFAULT

Один экземпляр на приложение.

### REQUEST

Новый экземпляр для каждого HTTP request.

### TRANSIENT

Новый экземпляр для каждого consumer.

---

# 39. ConfigModule

Для конфигурации приложения обычно используют `@nestjs/config`.

```
ConfigModule.forRoot()
```

Получение значения:

```
@Injectable()
export class AppService {
  constructor(
    private readonly configService: ConfigService,
  ) {}

  getPort() {
    return this.configService.get<number>('PORT');
  }
}
```

`.env`:

```
PORT=3000
JWT_SECRET=secret
```

---

# 40. Routing

Можно создавать вложенные маршруты.

```
@Controller('users')
export class UserController {
  @Get()
  findAll() {}

  @Get(':id')
  findOne() {}
}
```

Получаем:

```
GET /users
GET /users/:id
```

---

# 41. HTTP Status Codes

Можно явно указать статус:

```
@Post()
@HttpCode(HttpStatus.CREATED)
create() {}
```

Или:

```
@HttpCode(204)
delete() {}
```

По умолчанию Nest сам выбирает статус в зависимости от HTTP метода.

---

# 42. Response

Можно вернуть обычный объект:

```
@Get()
findOne() {
  return {
    id: 1,
    name: 'Alex',
  };
}
```

Nest сериализует его в JSON.

Можно использовать:

```
@Res()
```

для ручной работы с response:

```
@Get()
find(@Res() res: Response) {
  res.status(200).json({
    message: 'ok',
  });
}
```

Но без необходимости лучше позволять Nest самому обрабатывать response.

---

# 43. Async handlers

Nest нормально работает с Promise.

```
@Get()
async findAll() {
  return await this.userService.findAll();
}
```

Можно вернуть Promise напрямую:

```
@Get()
findAll() {
  return this.userService.findAll();
}
```

---

# 44. Express vs Fastify

Nest абстрагирует HTTP-слой.

По умолчанию:

```
NestJS
  ↓
Express
```

Можно использовать Fastify:

```
NestJS
  ↓
Fastify
```

Основной код NestJS при этом остается практически тем же.

---

# 45. Архитектурный принцип

Хорошая структура:

```
Controller
    ↓
Service
    ↓
другие providers
```

Контроллер:

```
@Post()
create(@Body() dto: CreateUserDto) {
  return this.userService.create(dto);
}
```

Service:

```
create(dto: CreateUserDto) {
  // бизнес-логика
}
```

Контроллер не должен содержать большую бизнес-логику.

---

# 46. Частый вопрос: Controller vs Service

**Controller** отвечает за транспортный слой:

- HTTP методы
- URL
- params
- body
- status codes

**Service** отвечает за бизнес-логику:

- вычисления
- правила
- взаимодействие с внешними сервисами
- работу с данными

```
HTTP
 ↓
Controller
 ↓
Service
 ↓
Business logic
```

---

# 47. Частый вопрос: Guard vs Interceptor vs Pipe

|Механизм|Основная задача|
|---|---|
|Middleware|обработка request до Nest pipeline|
|Guard|разрешить или запретить выполнение|
|Pipe|валидация и преобразование данных|
|Interceptor|выполнить код до/после handler|
|Filter|обработка exceptions|

Запомнить:

```
Middleware -> подготовить request
Guard      -> можно ли выполнять?
Pipe       -> данные корректны?
Interceptor -> что сделать до/после?
Filter     -> как обработать ошибку?
```

---

# 48. Частый вопрос: зачем NestJS DI?

Без DI:

```
class UserController {
  private service = new UserService();
}
```

С DI:

```
class UserController {
  constructor(
    private readonly service: UserService,
  ) {}
}
```

Преимущества:

- слабая связанность
- проще тестировать
- проще заменять реализации
- Nest управляет жизненным циклом зависимостей

Например, в тесте можно заменить настоящий service на mock.

---

# 49. Тестирование

NestJS предоставляет инструменты для создания TestingModule.

```
const module = await Test.createTestingModule({
  providers: [UserService],
}).compile();

const service = module.get(UserService);
```

Можно заменить dependency:

```
{
  provide: UserRepository,
  useValue: mockRepository,
}
```

Это один из главных плюсов DI.

---

# 50. Unit vs E2E

### Unit test

Тестирует отдельный класс или service.

```
UserService
    ↓
mock dependencies
```

### E2E

Тестирует приложение целиком через HTTP:

```
HTTP request
   ↓
Controller
   ↓
Service
   ↓
Response
```

---

# 51. Circular dependency

Проблема:

```
UserService → AuthService
AuthService → UserService
```

Получается циклическая зависимость.

Nest предоставляет `forwardRef()` для таких случаев:

```
@Inject(forwardRef(() => AuthService))
private readonly authService: AuthService;
```

Но лучше по возможности изменить архитектуру и убрать циклическую зависимость.

---

# 52. Что такое Dynamic Module

Dynamic Module позволяет создавать модуль с конфигурацией во время запуска.

Например:

```
@Module({})
export class DatabaseModule {
  static forRoot(config: Config) {
    return {
      module: DatabaseModule,
      providers: [
        {
          provide: 'CONFIG',
          useValue: config,
        },
      ],
    };
  }
}
```

Использование:

```
imports: [
  DatabaseModule.forRoot({
    host: 'localhost',
  }),
]
```

Часто встречается в библиотеках NestJS.

---

# 53. Decorator composition

Можно объединить несколько декораторов:

```
export function Auth() {
  return applyDecorators(
    UseGuards(AuthGuard),
    ApiBearerAuth(),
  );
}
```

Теперь:

```
@Auth()
@Get('profile')
getProfile() {}
```

Вместо нескольких декораторов.

---

# 54. Основная схема NestJS

```
                    NestJS
                       │
             ┌─────────┴─────────┐
             │                   │
          Modules             DI Container
             │
      ┌──────┼──────┐
      │      │      │
 Controller Service Providers
      │      │
      │      └────── Business logic
      │
      └── HTTP layer
             │
     ┌───────┼────────┐
     │       │        │
 Middleware Guard    Interceptor
                     │
                    Pipe
                     │
                 Controller
                     │
                   Service
                     │
                  Response
```

# Что особенно важно знать на собеседовании

Если времени мало, в первую очередь выучи:

1. **Module** - `imports`, `controllers`, `providers`, `exports`
2. **Controller** - маршруты и HTTP-запросы
3. **Provider / Service**
4. **Dependency Injection**
5. **DTO**
6. **ValidationPipe**
7. **Middleware**
8. **Guard**
9. **Interceptor**
10. **Exception Filter**
11. **Request lifecycle**
12. **Scopes**
13. **Lifecycle hooks**
14. **Custom providers**
15. **Global pipes/guards/interceptors/filters**
16. **Module imports/exports**
17. **Unit и E2E testing**
18. **Express vs Fastify**
19. **Circular dependencies**
20. **Dynamic modules**
