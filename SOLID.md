## SOLID в контексте React / JavaScript

SOLID — это 5 принципов проектирования, пришедших из ООП, но они отлично применяются и в React/JS, особенно при работе с крупными SPA, компонентной архитектурой, hooks и бизнес-логикой.

В React SOLID помогает:

- уменьшать связанность компонентов;
- упрощать поддержку;
- переиспользовать код;
- избегать «god-компонентов»;
- легче тестировать приложение.

---

# S — Single Responsibility Principle

## Принцип единственной ответственности

> Модуль должен иметь только одну причину для изменения.

В React:

- компонент отвечает только за UI;
- hook — только за бизнес-логику;
- API слой — только за запросы;
- utils — только за преобразование данных.

---

## ❌ Плохой пример

```
function UserProfile() {  const [user, setUser] = useState(null);  useEffect(() => {    fetch('/api/user')      .then(r => r.json())      .then(setUser);  }, []);  function formatDate(date) {    return new Date(date).toLocaleDateString();  }  if (!user) return 'Loading';  return (    <div>      <h1>{user.name}</h1>      <p>{formatDate(user.createdAt)}</p>    </div>  );}
```

Компонент:

- рендерит UI;
- делает запросы;
- содержит formatting;
- управляет состоянием.

Слишком много ответственности.

---

## ✅ Хороший пример

### API

```
export async function getUser() {  const response = await fetch('/api/user');  return response.json();}
```

### Hook

```
export function useUser() {  const [user, setUser] = useState(null);  useEffect(() => {    getUser().then(setUser);  }, []);  return user;}
```

### UI

```
function UserProfile() {  const user = useUser();  if (!user) return 'Loading';  return <UserCard user={user} />;}
```

---

# O — Open/Closed Principle

## Открыт для расширения, закрыт для изменения

> Код должен расширяться без изменения существующего кода.

В React это:

- composition вместо if/switch;
- children/render props;
- конфигурируемые компоненты;
- strategy pattern.

---

## ❌ Плохой пример

```
function Button({ type }) {  if (type === 'primary') {    return <button className="blue">...</button>;  }  if (type === 'danger') {    return <button className="red">...</button>;  }}
```

Каждый новый type требует изменения компонента.

---

## ✅ Хороший пример

```
function Button({ className, children }) {  return (    <button className={className}>      {children}    </button>  );}
```

Использование:

```
<Button className="blue">Save</Button><Button className="red">Delete</Button>
```

Или:

```
<Button variant="danger" />
```

где variants вынесены в конфиг.

---

# L — Liskov Substitution Principle

## Принцип подстановки Барбары Лисков

> Наследник должен корректно заменять базовый тип.

В React наследование используется редко, но принцип актуален для:

- component contracts;
- props API;
- кастомных hooks;
- polymorphic components.

---

## ❌ Плохой пример

```
function Input(props) {  return <input {...props} />;}function PhoneInput(props) {  return <input value="+7" />;}
```

`PhoneInput` ломает ожидаемое поведение:

- игнорирует value;
- не работает как обычный Input.

---

## ✅ Хороший пример

```
function PhoneInput({ value, onChange }) {  return (    <input      value={value}      onChange={onChange}    />  );}
```

Компонент сохраняет контракт обычного input.

---

# I — Interface Segregation Principle

## Принцип разделения интерфейсов

> Не заставляйте клиентов зависеть от ненужных методов.

В React:

- не передавать огромные props object;
- разделять hooks;
- избегать универсальных компонентов на 100 пропсов.

---

## ❌ Плохой пример

```
<UserCard  user={user}  theme={theme}  locale={locale}  permissions={permissions}  analytics={analytics}  featureFlags={featureFlags}/>
```

Компонент знает слишком много.

---

## ✅ Хороший пример

```
<UserCard user={user} />
```

А нужное:

- theme → через context;
- analytics → через hook;
- permissions → выше по дереву.

---

## Hooks тоже можно делить

❌

```
useDashboard()
```

который:

- грузит данные;
- управляет фильтрами;
- делает websocket;
- считает статистику.

✅

```
useUsers()useFilters()useStatistics()useRealtime()
```

---

# D — Dependency Inversion Principle

## Инверсия зависимостей

> Зависеть нужно от абстракций, а не от конкретных реализаций.

В React:

- dependency injection;
- context;
- адаптеры API;
- абстракция над storage/fetch.

---

## ❌ Плохой пример

```
async function getUsers() {  return fetch('/api/users');}
```

Компонент жёстко привязан к fetch.

---

## ✅ Хороший пример

```
export function createUserService(client) {  return {    getUsers() {      return client.get('/users');    }  };}
```

Использование:

```
const service = createUserService(axiosClient);
```

Теперь можно:

- заменить axios;
- мокать сервисы;
- подключить GraphQL;
- использовать тестовый client.