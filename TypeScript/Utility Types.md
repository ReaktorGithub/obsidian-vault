Utility Types - встроенные обобщённые типы TypeScript, которые позволяют преобразовывать существующие типы без необходимости описывать их заново.

| Utility Type               | Назначение                                             |
| -------------------------- | ------------------------------------------------------ |
| `Partial<T>`               | Делает все свойства необязательными                    |
| `Required<T>`              | Делает все свойства обязательными                      |
| `Readonly<T>`              | Запрещает изменение свойств через тип                  |
| `Pick<T, K>`               | Выбирает свойства                                      |
| `Omit<T, K>`               | Исключает свойства                                     |
| `Record<K, T>`             | Создаёт объектный тип с заданными ключами и значениями |
| `Exclude<T, U>`            | Исключает варианты Union Type                          |
| `Extract<T, U>`            | Оставляет подходящие варианты Union Type               |
| `NonNullable<T>`           | Исключает `null` и `undefined`                         |
| `Awaited<T>`               | Извлекает тип результата Promise                       |
| `Parameters<T>`            | Получает типы параметров функции                       |
| `ReturnType<T>`            | Получает тип возвращаемого значения                    |
| `ConstructorParameters<T>` | Получает типы параметров конструктора                  |
| `InstanceType<T>`          | Получает тип экземпляра класса                         |
| `ThisParameterType<T>`     | Извлекает тип `this`                                   |
| `OmitThisParameter<T>`     | Удаляет параметр `this` из типа функции                |
| `ThisType<T>`              | Задаёт контекстный тип `this`                          |
| `Uppercase<S>`             | Переводит строку в верхний регистр                     |
| `Lowercase<S>`             | Переводит строку в нижний регистр                      |
| `Capitalize<S>`            | Делает первую букву заглавной                          |
| `Uncapitalize<S>`          | Делает первую букву строчной                           |
| `NoInfer<T>`               | Запрещает вывод типа из конкретного вхождения          |
## Что важно запомнить для собеседования

- `Partial`, `Required`, `Readonly`, `Pick`, `Omit`, `Record` - работа со свойствами объектов.
    
- `Exclude`, `Extract`, `NonNullable` - преобразование типов и объединений.
    
- `Parameters`, `ReturnType`, `ConstructorParameters`, `InstanceType` - извлечение типов из функций и классов.
    
- `Awaited` - извлечение результата асинхронных операций.
    
- `Uppercase`, `Lowercase`, `Capitalize`, `Uncapitalize` - преобразование строковых литеральных типов.
    
- `NoInfer` - управление выводом типов.
    

Все перечисленные типы встроены в TypeScript. Они работают преимущественно на этапе проверки типов и не создают дополнительный JavaScript-код во время выполнения.

Ниже - основные встроенные Utility Types стандартной библиотеки TypeScript, с кратким описанием и примером каждого.

Для примеров будем использовать тип:

```
type User = {
  id: number;
  name: string;
  age?: number;
};
```

## 1. Преобразование свойств объектов

### `Partial<T>`

Делает все свойства необязательными.

```
type UserUpdate = Partial<User>;

// { id?: number; name?: string; age?: number }
```

### `Required<T>`

Делает все свойства обязательными.

```
type FullUser = Required<User>;

// { id: number; name: string; age: number }
```

### `Readonly<T>`

Делает все свойства доступными только для чтения.

```
type ReadonlyUser = Readonly<User>;

const user: ReadonlyUser = { id: 1, name: "Oleg" };
// user.name = "Ivan"; // Ошибка
```

### `Pick<T, K>`

Выбирает указанные свойства из типа.

```
type UserName = Pick<User, "id" | "name">;

// { id: number; name: string }
```

### `Omit<T, K>`

Исключает указанные свойства из типа.

```
type UserWithoutId = Omit<User, "id">;

// { name: string; age?: number }
```

### `Record<K, T>`

Создаёт объектный тип с ключами `K` и значениями типа `T`.

```
type Roles = Record<"admin" | "user", boolean>;

// { admin: boolean; user: boolean }
```

Можно использовать и динамические ключи:

```
type UserMap = Record<string, User>;
```

## 2. Преобразование Union Types

### `Exclude<T, U>`

Исключает из типа `T` варианты, которые входят в `U`.

```
type Status = "success" | "error" | "loading";

type FinalStatus = Exclude<Status, "loading">;
// "success" | "error"
```

### `Extract<T, U>`

Оставляет из `T` только варианты, совместимые с `U`.

```
type Status = "success" | "error" | "loading";

type ErrorStatus = Extract<Status, "error" | "loading">;
// "error" | "loading"
```

### `NonNullable<T>`

Исключает `null` и `undefined`.

```
type Value = string | number | null | undefined;

type SafeValue = NonNullable<Value>;
// string | number
```

### `Awaited<T>`

Рекурсивно извлекает тип значения, которое возвращает `Promise` после разрешения.

```
type Result = Awaited<Promise<string>>;
// string

type Nested = Awaited<Promise<Promise<number>>>;
// number
```

Полезен при типизации асинхронных функций.

## 3. Получение типов из функций

### `Parameters<T>`

Получает кортеж типов параметров функции.

```
function sum(a: number, b: string) {}

type Args = Parameters<typeof sum>;
// [a: number, b: string]
```

### `ReturnType<T>`

Получает тип возвращаемого значения функции.

```
function getUser() {
  return { id: 1, name: "Oleg" };
}

type Result = ReturnType<typeof getUser>;
// { id: number; name: string }
```

### `ConstructorParameters<T>`

Получает кортеж типов параметров конструктора класса.

```
class User {
  constructor(public name: string, public age: number) {}
}

type Args = ConstructorParameters<typeof User>;
// [name: string, age: number]
```

### `InstanceType<T>`

Получает тип экземпляра класса по типу его конструктора.

```
class User {
  name = "Oleg";
}

type UserInstance = InstanceType<typeof User>;
// User
```

### `ThisParameterType<T>`

Извлекает тип параметра `this` из функции. Если явный тип `this` не объявлен, результатом будет `unknown`.

```
function greet(this: { name: string }) {
  return this.name;
}

type Context = ThisParameterType<typeof greet>;
// { name: string }
```

### `OmitThisParameter<T>`

Удаляет явный параметр `this` из типа функции.

```
function greet(this: { name: string }, message: string) {
  return `${message}, ${this.name}`;
}

type Greet = OmitThisParameter<typeof greet>;
// (message: string) => string
```

### `ThisType<T>`

Задаёт тип контекста `this` внутри объектных литералов. Сам по себе не преобразует тип и работает как маркерный тип в соответствующем контексте типизации.

```
type Descriptor = {
  name: string;
  methods: {
    greet(): string;
  } & ThisType<{ name: string }>;
};

const descriptor: Descriptor = {
  name: "Oleg",
  methods: {
    greet() {
      return this.name; // string
    },
  },
};
```

Важно: `ThisType<T>` не извлекает тип `this`, а помогает его задать.

## 4. Преобразование строковых типов

Эти Utility Types работают со строковыми литеральными типами на этапе компиляции.

### `Uppercase<S>`

Преобразует строку в верхний регистр.

```
type Result = Uppercase<"hello">;
// "HELLO"
```

### `Lowercase<S>`

Преобразует строку в нижний регистр.

```
type Result = Lowercase<"HELLO">;
// "hello"
```

### `Capitalize<S>`

Делает первую букву строки заглавной.

```
type Result = Capitalize<"hello">;
// "Hello"
```

### `Uncapitalize<S>`

Делает первую букву строки строчной.

```
type Result = Uncapitalize<"Hello">;
// "hello"
```

## 5. Управление выводом типов

### `NoInfer<T>`

Запрещает TypeScript использовать конкретное вхождение типа для вывода типа `T`. Полезен, когда тип должен определяться одним аргументом, а остальные аргументы должны ему соответствовать.

```
function choose<C extends string>(
  choices: C[],
  defaultChoice: NoInfer<C>
) {
  return defaultChoice;
}

choose(["red", "green"], "red");  // OK
choose(["red", "green"], "blue"); // Ошибка
```

Без `NoInfer` TypeScript может учитывать второй аргумент при выводе `C` и расширить его до `"red" | "green" | "blue"`. С `NoInfer` тип определяется по `choices`.

