**`fs` (File System)** - встроенный модуль Node.js для работы с файловой системой. Он позволяет читать, создавать, изменять, удалять файлы и директории, получать информацию о файлах и отслеживать изменения в них.

Например, с помощью `fs` можно читать конфигурацию приложения, сохранять загруженные файлы, формировать отчёты и записывать логи.

## 1. Подключение модуля

Модуль `fs` поддерживает два основных подхода: CommonJS и ES Modules.

**CommonJS:**

```
const fs = require('node:fs');
```

**ES Modules:**

```
import fs from 'node:fs';
```

Существует также модуль `fs/promises`, предоставляющий Promise-based API для асинхронной работы:

```
const fs = require('node:fs/promises');
```

```
import fs from 'node:fs/promises';
```

Основные способы работы с файлами:

- **Синхронный API** - операция блокирует основной поток JavaScript до завершения.
- **Асинхронный API с callbacks** - операция выполняется без блокировки основного потока, результат передаётся в callback.
- **Promise API** - асинхронная операция возвращает Promise, который можно использовать с `async/await`.

## 2. Чтение файлов

### Синхронное чтение

Метод `readFileSync()` читает файл и возвращает его содержимое.

```
const fs = require('node:fs');

const data = fs.readFileSync('config.json', 'utf-8');

console.log(data);
```

Второй аргумент `'utf-8'` указывает кодировку. Если её не указать, метод вернёт объект `Buffer`.

**Важно:** синхронное чтение блокирует основной поток на время операции. В серверном приложении это может задержать обработку других запросов.

### Асинхронное чтение через callback

```
const fs = require('node:fs');

fs.readFile('config.json', 'utf-8', (err, data) => {
  if (err) {
    console.error('Ошибка чтения:', err);
    return;
  }

  console.log(data);
});

console.log('Этот код выполнится, пока файл читается');
```

Метод `readFile()` запускает операцию чтения, а после её завершения вызывает callback.

Пока файл читается, Node.js может выполнять другой JavaScript-код и обрабатывать другие задачи.

### Асинхронное чтение через Promise

На практике часто используют `fs/promises` вместе с `async/await`.

```
const fs = require('node:fs/promises');

async function readConfig() {
  try {
    const data = await fs.readFile('config.json', 'utf-8');
    console.log(data);
  } catch (err) {
    console.error('Ошибка чтения:', err);
  }
}

readConfig();
```

`await` приостанавливает выполнение текущей `async`-функции до завершения операции, но не блокирует основной поток JavaScript.

**Когда использовать:** для серверных приложений обычно предпочтительнее асинхронные методы, поскольку они не блокируют Event Loop на время файловой операции.

---

## 3. Запись файлов

### `writeFile()` - создание или перезапись файла

```
const fs = require('node:fs/promises');

async function saveData() {
  await fs.writeFile('data.txt', 'Привет, Node.js!', 'utf-8');
}

saveData();
```

Если файла нет, он будет создан. Если файл существует, его содержимое по умолчанию будет перезаписано.

### Добавление содержимого: `appendFile()`

```
const fs = require('node:fs/promises');

async function writeLog() {
  await fs.appendFile('app.log', 'Приложение запущено\n', 'utf-8');
}

writeLog();
```

`appendFile()` добавляет данные в конец файла, а не заменяет существующее содержимое.

Такой подход можно использовать для простого логирования, хотя для production-приложений обычно применяют специализированные библиотеки логирования.

### Запись JSON

```
const fs = require('node:fs/promises');

async function saveConfig() {
  const config = {
    port: 3000,
    environment: 'development',
  };

  await fs.writeFile(
    'config.json',
    JSON.stringify(config, null, 2),
    'utf-8'
  );
}

saveConfig();
```

`JSON.stringify()` преобразует JavaScript-объект в строку JSON, которую можно сохранить в файле.

Чтобы прочитать такой файл и получить объект:

```
const fs = require('node:fs/promises');

async function loadConfig() {
  const data = await fs.readFile('config.json', 'utf-8');
  const config = JSON.parse(data);

  console.log(config.port);
}

loadConfig();
```

Обрати внимание: `JSON.parse()` может выбросить исключение, если содержимое файла не является корректным JSON.

---

## 4. Удаление и переименование файлов

### Удаление: `unlink()`

```
const fs = require('node:fs/promises');

async function deleteFile() {
  await fs.unlink('old.txt');
}

deleteFile();
```

Метод удаляет файл. Если файл не существует или нет прав на удаление, Promise будет отклонён.

### Переименование: `rename()`

```
const fs = require('node:fs/promises');

async function renameFile() {
  await fs.rename('old.txt', 'new.txt');
}

renameFile();
```

Метод переименовывает файл или перемещает его в другую директорию в пределах возможностей файловой системы.

---

## 5. Работа с директориями

### Создание директории: `mkdir()`

```
const fs = require('node:fs/promises');

async function createFolder() {
  await fs.mkdir('uploads', { recursive: true });
}

createFolder();
```

Параметр `recursive: true` позволяет создать вложенные директории, если их ещё нет.

Например:

```
await fs.mkdir('uploads/images/avatars', { recursive: true });
```

### Чтение содержимого директории: `readdir()`

```
const fs = require('node:fs/promises');

async function listFiles() {
  const files = await fs.readdir('uploads');

  console.log(files);
}

listFiles();
```

Результатом будет массив имён файлов и поддиректорий.

Чтобы получить информацию о каждом элементе, можно использовать параметр `withFileTypes`:

```
const entries = await fs.readdir('uploads', {
  withFileTypes: true,
});

for (const entry of entries) {
  if (entry.isFile()) {
    console.log('Файл:', entry.name);
  } else if (entry.isDirectory()) {
    console.log('Директория:', entry.name);
  }
}
```

### Удаление директории: `rm()`

```
const fs = require('node:fs/promises');

async function removeFolder() {
  await fs.rm('uploads', {
    recursive: true,
    force: true,
  });
}

removeFolder();
```

- `recursive: true` позволяет удалить директорию вместе с содержимым.
- `force: true` позволяет не выбрасывать ошибку, если путь не существует.

**Внимание:** при использовании `recursive: true` можно удалить целое дерево файлов. Перед удалением пользовательских данных нужно проверять путь и права доступа.

---

## 6. Получение информации о файлах

Метод `stat()` возвращает метаданные файла или директории.

```
const fs = require('node:fs/promises');

async function getFileInfo() {
  const stats = await fs.stat('data.txt');

  console.log('Размер:', stats.size);
  console.log('Создан:', stats.birthtime);
  console.log('Изменён:', stats.mtime);
  console.log('Это файл:', stats.isFile());
  console.log('Это директория:', stats.isDirectory());
}

getFileInfo();
```

Основные свойства и методы:

|Свойство или метод|Назначение|
|---|---|
|`size`|Размер в байтах|
|`birthtime`|Время создания, если поддерживается файловой системой|
|`mtime`|Время последнего изменения содержимого|
|`isFile()`|Проверяет, является ли объект обычным файлом|
|`isDirectory()`|Проверяет, является ли объект директорией|
|`isSymbolicLink()`|Проверяет, является ли объект символической ссылкой|

### `stat()` и `lstat()`

- `stat()` возвращает информацию об объекте, на который указывает символическая ссылка.
- `lstat()` возвращает информацию о самой символической ссылке.

Это важно, когда приложение работает с пользовательскими файлами и ссылками на другие пути.

---

## 7. Проверка существования файла

В старых примерах можно встретить `existsSync()`, но для асинхронного кода обычно лучше использовать обработку ошибок или `access()`.

```
const fs = require('node:fs/promises');

async function checkFile() {
  try {
    await fs.access('config.json');
    console.log('Файл доступен');
  } catch {
    console.log('Файл отсутствует или недоступен');
  }
}

checkFile();
```

Однако `access()` не гарантирует, что последующая операция чтения или записи обязательно завершится успешно: между проверкой и использованием файла его состояние может измениться.

Поэтому зачастую лучше сразу выполнять нужную операцию и обрабатывать ошибку.

```
const fs = require('node:fs/promises');

async function readConfig() {
  try {
    return await fs.readFile('config.json', 'utf-8');
  } catch (err) {
    if (err.code === 'ENOENT') {
      console.log('Файл не найден');
      return null;
    }

    throw err;
  }
}
```

`ENOENT` означает, что указанный файл или компонент пути не найден.

---

## 8. Потоки: `createReadStream()` и `createWriteStream()`

Методы `readFile()` и `writeFile()` удобны, когда нужно обработать файл целиком. Но если файл большой, чтение всего содержимого в память может потребовать слишком много оперативной памяти.

Для таких случаев используют потоки (Streams).

### Чтение файла потоком

```
const fs = require('node:fs');

const stream = fs.createReadStream('large-file.txt', {
  encoding: 'utf-8',
});

stream.on('data', (chunk) => {
  console.log('Получена часть файла:', chunk);
});

stream.on('end', () => {
  console.log('Чтение завершено');
});

stream.on('error', (err) => {
  console.error('Ошибка:', err);
});
```

Файл читается частями, называемыми `chunks`, вместо загрузки всего содержимого в память.

### Запись через поток

```
const fs = require('node:fs');

const stream = fs.createWriteStream('output.txt');

stream.write('Первая строка\n');
stream.write('Вторая строка\n');

stream.end();
```

`createWriteStream()` позволяет постепенно записывать данные в файл.

Если поток используется для записи большого объёма данных, необходимо учитывать механизм backpressure: если данные поступают быстрее, чем записываются, производитель должен замедлиться. Для передачи данных между потоками обычно удобно использовать `pipe()` или `pipeline()`.

### Пример копирования большого файла

```
const fs = require('node:fs');
const { pipeline } = require('node:stream/promises');

async function copyFile() {
  await pipeline(
    fs.createReadStream('large-file.txt'),
    fs.createWriteStream('copy.txt')
  );
}

copyFile().catch(console.error);
```

Здесь данные передаются между потоками, а `pipeline()` корректно обрабатывает ошибки и завершение передачи.

---

## 9. Отслеживание изменений файлов

Модуль `fs` предоставляет метод `watch()`, который позволяет отслеживать изменения файлов и директорий.

```
const fs = require('node:fs');

const watcher = fs.watch('config.json', (eventType, filename) => {
  console.log('Событие:', eventType);
  console.log('Файл:', filename);
});
```

`eventType` обычно содержит `rename` или `change`.

Например, этот механизм можно использовать для автоматической перезагрузки конфигурации при изменении файла.

Важно учитывать, что поведение `fs.watch()` зависит от операционной системы и файловой системы. События могут объединяться или не приходить в некоторых условиях, поэтому это не универсальный механизм гарантированной доставки всех изменений.

---

## 10. Синхронные и асинхронные методы

|Синхронный метод|Асинхронный метод|Назначение|
|---|---|---|
|`readFileSync()`|`readFile()`|Чтение файла|
|`writeFileSync()`|`writeFile()`|Запись файла|
|`appendFileSync()`|`appendFile()`|Добавление данных|
|`mkdirSync()`|`mkdir()`|Создание директории|
|`readdirSync()`|`readdir()`|Чтение директории|
|`statSync()`|`stat()`|Получение информации|
|`unlinkSync()`|`unlink()`|Удаление файла|
|`renameSync()`|`rename()`|Переименование|

Асинхронные методы без суффикса `Sync` в основном API обычно используют callback, а методы `fs/promises` возвращают Promise.

**Почему это важно для производительности?**

Предположим, Node.js-сервер обрабатывает запросы от множества пользователей. Если при каждом запросе выполнять `readFileSync()`, основной поток будет ждать завершения чтения файла. В это время Event Loop не сможет выполнять другие callbacks JavaScript.

Асинхронный `readFile()` позволяет не блокировать основной поток на время операции ввода-вывода.

При этом асинхронность не означает, что операция бесплатна: она всё равно использует ресурсы операционной системы, а некоторые операции файловой системы выполняются через пул потоков libuv.

---

## 11. Практический пример: сохранение загруженных данных

Представим, что backend получает данные и должен сохранить их в JSON-файл.

```
const fs = require('node:fs/promises');
const path = require('node:path');

async function saveUser(user) {
  const directory = path.resolve('data');
  const filePath = path.join(directory, `${user.id}.json`);

  await fs.mkdir(directory, { recursive: true });

  await fs.writeFile(
    filePath,
    JSON.stringify(user, null, 2),
    'utf-8'
  );

  return { success: true };
}

saveUser({
  id: 123,
  name: 'Oleg',
}).catch(console.error);
```

В этом примере:

1. `path.resolve()` формирует абсолютный путь к директории.
2. `path.join()` создаёт путь к файлу пользователя.
3. `mkdir()` создаёт директорию, если её нет.
4. `writeFile()` сохраняет данные в формате JSON.

В реальном приложении нужно дополнительно проверять входные данные и контролировать формирование путей. Например, нельзя без проверки использовать произвольный пользовательский ввод в имени файла: это может привести к выходу за пределы разрешённой директории.

Также файловая система не заменяет полноценную базу данных, если требуются конкурентные обновления, транзакции и сложные запросы.

---

## 12. Что важно знать для собеседования

**1. Что такое `fs`?**

Встроенный модуль Node.js для работы с файловой системой: чтения, записи, удаления файлов, создания директорий и получения метаданных.

**2. Чем отличаются `readFile()` и `readFileSync()`?**

`readFile()` выполняет чтение асинхронно и вызывает callback после завершения операции. `readFileSync()` блокирует основной поток до завершения чтения.

**3. Чем отличаются `fs` и `fs/promises`?**

`fs` предоставляет callback-based и синхронные методы, а `fs/promises` - Promise-based API, позволяющий использовать `async/await`.

**4. Когда использовать потоки?**

Когда нужно обрабатывать большие файлы или передавать данные частями, не загружая всё содержимое в память.

**5. Как обработать ошибку при чтении файла?**

В callback API проверять аргумент `err`, а в Promise API использовать `try/catch`.

**6. Что такое `Buffer`?**

Объект для работы с бинарными данными. Если читать файл без указания кодировки, `readFile()` возвращает `Buffer`, а не строку.

**7. Блокирует ли `await fs.readFile()` Event Loop?**

Нет. `await` приостанавливает выполнение текущей асинхронной функции, но не блокирует основной поток JavaScript на время файловой операции.

---

### Краткая шпаргалка

|Задача|Метод|
|---|---|
|Прочитать файл|`fs.readFile()`|
|Записать или перезаписать файл|`fs.writeFile()`|
|Добавить данные в конец|`fs.appendFile()`|
|Удалить файл|`fs.unlink()`|
|Переименовать файл|`fs.rename()`|
|Создать директорию|`fs.mkdir()`|
|Прочитать директорию|`fs.readdir()`|
|Получить метаданные|`fs.stat()`|
|Прочитать большой файл частями|`fs.createReadStream()`|
|Записать большой объём данных потоком|`fs.createWriteStream()`|
|Отслеживать изменения|`fs.watch()`|

**Главное правило:** для обычных файловых операций в серверном Node.js-коде предпочитай асинхронный API или `fs/promises`. Синхронные методы подходят для отдельных сценариев, например простых скриптов, CLI-инструментов и первоначальной загрузки конфигурации до начала обработки запросов.