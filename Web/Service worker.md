**Service Worker** это JavaScript-файл, который работает в фоне отдельно от страницы и может перехватывать сетевые запросы, управлять кэшем, поддерживать офлайн-режим и обрабатывать push-уведомления.
### Как работает

Типичный жизненный цикл:

```
Регистрация
    ↓
install
    ↓
waiting
    ↓
activate
    ↓
fetch / push / sync
```

Например, приложение регистрирует Service Worker:

```
navigator.serviceWorker.register('/sw.js');
```

После этого браузер загружает `sw.js` и устанавливает его.

### `install`

Используется для первоначального создания кэша:

```
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open('app-cache').then((cache) => {
      return cache.addAll([
        '/',
        '/index.html',
        '/styles.css'
      ]);
    })
  );
});
```

### `activate`

Вызывается после установки и используется, например, для удаления старых кэшей:

```
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.delete('old-cache')
  );
});
```

### `fetch`

Самая интересная часть. Service Worker может перехватывать HTTP-запросы:

```
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request)
      .then((cached) => cached || fetch(event.request))
  );
});
```

Теперь при запросе браузер сначала проверит Cache Storage. Если там есть ответ, его можно вернуть без обращения к серверу.

Например:

```
React приложение
      ↓
   fetch()
      ↓
Service Worker
   ↙       ↘
cache     network
```

### Что умеет Service Worker

- **Offline**: отдавать ранее закэшированные ресурсы без сети.
- **Caching**: управлять Cache Storage.
- **Перехватывать fetch**: изменять стратегию получения данных.
- **PWA**: является одной из основных технологий Progressive Web Apps.
- **Push notifications**: получать push-события в фоне.
- **Background Sync**: повторять операции после восстановления сети.

### Важный момент

Service Worker **не имеет доступа к DOM**:

```
document.querySelector(...)
```

внутри Service Worker не работает.

Он также работает в отдельном контексте и общается со страницей через механизмы вроде `postMessage`.

### Service Worker и обычный Web Worker

|Web Worker|Service Worker|
|---|---|
|Вычисления в отдельном потоке|Работа в фоне между приложением и сетью|
|Не управляет запросами страницы|Может перехватывать `fetch`|
|Живет пока нужен странице|Может работать независимо от открытой страницы|
|Нет Cache API как основной задачи|Активно используется Cache API|
|Для тяжелых вычислений|Для PWA, offline, cache, push|

**На собеседовании:** Service Worker это фоновый скрипт браузера, который работает независимо от страницы и позволяет перехватывать сетевые запросы, управлять кэшем, реализовывать offline-режим, PWA, push-уведомления и background sync.
