Critical Rendering Path — это последовательность шагов, которые браузер выполняет для отображения страницы на экране.

Основная цель оптимизации CRP — как можно быстрее показать пользователю первый контент (First Paint / First Contentful Paint).

---

## Этапы Critical Rendering Path

1. Получение HTML
2. Парсинг HTML → построение DOM
3. Загрузка и парсинг CSS → построение CSSOM
4. Объединение DOM + CSSOM → Render Tree
5. Layout (Reflow) — вычисление размеров и позиций элементов
6. Paint — отрисовка пикселей
7. Composite — сборка слоёв на GPU

---

## Важные особенности

### CSS блокирует рендеринг

Браузер не может построить Render Tree без CSSOM.

```
<link rel="stylesheet" href="styles.css">
```

Пока CSS не загрузится — страница может не отображаться.

---

### JS может блокировать парсинг HTML

```
<script src="app.js"></script>
```

Браузер:

- останавливает парсинг HTML;
- загружает JS;
- выполняет JS;
- только потом продолжает.

Поэтому используют:

```
<script defer src="app.js"></script>
```

или

```
<script async src="app.js"></script>
```

---

## Что влияет на скорость CRP

- размер HTML/CSS/JS;
- количество blocking resources;
- синхронный JS;
- тяжёлый CSS;
- web fonts;
- количество reflow/repaint;
- network latency.

---

## Как оптимизируют CRP

### Минификация

- CSS
- JS
- HTML

---

### Code Splitting

Загружать только нужный JS.

```
import('./HeavyComponent')
```

---

### Lazy Loading

```
<img loading="lazy">
```

---

### Critical CSS

Инлайнить CSS для первого экрана.

---

### defer / async

```
<script defer src="app.js"></script>
```

---

### CDN и кеширование

Уменьшают время загрузки ресурсов.

---

## В React / SPA

CRP особенно важен, потому что:

- большой bundle может задерживать hydration;
- CSR показывает пустой экран до загрузки JS;
- поэтому используют SSR/SSG/streaming.

---

## Короткий ответ для собеседования

> Critical Rendering Path — это процесс преобразования HTML, CSS и JS в готовое изображение страницы на экране. Он включает построение DOM, CSSOM, Render Tree, Layout и Paint. Оптимизация CRP направлена на уменьшение blocking resources и ускорение First Paint/FCP.