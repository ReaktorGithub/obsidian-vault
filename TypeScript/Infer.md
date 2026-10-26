`infer` в TypeScript - ключевое слово, которое позволяет **вывести тип из другого типа внутри условного типа (`extends`)**.

### Пример

```
type ReturnTypeOf<T> = T extends (...args: any[]) => infer R
  ? R
  : never;

type Result = ReturnTypeOf<() => string>; // string
type Result2 = ReturnTypeOf<() => number>; // number
```

**Как это работает:**

1. `T extends (...args: any[]) => ...` проверяет, является ли `T` типом функции.
2. `infer R` позволяет вывести тип возвращаемого значения функции.
3. Если условие выполняется, возвращается `R`, иначе `never`.

### Пример с массивом

```
type ArrayElement<T> = T extends (infer U)[] ? U : never;

type A = ArrayElement<string[]>; // string
type B = ArrayElement<number[]>; // number
type C = ArrayElement<boolean>;  // never
```

Здесь `infer U` извлекает тип элемента массива.

### Где используется?

- Получение типа возвращаемого значения функции.
- Извлечение типа элементов массива.
- Извлечение типов параметров и свойств.
- Создание сложных условных типов.

Например, TypeScript уже предоставляет встроенный `ReturnType<T>`:

```
function getUser() {
  return { id: 1, name: "Oleg" };
}

type User = ReturnType<typeof getUser>;
// { id: number; name: string }
```

### Коротко для Obsidian

- `infer` - позволяет вывести и сохранить тип в переменную типа внутри условного типа.
- Используется внутри `T extends ... ? ... : ...`.
- Позволяет извлекать типы из функций, массивов, промисов и других структур.
- Применяется при создании сложных обобщённых типов (Generics).
