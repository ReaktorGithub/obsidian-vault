**React Server Components (RSC)** — это компоненты, которые выполняются **на сервере**, а их результат передаётся клиенту в специальном формате React. Их JavaScript-код не попадает в клиентский bundle.

В **Next.js App Router** компоненты по умолчанию являются Server Components:

```
// Server Component
export default async function Page() {
  const data = await getData()

  return <div>{data.title}</div>
}
```

Они могут напрямую получать данные на сервере, обращаться к БД или серверным API и не требуют `useState`, `useEffect` или обработчиков браузерных событий.

**Client Component** обозначается `"use client"`:

```
'use client'

export default function Counter() {
  const [count, setCount] = useState(0)

  return <button onClick={() => setCount(count + 1)}>{count}</button>
}
```

Такой компонент отправляется браузеру и может использовать состояние, эффекты, события и browser API.

### Главное отличие

| Server Component                         | Client Component                                 |
| ---------------------------------------- | ------------------------------------------------ |
| Выполняется на сервере                   | Выполняется в браузере                           |
| JS компонента не отправляется клиенту    | JS отправляется в bundle                         |
| Может работать с серверными ресурсами    | Может работать с `window`, `localStorage` и т.д. |
| Нет `useState`, `useEffect`              | Есть state и effects                             |
| Не может напрямую обрабатывать `onClick` | Может обрабатывать события                       |
| По умолчанию в Next.js App Router        | Нужен `"use client"`                             |

**На практике** я бы оставлял компоненты Server Components, если им не нужна интерактивность. А `"use client"` добавлял только на границе, где появляются state, события или browser API. Это позволяет не тащить лишний JavaScript в браузер.

