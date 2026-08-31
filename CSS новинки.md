Короткая шпаргалка по современному CSS:

- **Container Queries** - стилизация элемента в зависимости от размера его контейнера, а не viewport.  
    `@container (min-width: 500px)`
- **`:has()`** - позволяет выбирать элемент на основе его потомков или соседей. По сути, «родительский селектор».  
    `.card:has(img)`
- **CSS Nesting** - вложенность селекторов прямо в CSS, как в SCSS.  
    `.card { & .title { ... } }`
- **Cascade Layers `@layer`** - управление приоритетом групп CSS-правил.  
    `@layer reset, components, utilities`
- **`@scope`** - ограничивает область действия CSS-селекторов конкретным элементом.
- **`subgrid`** - позволяет дочерним элементам использовать сетку родительского `grid`.
- **`color-mix()`** - смешивание цветов прямо в CSS.  
    `color-mix(in srgb, red 30%, blue)`
- **`light-dark()`** - позволяет задавать разные значения для светлой и тёмной темы.
- **Custom Properties** - CSS-переменные стали гораздо мощнее и активно используются вместе с `var()`, `calc()`, `clamp()`.
- **`clamp()`** - адаптивное значение между минимумом и максимумом.  
    `font-size: clamp(16px, 2vw, 24px)`
- **Logical Properties** - `margin-inline`, `padding-block`, `inset-inline` вместо привязки к `left`, `right`, `top`, `bottom`.
- **View Transitions API** - плавные переходы между состояниями и страницами без сложной ручной анимации.
- **`@starting-style`** - позволяет задавать начальные стили для элементов, например при появлении через `display: none`.
- **`text-wrap: balance`** - автоматически балансирует строки заголовков.  
    `text-wrap: balance`
- **`field-sizing: content`** - позволяет `input` и `textarea` автоматически подстраиваться под содержимое.
- **`interpolate-size`** - позволяет анимировать размеры, включая переходы вроде `height: auto`.

На собеседовании я бы особенно выделил **`:has()`, Container Queries, Nesting, Cascade Layers, subgrid и View Transitions** - это наиболее заметные современные возможности CSS.
