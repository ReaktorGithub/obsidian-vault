`node:util` - встроенный модуль Node.js, предоставляющий вспомогательные функции для работы с объектами, типами данных, промисами, отладки и форматирования вывода.

## 1. `util.inspect()`

Преобразует объект в строку для удобного просмотра его структуры. Используется при отладке.

```
const util = require('node:util');

const user = {
  name: 'Oleg',
  skills: ['React', 'Node.js']
};

console.log(util.inspect(user, { depth: null }));
```

`depth` задаёт глубину отображения вложенных объектов. Значение `null` позволяет показать всю структуру.

## 2. `util.promisify()`

Преобразует функцию с callback в функцию, возвращающую `Promise`.

Полезно при работе со старыми API Node.js, использующими колбэки.

```
const util = require('node:util');
const fs = require('node:fs');

const readFileAsync = util.promisify(fs.readFile);

async function main() {
  const data = await readFileAsync('file.txt', 'utf8');
  console.log(data);
}

main();
```

Вместо передачи callback можно использовать `await`.

**Важно:** многие современные API Node.js уже поддерживают промисы самостоятельно. Например, `node:fs/promises`, поэтому преобразование не всегда необходимо.

## 3. `util.callbackify()`

Выполняет обратное преобразование: превращает функцию, возвращающую `Promise`, в функцию с callback.

```
const util = require('node:util');

async function getUser() {
  return { name: 'Oleg' };
}

const getUserCallback = util.callbackify(getUser);

getUserCallback((err, user) => {
  if (err) throw err;

  console.log(user);
});
```

Функция с callback принимает ошибку первым аргументом, а результат - вторым.

## 4. `util.isDeepStrictEqual()`

Сравнивает два значения на глубокое строгое равенство, включая вложенные свойства объектов.

```
const util = require('node:util');

const a = { user: { id: 1 } };
const b = { user: { id: 1 } };

console.log(util.isDeepStrictEqual(a, b)); // true
```

Обычное сравнение `a === b` вернёт `false`, потому что объекты находятся в разных областях памяти.

## 5. `util.format()`

Форматирует строку с использованием спецификаторов.

```
const util = require('node:util');

const message = util.format(
  'User: %s, age: %d',
  'Oleg',
  25
);

console.log(message);
// User: Oleg, age: 25
```

Основные спецификаторы:

- `%s` - строка.
- `%d` - число.
- `%i` - целое число.
- `%f` - число с плавающей точкой.
- `%j` - JSON-представление.
- `%o` - объект с дополнительными сведениями.

## 6. `util.debuglog()`

Создаёт функцию для вывода отладочных сообщений, которые включаются через переменную окружения `NODE_DEBUG`.

```
const util = require('node:util');

const debug = util.debuglog('app');

debug('Application started');
```

В Windows PowerShell можно включить вывод для категории `app` так:

```
$env:NODE_DEBUG="app"
node index.js
```

Без включения соответствующей категории отладочные сообщения не выводятся.

## 7. `util.types`

Содержит функции для проверки типов объектов, включая специальные встроенные типы JavaScript.

```
const { types } = require('node:util');

console.log(types.isDate(new Date())); // true
console.log(types.isPromise(Promise.resolve())); // true
console.log(types.isRegExp(/abc/)); // true
```

Полезно, когда требуется определить конкретный тип объекта, а обычного `typeof` недостаточно.

Например:

```
console.log(typeof null); // "object"
```

При этом `types.isDate()`, `types.isPromise()` и другие методы позволяют проверять конкретные типы.

## 8. `util.deprecate()`

Создаёт обёртку для устаревшей функции. При её вызове Node.js может выводить предупреждение о необходимости использовать альтернативу.

```
const util = require('node:util');

const oldFunction = util.deprecate(
  () => console.log('Old API'),
  'Use newFunction instead'
);

oldFunction();
```

Предупреждение помогает обнаружить использование устаревшего API.

---

## Краткая шпаргалка для Obsidian

|Метод|Назначение|
|---|---|
|`util.inspect()`|Представление объектов в виде строки для отладки|
|`util.promisify()`|Преобразование callback-функции в Promise API|
|`util.callbackify()`|Преобразование Promise-функции в callback API|
|`util.isDeepStrictEqual()`|Глубокое строгое сравнение значений|
|`util.format()`|Форматирование строк|
|`util.debuglog()`|Вывод отладочных сообщений по категориям|
|`util.types`|Проверка специальных типов объектов|
|`util.deprecate()`|Пометка функций как устаревших|

**Для собеседования:** в первую очередь стоит знать `promisify()`, `callbackify()`, `inspect()` и `isDeepStrictEqual()`. Они показывают понимание работы с асинхронностью, отладкой и объектами в Node.js.
