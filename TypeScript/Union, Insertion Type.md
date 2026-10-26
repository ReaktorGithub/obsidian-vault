В TypeScript операторы `|` и `&` используются для создания составных типов.

## 1. `|` — Union Type (объединение)

Позволяет переменной иметь **одно из нескольких указанных типов**.

```
type ID = string | number;

let id: ID = 123;
id = "abc"; // OK
id = true;  // Ошибка
```

Для объектов можно указать несколько вариантов структуры:

```
type Cat = {
  meow: () => void;
};

type Dog = {
  bark: () => void;
};

type Animal = Cat | Dog;

const animal: Animal = {
  meow: () => console.log("Meow"),
};
```

`Animal` может быть `Cat` или `Dog`. При этом объект может содержать и свойства обоих типов.

## 2. `&` — Intersection Type (пересечение)

Объединяет требования нескольких типов. Результат должен удовлетворять **всем указанным типам одновременно**.

```
type User = {
  name: string;
};

type Employee = {
  salary: number;
};

type Worker = User & Employee;

const worker: Worker = {
  name: "Oleg",
  salary: 100000,
};
```

`Worker` содержит свойства и `User`, и `Employee`.

### Что будет при конфликте типов?

```
type A = {
  value: string;
};

type B = {
  value: number;
};

type C = A & B;
```

У `C` свойство `value` должно одновременно иметь тип `string` и `number`. Это невозможно, поэтому его тип становится `never`.

## 3. Главное отличие

|Оператор|Название|Смысл|
|---|---|---|
|`\|`|Union|Один из типов|
|`&`|Intersection|Все типы одновременно|

**Пример для запоминания:**

```
type A = { name: string };
type B = { age: number };

type Union = A | B;
type Intersection = A & B;

// Union: объект должен соответствовать A или B
const a: Union = { name: "Oleg" };
const b: Union = { age: 25 };

// Intersection: объект должен соответствовать обоим типам
const c: Intersection = {
  name: "Oleg",
  age: 25,
};
```

**Важно:** `|` и `&` в TypeScript работают на уровне типов. Это не то же самое, что логические операторы `||` и `&&`, которые работают со значениями во время выполнения программы.

