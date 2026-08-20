## CommonJS (CJS)

Старая модульная система Node.js.

```
const fs = require('fs');module.exports = myFunction;
```

### Особенности

- синхронная загрузка
- используется в старом Node.js
- `require`
- `module.exports`
- загружается во время выполнения
- dynamic import проще

---

## ES Modules (ESM)

Стандартная модульная система JavaScript.

```
import fs from 'fs';export default myFunction;
```

### Особенности

- статический анализ импортов
- tree shaking
- async loading
- работает в браузере нативно
- лучше для bundlers
- strict mode по умолчанию

---

# Главное отличие

|CommonJS|ES Modules|
|---|---|
|require()|import|
|module.exports|export|
|sync loading|async/module graph|
|dynamic runtime|static analysis|
|old Node ecosystem|современный стандарт|
|хуже tree shaking|лучше tree shaking|

---

# Важный нюанс

## CJS

```
const mod = require(path)
```

Можно вызывать где угодно.

---

## ESM

```
import mod from './mod.js'
```

Импорты должны быть:

- на верхнем уровне
- анализируемыми заранее

---

# Node.js

В Node.js ESM включается через:

```
{  "type": "module"}
```

или `.mjs`.

---

# Коротко

- CommonJS — старый Node.js формат
- ES Modules — современный стандарт JS
- ESM лучше для frontend, bundlers и tree shaking
- CJS до сих пор часто встречается в legacy/npm ecosystem
- 