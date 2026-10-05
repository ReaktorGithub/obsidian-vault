### Web Worker

**Web Worker** это JavaScript-скрипт, который выполняется в отдельном потоке браузера и позволяет выполнять тяжелые вычисления, не блокируя основной UI-поток.

Например:

```
// main.js
const worker = new Worker('/worker.js');

worker.postMessage(100000000);

worker.onmessage = (event) => {
  console.log(event.data);
};
```

```
// worker.js
self.onmessage = (event) => {
  const result = heavyCalculation(event.data);

  self.postMessage(result);
};
```

### Зачем нужен

Если выполнить тяжелый код в основном потоке:

```
const result = heavyCalculation();
```

то браузер может перестать реагировать на UI:

```
Main Thread
├── React
├── DOM
├── обработка событий
└── тяжелые вычисления ← блокируют всё
```

С Web Worker:

```
Main Thread              Web Worker
├── React                 ├── тяжелые вычисления
├── DOM                   └── обработка данных
└── UI
       ↕
  postMessage()
```

### Важные особенности

- Работает в **отдельном потоке**.
- Не имеет прямого доступа к **DOM**.
- Общается с основным потоком через `postMessage`.
- Хорош для тяжелых вычислений, парсинга больших данных, обработки изображений и других CPU-intensive задач.
- Создание Worker тоже имеет стоимость, поэтому для небольших вычислений он обычно не нужен.

**На собеседовании:** Web Worker позволяет вынести тяжелые JavaScript-вычисления из main thread в отдельный поток, чтобы не блокировать UI и обработку пользовательских событий.