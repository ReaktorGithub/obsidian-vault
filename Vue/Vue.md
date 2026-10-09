> Vue - прогрессивный JavaScript-фреймворк для создания пользовательских интерфейсов. Основные концепции - компонентный подход, реактивность, Composition API, props и события. Для маршрутизации обычно используется Vue Router, для глобального состояния - Pinia.

А основные вещи, которые стоит выучить:

```
Vue 3
│
├── Components
├── Template
├── ref / reactive
├── computed
├── watch
├── Props
├── Emits
├── Directives
│   ├── v-if
│   ├── v-for
│   ├── v-model
│   ├── v-bind
│   └── v-on
│
├── Lifecycle
│   ├── onMounted
│   └── onUnmounted
│
├── Composition API
├── Composables
├── Pinia
└── Vue Router
```

**Главное отличие от React:** Vue имеет встроенную реактивную систему и свой template-синтаксис с директивами, тогда как React в основном строит UI через JavaScript/JSX.

### 1. Компонент Vue

Обычно компонент находится в `.vue` файле:

```
<script setup>
import { ref } from 'vue'

const count = ref(0)

const increment = () => {
  count.value++
}
</script>

<template>
  <button @click="increment">
    {{ count }}
  </button>
</template>

<style scoped>
button {
  padding: 10px;
}
</style>
```

Vue-компонент состоит из:

- `<script>` - логика
- `<template>` - разметка
- `<style>` - стили

Аналогия с React:

```
function Counter() {
  const [count, setCount] = useState(0)

  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  )
}
```

---

## 2. `ref` — аналог `useState`

```
const count = ref(0)
```

В шаблоне:

```
{{ count }}
```

В JavaScript:

```
count.value++
```

Почему `.value`?

`ref()` возвращает специальный реактивный объект:

```
{
  value: 0
}
```

Vue отслеживает изменение `value` и обновляет UI.

---

## 3. `reactive`

Для объектов часто используют:

```
const user = reactive({
  name: 'Oleg',
  age: 30
})

user.age++
```

В отличие от `ref`:

```
const user = ref({
  name: 'Oleg'
})

user.value.name
```

Упрощённо:

- `ref` - удобно для отдельных значений
- `reactive` - удобно для объектов

---

## 4. Двустороннее связывание — `v-model`

Очень важная концепция Vue.

```
<input v-model="name">

<p>{{ name }}</p>
```

Изменение input автоматически меняет `name`.

В React обычно:

```
<input
  value={name}
  onChange={e => setName(e.target.value)}
/>
```

Во Vue это:

```
<input v-model="name">
```

`v-model` особенно часто используется в формах.

---

## 5. Директивы

Vue активно использует специальные атрибуты - **директивы**.

### `v-if`

```
<div v-if="isLogged">
  Личный кабинет
</div>
```

Аналог:

```
{isLogged && <div>Личный кабинет</div>}
```

### `v-else`

```
<div v-if="isLogged">
  Кабинет
</div>

<div v-else>
  Авторизация
</div>
```

### `v-for`

```
<ul>
  <li v-for="user in users" :key="user.id">
    {{ user.name }}
  </li>
</ul>
```

Аналог React:

```
users.map(user => (
  <li key={user.id}>
    {user.name}
  </li>
))
```

### `v-bind`

```
<img :src="imageUrl">
```

Сокращённая форма:

```
:src="imageUrl"
```

### `v-on`

```
<button v-on:click="increment">
```

Сокращённо:

```
<button @click="increment">
```

---

## 6. Computed

Очень важная штука во Vue.

```
const firstName = ref('Oleg')
const lastName = ref('Verushkin')

const fullName = computed(() => {
  return `${firstName.value} ${lastName.value}`
})
```

В шаблоне:

```
{{ fullName }}
```

`computed` похож на **мемоизированное вычисляемое значение**.

В React ближайшая аналогия:

```
const fullName = useMemo(
  () => `${firstName} ${lastName}`,
  [firstName, lastName]
)
```

Но `computed` - естественная часть реактивной системы Vue.

---

## 7. `watch`

Позволяет реагировать на изменение состояния.

```
watch(count, (newValue, oldValue) => {
  console.log(newValue)
})
```

Например:

```
watch(search, async (value) => {
  await fetchUsers(value)
})
```

Аналогия с React:

```
useEffect(() => {
  fetchUsers(search)
}, [search])
```

---

## 8. Props

Родитель передаёт данные ребёнку:

```
<UserCard :user="user" />
```

В дочернем компоненте:

```
<script setup>
const props = defineProps({
  user: Object
})
</script>

<template>
  <div>{{ props.user.name }}</div>
</template>
```

В современном Vue часто используют TypeScript:

```
const props = defineProps<{
  user: User
}>()
```

---

## 9. События от ребёнка к родителю

В React:

```
<button onClick={onDelete}>
```

Во Vue ребёнок может сделать:

```
const emit = defineEmits(['delete'])

emit('delete', user.id)
```

Родитель:

```
<UserCard @delete="deleteUser" />
```

Получается схема:

```
Parent
  ↓ props
Child
  ↑ emit
Parent
```

То есть концепция очень похожа на React.

---

## 10. Lifecycle

Во Vue есть lifecycle hooks:

```
onMounted(() => {
  console.log('mounted')
})

onUpdated(() => {
  console.log('updated')
})

onUnmounted(() => {
  console.log('unmounted')
})
```

Примерно соответствуют React:

```
Vue                  React

onMounted            useEffect(..., [])
onUpdated            useEffect(...)
onUnmounted          useEffect(() => {
                       return cleanup
                     })
```

---

## 11. Composition API

В современном Vue 3 основной подход - **Composition API**.

```
<script setup>
import {
  ref,
  computed,
  watch,
  onMounted
} from 'vue'
</script>
```

Идея похожа на React Hooks:

```
Vue Composition API     React

ref                      useState
computed                 useMemo
watch                    useEffect
onMounted                useEffect
composables              custom hooks
```

Например, переиспользуемую логику можно вынести:

```
// useCounter.js

export function useCounter() {
  const count = ref(0)

  const increment = () => {
    count.value++
  }

  return {
    count,
    increment
  }
}
```

Это очень похоже на:

```
function useCounter() {
  const [count, setCount] = useState(0)

  ...
}
```

---

## 12. Pinia

Для глобального состояния в современном Vue обычно используют **Pinia**.

Например:

```
export const useUserStore = defineStore('user', () => {
  const user = ref(null)

  const login = () => {
    // ...
  }

  return {
    user,
    login
  }
})
```

В React аналогичная задача решается через Redux, Zustand, MobX и т.д.

---

## 13. Vue Router

Для роутинга используется Vue Router:

```
const routes = [
  {
    path: '/',
    component: Home
  },
  {
    path: '/users',
    component: Users
  }
]
```

В компоненте:

```
<RouterLink to="/users">
  Пользователи
</RouterLink>

<RouterView />
```

Аналог React Router.