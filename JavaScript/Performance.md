todo - переделать конспект

Performance

в JavaScript — это инструмент браузера для точного замера времени выполнения кода, отслеживания загрузки ресурсов и анализа производительности страницы. Он является частью **Performance API** и позволяет работать с высокоточным временем (до 5 микросекунд). [developer.mozilla.org](https://developer.mozilla.org/ru/docs/Web/API/Performance)[purpleschool.ru](https://purpleschool.ru/knowledge-base/javascript/browser-environment-and-api/performance)[doka.guide](https://doka.guide/js/performance/)[devdoc.net](https://devdoc.net/web/developer.mozilla.org/en-US/docs/Web/API/Performance_API.html)

## Как получить доступ

Объект этого интерфейса доступен через свойство

window.performance

(только для чтения). [developer.mozilla.org](https://developer.mozilla.org/ru/docs/Web/API/Performance)

## Основные методы и свойства

**Свойства:**

- **
    
    timing
    
    **— возвращает объект
    
    PerformanceTiming
    
    с информацией о задержках: время начала навигации, время обработки редиректов, время получения ответов и т.д..
- **
    
    navigation
    
    **— возвращает объект
    
    PerformanceNavigation
    
    , который описывает тип навигации (например, переход по ссылке или из истории браузера).

 [developer.mozilla.org](https://developer.mozilla.org/ru/docs/Web/API/Performance)[devdoc.net](https://devdoc.net/web/developer.mozilla.org/en-US/docs/Web/API/Performance_API.html)

**Методы:**

- **
    
    now()
    
    **— возвращает текущую высокоточную временную метку (в миллисекундах). [devdoc.net](https://devdoc.net/web/developer.mozilla.org/en-US/docs/Web/API/Performance_API.html)
- **
    
    mark(name)
    
    **— создаёт именованную метку в буфере производительности.
    
    name
    
    — уникальный идентификатор метки. [purpleschool.ru](https://purpleschool.ru/knowledge-base/javascript/browser-environment-and-api/performance)
- **
    
    measure(name, startName, endName)
    
    **— вычисляет разницу между двумя метками и создаёт запись измерения. [purpleschool.ru](https://purpleschool.ru/knowledge-base/javascript/browser-environment-and-api/performance)
- **
    
    getEntries()
    
    **— возвращает все записи из буфера производительности. [8host.com](https://www.8host.com/blog/nachalo-raboty-s-javascript-performance-api/)
- **
    
    getEntriesByType(type)
    
    **— фильтрует записи по типу (например,
    
    mark
    
    ,
    
    measure
    
    ,
    
    resource
    
    ). [8host.com](https://www.8host.com/blog/nachalo-raboty-s-javascript-performance-api/)
- **
    
    getEntriesByName(name, type)
    
    **— фильтрует записи по имени и типу. [8host.com](https://www.8host.com/blog/nachalo-raboty-s-javascript-performance-api/)
- **
    
    clearMarks()
    
    **— удаляет из буфера указанную метку. [developer.mozilla.org](https://developer.mozilla.org/ru/docs/Web/API/Performance)
- **
    
    clearMeasures()
    
    **и**
    
    clearResourceTimings()
    
    **— очищают соответствующие буферы. [developer.mozilla.org](https://developer.mozilla.org/ru/docs/Web/API/Performance)
- **
    
    toJSON()
    
    **— сериализует объект
    
    Performance
    
    в JSON. [devdoc.net](https://devdoc.net/web/developer.mozilla.org/en-US/docs/Web/API/Performance_API.html)

## Типы записей

В буфере производительности хранятся записи разных типов:

- **
    
    mark
    
    **— пользовательские метки, созданные через
    
    mark()
    
    .
- **
    
    measure
    
    **— измерения разницы между двумя метками, созданные через
    
    measure()
    
    .
- **
    
    resource
    
    **— записи о загрузке внешних ресурсов (CSS, скрипты, изображения).
- **
    
    navigation
    
    **— данные о навигации (время начала навигации, время до первого байта и т.д.).
- **
    
    paint
    
    **— события отрисовки пикселей на экране.
- **
    
    longtask
    
    **— задачи, выполнение которых занимает более 50 мс (выявляются PerformanceObserver).

 [8host.com](https://www.8host.com/blog/nachalo-raboty-s-javascript-performance-api/)[dev-notes.ru](https://www.dev-notes.ru/articles/frontend/performance-web-api-in-javascript/)

## Для чего это нужно

С помощью

Performance

можно:

- измерять время выполнения конкретных функций или участков кода; [purpleschool.ru](https://purpleschool.ru/knowledge-base/javascript/browser-environment-and-api/performance)
- анализировать время загрузки ресурсов и выявлять «узкие места»; [purpleschool.ru](https://purpleschool.ru/knowledge-base/javascript/browser-environment-and-api/performance)[dev-notes.ru](https://www.dev-notes.ru/articles/frontend/performance-web-api-in-javascript/)
- отслеживать ключевые метрики (Core Web Vitals); [cossa.ru](https://www.cossa.ru/promopult/345240/)
- проводить бенчмаркинг (сравнение производительности разных версий кода). [devdoc.net](https://devdoc.net/web/developer.mozilla.org/en-US/docs/Web/API/Performance_API.html)

## Важные нюансы

- Данные
    
    Performance
    
    доступны в основном потоке и в фоновых потоках (Web Workers), но есть ограничения: например, свойство
    
    navigation
    
    недоступно в Workers.
- Метки, созданные в одном контексте (потоке), не видны в другом.
- Для **непрерывного мониторинга** изменений производительности (например, отслеживания LCP — времени до первого байта) используется
    
    PerformanceObserver
    
    .

## Пример использования

// Создание меток
performance.mark('start');

// Код, время выполнения которого нужно измерить

performance.measure('myFunction', 'start', 'end');

// Получение записей
const entries = performance.getEntriesByType('measure');
console.log(entries.duration); // Время выполнения
``` 

В инструментах разработчика браузера (DevTools) есть вкладка «Производительность» (Performance), где можно записать профиль и наглядно проанализировать собранные данные. [9](https://doka.guide/js/performance/)


