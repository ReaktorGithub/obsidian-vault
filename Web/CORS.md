### Короткий ответ на собеседовании

> CORS - это механизм браузера, который срабатывает при cross-origin запросах и проверяет, разрешил ли сервер такой origin. Для простых запросов браузер сразу отправляет запрос, а для более сложных сначала делает preflight OPTIONS и проверяет `Access-Control-Allow-Origin`, методы и заголовки.

CORS - это встроенная защита браузера от запросов между разными origin. Настраивается на сервере через HTTP-заголовки и определяет, с каких origin разрешены запросы к ресурсу.

Например:

```
https://frontend.com
        ↓ fetch
https://api.com
```

Это cross-origin запрос, потому что отличаются origin.

### Когда CORS активируется?

Когда браузер делает **cross-origin запрос** из страницы.

Важно: **CORS - это механизм браузера**, а не серверной защиты в целом. Сервер может отвечать на запрос, но браузер может запретить JavaScript прочитать этот ответ, если CORS-заголовки не разрешают доступ.

### Same-Origin Policy

Это базовое правило безопасности браузера:

> Скрипт с одного origin не может свободно получать данные с другого origin.

Origin определяется:

```
protocol + host + port
```

Например:

```
https://example.com:443
```

и

```
http://example.com:443
```

имеют разные origin из-за протокола.

---

### Простые запросы

Есть запросы, которые браузер может отправить **без предварительного OPTIONS-запроса**. Их называют simple requests.

Чтобы запрос считался простым, одновременно выполняются условия:

1. Метод: **`GET`, `HEAD` или `POST`**.
2. Заголовки - только разрешённые CORS **CORS-safelisted** заголовки, например `Accept`, `Accept-Language`, `Content-Language`, `Content-Type`.
3. Если есть `Content-Type`, он должен быть одним из:
    - `application/x-www-form-urlencoded`
    - `multipart/form-data`

Например:

```
fetch('https://api.example.com/users', {
  method: 'GET'
})
```

Если запрос соответствует требованиям simple request, браузер сразу отправляет его серверу.

Но сервер всё равно должен вернуть правильный CORS-заголовок:

```
Access-Control-Allow-Origin: https://frontend.com
```

Тогда браузер позволит JavaScript получить ответ.

---

### Preflight

Если запрос потенциально более опасный или не соответствует требованиям simple request, браузер сначала делает **preflight** - запрос `OPTIONS`.

Например:

```
fetch('https://api.example.com/users', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer token',
    'Content-Type': 'application/json'
  }
})
```

Сначала браузер может отправить:

```
OPTIONS /users
Origin: https://frontend.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Authorization, Content-Type
```

Сервер отвечает, например:

```
Access-Control-Allow-Origin: https://frontend.com
Access-Control-Allow-Methods: POST, GET
Access-Control-Allow-Headers: Authorization, Content-Type
```

Если сервер разрешил такой запрос, браузер отправляет настоящий `POST`.

---

### `Access-Control-Allow-Origin`

Это один из основных CORS-заголовков:

```
Access-Control-Allow-Origin: https://frontend.com
```

Он говорит браузеру:

> Запросы от этого origin разрешены, и JavaScript может прочитать ответ.

Можно разрешить любой origin:

```
Access-Control-Allow-Origin: *
```

Но `*` нельзя использовать вместе с credentials, например cookie.

