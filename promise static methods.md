
# Promise.resolve()

Создаёт успешно завершённый Promise.

```
Promise.resolve(42)
```

Эквивалент:

```
new Promise(res => res(42))
```

---

# Promise.reject()

Создаёт отклонённый Promise.

```
Promise.reject(new Error('fail'))
```

---

# Promise.all()

Ждёт успешного выполнения ВСЕХ промисов.

```
Promise.all([  fetchUser(),  fetchPosts()])
```

Если один упадёт → весь `all` reject.

---

## Особенности

- порядок результатов сохраняется
- выполняются параллельно
- fail-fast

---

# Promise.allSettled()

Ждёт завершения ВСЕХ промисов:

- success
- reject

Никогда не падает.

```
Promise.allSettled([  p1,  p2,  p3])
```

Результат:

```
[  { status: 'fulfilled', value: ... },  { status: 'rejected', reason: ... }]
```

---

# Promise.race()

Возвращает результат первого завершившегося промиса:

- success
- reject

```
Promise.race([  fetchData(),  timeoutPromise()])
```

Часто используют для timeout.

---

# Promise.any()

Возвращает первый УСПЕШНЫЙ промис.

```
Promise.any([  api1(),  api2(),  api3()])
```

Игнорирует reject'ы, пока не упадут все.

Если все rejected:

- кидает `AggregateError`

---

# Promise.withResolvers() (новый)

Современный API.

```
const {  promise,  resolve,  reject} = Promise.withResolvers();
```

Позволяет получить:

- promise
- resolve
- reject

без внешних переменных.

---

## Старый способ

```
let resolve;const promise = new Promise(res => {  resolve = res;});
```

---

# Сравнение

|Метод|Что делает|
|---|---|
|Promise.resolve|successful promise|
|Promise.reject|rejected promise|
|Promise.all|все должны succeed|
|Promise.allSettled|ждёт все результаты|
|Promise.race|первый завершившийся|
|Promise.any|первый successful|
|Promise.withResolvers|создаёт promise + handlers|

---

# Частые практические кейсы

## Promise.all

- параллельные API запросы
- preload данных

---

## Promise.allSettled

- batch операции
- частично успешные результаты

---

## Promise.race

- timeout
- fallback server

---

## Promise.any

- CDN fallback
- multiple mirrors

---

## Promise.withResolvers

- event-driven architecture
- bridges between callbacks/promises
- deferred pattern
