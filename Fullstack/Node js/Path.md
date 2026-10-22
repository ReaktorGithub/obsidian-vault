**`path`** - встроенный модуль Node.js для работы с путями к файлам и директориям. Он позволяет объединять пути, извлекать имена файлов и расширения, а также нормализовать пути с учётом особенностей операционной системы.

### Основные методы

**1. `path.join()` - объединение частей пути**

```
const path = require('node:path');

const filePath = path.join('src', 'components', 'App.tsx');

console.log(filePath);
// Windows: src\components\App.tsx
// Linux/macOS: src/components/App.tsx
```

Объединяет части пути и нормализует разделители.

**2. `path.resolve()` - получение абсолютного пути**

```
const path = require('node:path');

console.log(path.resolve('src', 'index.js'));
// Например: C:\project\src\index.js
```

Преобразует путь в абсолютный относительно текущей рабочей директории. Фактический результат зависит от окружения.

**3. `path.basename()` - получение имени файла**

```
console.log(path.basename('/app/src/index.js'));
// index.js
```

Можно также получить имя без расширения:

```
console.log(path.basename('/app/src/index.js', '.js'));
// index
```

**4. `path.dirname()` - получение директории файла**

```
console.log(path.dirname('/app/src/index.js'));
// /app/src
```

**5. `path.extname()` - получение расширения файла**

```
console.log(path.extname('index.js'));
// .js

console.log(path.extname('photo.png'));
// .png
```

**6. `path.parse()` - разбор пути на составляющие**

```
console.log(path.parse('/app/src/index.js'));
```

Результат:

```
{
  root: '/',
  dir: '/app/src',
  base: 'index.js',
  ext: '.js',
  name: 'index'
}
```

**7. `path.isAbsolute()` - проверка абсолютного пути**

```
console.log(path.isAbsolute('/app/index.js'));
// true

console.log(path.isAbsolute('./index.js'));
// false
```

### Практический пример

Допустим, нужно сформировать путь к файлу конфигурации относительно директории текущего модуля:

```
const path = require('node:path');

const configPath = path.join(__dirname, 'config', 'app.json');

console.log(configPath);
```

Здесь `__dirname` содержит путь к директории текущего CommonJS-модуля, а `path.join()` добавляет к нему `config/app.json`.

В ES-модулях (`import`) переменная `__dirname` по умолчанию недоступна.

### Чем отличаются `path.join()` и `path.resolve()`?

- `path.join()` объединяет сегменты пути и нормализует результат.
- `path.resolve()` строит абсолютный путь, разрешая сегменты справа налево до получения абсолютного пути.

```
path.join('/app', 'src', '../index.js');
// /app/index.js

path.resolve('/app', 'src', '../index.js');
// /app/index.js
```

В этом примере результаты совпадают, но если передать относительные сегменты без начального абсолютного пути, `resolve()` дополнит путь текущей рабочей директорией.

### Коротко для собеседования

`path` - встроенный модуль Node.js для кроссплатформенной работы с файловыми путями. Чаще всего использую `join()` для объединения сегментов, `resolve()` для получения абсолютного пути, `basename()` для имени файла, `dirname()` для директории и `extname()` для расширения.


path.sep - разделитель в ОС
