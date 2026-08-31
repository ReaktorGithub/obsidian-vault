Вот **10 наиболее вероятных вопросов по Express.js** для frontend-разработчика уровня Middle, с короткими ответами.

### 1. Что такое Express.js?

**Express.js** - минималистичный web-фреймворк для Node.js. Используется для создания HTTP-серверов, API, middleware и обработки маршрутов.

### 2. Что такое middleware?

Middleware - функция, которая получает `req`, `res`, `next` и выполняется между получением запроса и отправкой ответа.

```
app.use((req, res, next) => {
  console.log(req.url);
  next();
});
```

`next()` передаёт управление следующему middleware.

### 3. Что такое routing?

Routing - определение того, **какой обработчик должен выполнить запрос** в зависимости от HTTP-метода и URL.

```
app.get('/users', (req, res) => {
  res.json(users);
});
```

### 4. Как обрабатывать параметры URL?

Есть несколько типов:

```
/users/:id
```

`req.params.id` - параметр пути.

```
/users?id=123
```

`req.query.id` - query-параметр.

Тело запроса:

```
req.body
```

### 5. Как обработать JSON в body?

Использовать встроенный middleware:

```
app.use(express.json());
```

После этого:

```
req.body
```

содержит распарсенный JSON.

### 6. Как обрабатывать ошибки?

Создаётся error-handling middleware с **четырьмя аргументами**:

```
app.use((err, req, res, next) => {
  res.status(500).json({
    message: err.message
  });
});
```

Важно поставить его после остальных middleware и роутов.

### 7. Что такое `req` и `res`?

`req` - объект входящего HTTP-запроса.

Содержит:

```
req.params
req.query
req.body
req.headers
```

`res` - объект ответа:

```
res.status(200).json(data);
```

### 8. Что такое `express.Router()`?

Позволяет разделять маршруты на отдельные модули.

```
const router = express.Router();

router.get('/users', getUsers);

app.use('/api', router);
```

Получаем:

```
GET /api/users
```

Это помогает структурировать большой проект.

### 9. Что такое CORS и как его настроить в Express?

**CORS** - механизм, который определяет, какие origin могут обращаться к серверу из браузера.

Например:

```
import cors from 'cors';

app.use(cors({
  origin: 'https://example.com'
}));
```

Важно понимать, что CORS - в первую очередь **ограничение браузера**, а не защита API от серверных запросов.

### 10. Как реализовать авторизацию в Express?

Обычно создают middleware, который проверяет токен или cookie:

```
app.use(authMiddleware);
```

Например:

```
Request
   ↓
authMiddleware
   ↓
проверка JWT
   ↓
next()
   ↓
Controller
```

Middleware проверяет пользователя и добавляет его данные в `req`, после чего запрос передаётся контроллеру.

### Что ещё могут спросить

Для Middle frontend-разработчика я бы отдельно подготовил **JWT + Express, cookies, CORS, CSRF, HTTP status codes, REST API, rate limiting и отличие middleware от interceptor** - эти темы часто идут глубже базовых вопросов.


### В чем отличие middleware от interceptor?

Главное отличие в том, **где и как они встроены в обработку запроса**.

### Middleware

Middleware - функция, которая выполняется **в цепочке обработки HTTP-запроса**:

```
Request
   ↓
Middleware
   ↓
Middleware
   ↓
Controller
   ↓
Response
```

В Express:

```
app.use((req, res, next) => {
  console.log('request');
  next();
});
```

Middleware может изменить `req`, `res`, проверить авторизацию, залогировать запрос и т.д.

### Interceptor

Interceptor - более высокоуровневая концепция. Он может перехватывать **и запрос до выполнения handler, и результат после него**:

```
Request
   ↓
Interceptor
   ↓
Controller
   ↓
Interceptor
   ↓
Response
```

Например, можно централизованно:

- преобразовать ответ
- добавить headers
- измерить время выполнения
- обработать результат
- логировать запрос и ответ

В **NestJS** interceptor выглядит примерно так:

```
@Injectable()
class LoggingInterceptor implements NestInterceptor {
  intercept(context, next) {
    console.log('before');

    return next.handle().pipe(
      tap(() => console.log('after'))
    );
  }
}
```

### Главное

В **Express** основная концепция - middleware.  
В **NestJS** есть middleware + guards + pipes + interceptors + filters.

Для собеседования:

> Middleware участвует в цепочке обработки HTTP-запроса и обычно передаёт управление дальше через `next()`. Interceptor оборачивает выполнение handler и позволяет выполнить логику как до его вызова, так и после получения результата.