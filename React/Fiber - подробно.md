## 1. Что такое React Fiber

**React Fiber - это внутренняя архитектура React, отвечающая за выполнение и планирование работы по обновлению UI.**

Fiber появился в React 16 и заменил старый reconciliation algorithm.

Главная идея Fiber - **разбить работу React на небольшие части и дать React возможность управлять их выполнением**.

Старый React в основном выполнял reconciliation как непрерывную синхронную работу:

```
render
  ↓
reconciliation
  ↓
обновление всего дерева
  ↓
commit
```

Если дерево большое, браузер мог долго не получать контроль обратно.

Fiber позволил React:

```
начать работу
   ↓
выполнить часть
   ↓
приостановиться
   ↓
дать браузеру поработать
   ↓
продолжить
```

Поэтому Fiber тесно связан с:

- concurrent rendering
- приоритетами обновлений
- `startTransition`
- `useTransition`
- Suspense
- scheduling
- reconciliation

---

# 2. Что такое Fiber Node

Fiber - это **объект, который описывает конкретный элемент или компонент React-дерева и состояние работы React над ним**.

Упрощенно Fiber можно представить так:

```
{
  type,
  key,
  props,
  stateNode,

  child,
  sibling,
  return,

  alternate,

  flags,
  subtreeFlags,

  lanes,
  childLanes
}
```

Например:

```
<App>
  <Header />
  <Content />
</App>
```

React создает Fiber-дерево примерно такого вида:

```
App
 ├── Header
 └── Content
```

Причем Fiber-дерево - это **не DOM-дерево**.

Fiber описывает работу React с компонентами, а DOM является результатом этой работы.

---

# 3. Связи между Fiber

У Fiber нет обычного массива `children`.

Основные связи:

```
child
sibling
return
```

Например:

```
<App>
  <Header />
  <Main />
  <Footer />
</App>
```

Можно представить:

```
        App
         |
       child
         ↓
      Header
         |
      sibling
         ↓
       Main
         |
      sibling
         ↓
      Footer
```

`return` указывает на родителя.

То есть:

```
Header.return → App
Main.return   → App
Footer.return → App
```

Такую структуру удобно использовать для обхода дерева без рекурсивного вызова функций.

---

# 4. Что такое reconciliation

**Reconciliation - процесс сравнения предыдущего React-дерева с новым и определения необходимых изменений.**

Например:

Было:

```
<div>
  <span>Hello</span>
</div>
```

Стало:

```
<div>
  <span>Hello World</span>
</div>
```

React понимает:

```
div тот же
span тот же
text изменился
```

И в DOM изменяется только текст.

---

# 5. Где здесь Fiber

Fiber представляет единицу работы React.

Условно:

```
Fiber App
   ↓
Fiber Header
   ↓
Fiber Content
   ↓
Fiber Button
```

React может обрабатывать эти Fiber по отдельности.

Поэтому вместо:

```
обработать всё дерево целиком
```

можно:

```
Fiber 1
↓
Fiber 2
↓
pause
↓
Fiber 3
↓
Fiber 4
```

Это фундаментальная идея Fiber.

---

# 6. Render Phase и Commit Phase

Это один из **самых важных вопросов на собеседовании**.

React условно разделяет обновление на две фазы.

## Render Phase

React:

- вызывает компоненты
- получает новый JSX
- выполняет reconciliation
- строит/обновляет Fiber
- определяет, какие изменения нужно сделать

Главное:

**Render Phase не должна иметь побочных эффектов.**

Например:

```
function Component() {
  console.log('render');

  return <div>Hello</div>;
}
```

`console.log` выполнится во время render phase.

---

## Commit Phase

После того как React закончил render phase, он применяет изменения.

Например:

```
Render Phase
    ↓
определили изменения
    ↓
Commit Phase
    ↓
изменили DOM
```

В commit phase React может:

- изменить DOM
- выполнить `useLayoutEffect`
- выполнить ref callbacks
- запланировать `useEffect`

---

# 7. Почему Render Phase может быть прервана

Это ключевое преимущество Fiber.

Допустим:

```
Render Phase
↓
Fiber A
↓
Fiber B
↓
Fiber C
```

React может остановиться между `B` и `C`.

Например, появился более приоритетный update.

```
Low priority update
        ↓
A → B → C
        ↓
      pause
        ↓
High priority update
        ↓
обработать его
```

После этого React может продолжить или вообще выбросить незавершенную работу и начать render заново.

**Commit Phase при этом не должна быть прервана таким же образом.**

Это важно:

> Render может быть interrupted, commit - нет.

---

# 8. Concurrent Rendering

Fiber стал фундаментом для Concurrent React.

Важно понимать:

**Concurrent Rendering не означает, что JavaScript действительно выполняется одновременно в нескольких потоках.**

JavaScript в браузере обычно выполняется в одном основном потоке.

Concurrent означает, что React может:

- прерывать работу
- продолжать ее позже
- менять приоритет
- выбрасывать незавершенный результат
- выбирать, какую работу выполнять первой

То есть это скорее:

```
cooperative scheduling
```

а не:

```
parallel execution
```

---

# 9. Приоритеты обновлений

Не все обновления одинаково важны.

Например:

```
setInputValue(value);
```

пользователь ожидает увидеть изменение практически сразу.

А вот:

```
setSearchResults(results);
```

может быть менее приоритетным.

Поэтому React может разделять работу по приоритетам.

Современный React использует систему **Lanes**.

---

# 10. Что такое Lanes

Lanes - внутренняя система React для представления **приоритетов и группировки обновлений**.

Можно представить:

```
Update A → lane 1
Update B → lane 2
Update C → lane 4
```

React смотрит на lanes и определяет, какую работу выполнять сейчас.

Упрощенно:

```
High priority
     ↓
обработать сейчас

Low priority
     ↓
можно отложить
```

Это позволяет React эффективнее планировать обновления.

---

# 11. `startTransition`

Пример:

```
const [input, setInput] = useState('');
const [results, setResults] = useState([]);

function handleChange(e) {
  const value = e.target.value;

  setInput(value);

  startTransition(() => {
    setResults(filterResults(value));
  });
}
```

Здесь:

```
setInput
```

- срочное обновление

А:

```
setResults
```

- transition update
- менее приоритетное

React может сначала обеспечить отзывчивость input, а затем заниматься тяжелым обновлением результатов.

---

# 12. `useTransition`

Позволяет дополнительно узнать, выполняется ли transition:

```
const [isPending, startTransition] = useTransition();
```

Например:

```
startTransition(() => {
  setPage(nextPage);
});
```

Можно показать:

```
{isPending && <Spinner />}
```

---

# 13. Что происходит при `setState`

Например:

```
setCount(count + 1);
```

Упрощенно:

```
setCount
   ↓
создается update
   ↓
update получает lane
   ↓
update попадает в очередь
   ↓
React планирует работу
   ↓
render phase
   ↓
reconciliation
   ↓
commit
   ↓
DOM обновляется
```

Важно:

**`setState` не означает "немедленно изменить DOM".**

Он сообщает React:

> состояние изменилось, нужно запланировать обновление.

---

# 14. Почему несколько setState могут объединяться

Например:

```
setCount(1);
setCount(2);
setCount(3);
```

React может обработать их в рамках одного render.

Это называется **batching**.

В современных версиях React automatic batching распространяется не только на React event handlers.

Например:

```
setTimeout(() => {
  setCount(1);
  setName('Oleg');
});
```

В современном React эти обновления также могут быть обработаны одним render.

---

# 15. Fiber и batching - не одно и то же

Это важно не путать.

**Fiber** отвечает за архитектуру и возможность планировать работу.

**Batching** - механизм группировки обновлений.

Они связаны, но это разные концепции.

---

# 16. `alternate`

У Fiber есть поле:

```
alternate
```

Оно связывает две версии Fiber.

Упрощенно:

```
Current Fiber Tree
        ↕
   alternate
        ↕
Work In Progress Tree
```

React использует две версии дерева:

```
current
workInProgress
```

---

# 17. Current и WorkInProgress

Допустим, сейчас приложение отображает:

```
Current Tree
```

React получает обновление.

Он начинает строить:

```
WorkInProgress Tree
```

Получается:

```
Current
   ↕
WIP
```

React может спокойно работать над WIP.

После завершения commit:

```
WIP
 ↓
становится Current
```

Это называют **double buffering**.

---

# 18. Почему нужен WorkInProgress

Представим, React начал обновление:

```
A
↓
B
↓
C
```

Но работа была прервана.

Пользователь при этом должен продолжать видеть старый UI.

Поэтому:

```
Current
↓
старый UI

WIP
↓
новая версия
```

React может работать с новой версией, не ломая текущий интерфейс.

---

# 19. Что такое Flags

Fiber содержит информацию о том, какие действия нужно выполнить во время commit.

Например, условно:

```
Placement
Update
Deletion
```

Например:

```
<div>Hello</div>
```

стал:

```
<div>Hello World</div>
```

Fiber получает информацию, что нужен update.

При commit React применяет это изменение к DOM.

---

# 20. Reconciliation и `key`

Fiber особенно важен при работе со списками.

Например:

```
users.map(user => (
  <User key={user.id} user={user} />
))
```

`key` помогает React понять:

```
это тот же Fiber
```

или:

```
это другой элемент
```

Например:

```
Было:

A
B
C

Стало:

X
A
B
C
```

С хорошими `key` React понимает:

```
X → новый
A → тот же
B → тот же
C → тот же
```

Без корректных `key` reconciliation может быть менее эффективным и приводить к неправильному сохранению состояния элементов.

---

# 21. Fiber и React.memo

`React.memo` может позволить React не выполнять повторный render компонента, если его props не изменились.

Например:

```
const User = memo(({ name }) => {
  return <div>{name}</div>;
});
```

Но `memo` не означает:

> Fiber вообще не существует.

Fiber всё равно есть.

Просто React может **bailout** и не выполнять ненужную работу для этой части дерева.

---

# 22. Bailout

Bailout - ситуация, когда React понимает:

> эту часть дерева можно не пересчитывать.

Например:

```
const Child = memo(ChildComponent);
```

Родитель перерендерился:

```
Parent
 ↓
Child
```

Но props `Child` не изменились.

React может сделать:

```
Parent → render
Child  → bailout
```

Это одна из важных оптимизаций reconciliation.

---

# 23. Fiber не делает React автоматически быстрым

Это частая ловушка на собеседовании.

Fiber дает React возможность:

- прерывать работу
- планировать работу
- менять приоритеты
- делать bailout
- эффективно выполнять concurrent rendering

Но если написать:

```
function App() {
  // тяжелые вычисления
  massiveCalculation();

  return <Component />;
}
```

Fiber не сделает `massiveCalculation()` магически быстрым.

Можно только получить возможность **не блокировать UI всей работой целиком**, если React может распределить работу подходящим образом.

---

# 24. Fiber и Event Loop

React работает внутри JavaScript runtime и взаимодействует с браузером через scheduling.

Упрощенная модель:

```
JavaScript
   ↓
React
   ↓
Fiber work
   ↓
scheduler
   ↓
браузер получает возможность обработать другие задачи
```

Но не стоит говорить на собеседовании:

> Fiber - это микротаска.

Это неправильно.

Fiber - **архитектура React**, а Scheduler - механизм планирования работы.

---

# 25. Fiber и Virtual DOM

Эти понятия тоже часто путают.

### Virtual DOM

Представление UI в JavaScript:

```
<div>Hello</div>
```

React использует React elements, чтобы описывать UI.

### Fiber

Внутренняя структура React, которая хранит информацию о:

- компоненте
- props
- state
- связях дерева
- pending updates
- приоритетах
- эффектах
- текущей работе

То есть:

```
React Element
      ↓
Fiber
      ↓
DOM
```

Очень грубая схема, но для понимания подходит.

---

# 26. Почему Fiber называют reconciliation architecture

Потому что Fiber - это не просто "оптимизация Virtual DOM".

Он изменил сам способ, которым React выполняет reconciliation.

Старый подход:

```
reconciliation
↓
длинная синхронная работа
```

Fiber:

```
reconciliation
↓
Fiber units
↓
scheduler
↓
priorities
↓
interruptible work
```

---

# 27. Что происходит при обновлении компонента

Например:

```
setCount(count + 1);
```

Упрощенно:

```
1. вызывается setCount
        ↓
2. создается update
        ↓
3. update получает priority lane
        ↓
4. React планирует render
        ↓
5. создается/обновляется WIP Fiber tree
        ↓
6. React вызывает компоненты
        ↓
7. сравнивает результат
        ↓
8. определяет изменения
        ↓
9. commit
        ↓
10. DOM обновляется
```

---

# 28. Что особенно важно запомнить

Если нужно ответить на собеседовании буквально за минуту:

> Fiber - это внутренняя архитектура React, которая представляет компоненты в виде Fiber-дерева и позволяет React разбивать render work на небольшие части. Благодаря этому React может прерывать и возобновлять render, назначать обновлениям разные приоритеты и делать bailout. Fiber лежит в основе Concurrent React, transitions и современного scheduling. При этом render phase может быть прервана, а commit phase применяется атомарно.

Это уже хороший ответ уровня Middle+.