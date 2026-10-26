`CONSTRAINT` — ограничение, которое обеспечивает корректность данных в таблице. Ограничения можно задавать при создании таблицы через `CREATE TABLE` или добавлять позже через `ALTER TABLE`.

## Основные виды ограничений

|Ограничение|Назначение|
|---|---|
|`PRIMARY KEY`|Уникально идентифицирует каждую строку|
|`FOREIGN KEY`|Обеспечивает связь с другой таблицей|
|`UNIQUE`|Запрещает повторяющиеся значения|
|`NOT NULL`|Запрещает `NULL`|
|`CHECK`|Проверяет выполнение условия|
|`DEFAULT`|Задаёт значение по умолчанию|

## PRIMARY KEY

`PRIMARY KEY` — первичный ключ, который однозначно идентифицирует каждую строку таблицы.

- Значения должны быть уникальными.
    
- `NULL` запрещён.
    
- В таблице может быть только один `PRIMARY KEY`, но он может состоять из нескольких столбцов.
    

### При создании таблицы

```
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL
);
```

Автоинкрементируемый идентификатор:

```
CREATE TABLE users (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name TEXT NOT NULL
);
```

### Составной PRIMARY KEY

Первичный ключ может состоять из нескольких столбцов.

```
CREATE TABLE student_courses (
    student_id INTEGER,
    course_id INTEGER,
    PRIMARY KEY (student_id, course_id)
);
```

Комбинация `student_id` и `course_id` должна быть уникальной. При этом каждое из значений по отдельности может повторяться.

### Добавление PRIMARY KEY в существующую таблицу

```
ALTER TABLE users
ADD CONSTRAINT users_pkey PRIMARY KEY (id);
```

Столбец `id` должен содержать уникальные значения и не иметь `NULL`.

### Удаление PRIMARY KEY

```
ALTER TABLE users
DROP CONSTRAINT users_pkey;
```

## Остальные ограничения

### UNIQUE

Запрещает дубликаты:

```
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    email TEXT UNIQUE
);
```

В PostgreSQL обычное ограничение `UNIQUE` по умолчанию допускает несколько значений `NULL`.

### FOREIGN KEY

Обеспечивает ссылочную целостность:

```
CREATE TABLE orders (
    id INTEGER PRIMARY KEY,
    user_id INTEGER REFERENCES users(id)
);
```

Значение `user_id` должно соответствовать существующему `users.id`, если оно не `NULL`.

### CHECK

Проверяет условие:

```
CREATE TABLE products (
    id INTEGER PRIMARY KEY,
    price NUMERIC CHECK (price >= 0)
);
```

Цена не может быть отрицательной. Само по себе `CHECK` не запрещает `NULL`.

### NOT NULL

Запрещает отсутствие значения:

```
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL
);
```

### DEFAULT

Устанавливает значение, если при вставке оно не указано:

```
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    is_active BOOLEAN DEFAULT TRUE
);
```

`DEFAULT` отличается от остальных ограничений тем, что подставляет значение, а не проверяет корректность введённого.

## Добавление и удаление ограничений

```
-- Добавить UNIQUE
ALTER TABLE users
ADD CONSTRAINT users_email_unique UNIQUE (email);

-- Добавить CHECK
ALTER TABLE products
ADD CONSTRAINT products_price_check CHECK (price >= 0);

-- Удалить ограничение
ALTER TABLE users
DROP CONSTRAINT users_email_unique;
```

`NOT NULL` изменяется отдельно через `ALTER COLUMN ... SET NOT NULL` или `DROP NOT NULL`.

## PRIMARY KEY vs UNIQUE

|Свойство|PRIMARY KEY|UNIQUE|
|---|---|---|
|Уникальность|Да|Да|
|Допускает `NULL`|Нет|Да, по умолчанию|
|Количество на таблицу|Один|Несколько|
|Составной вариант|Да|Да|
|Основное назначение|Идентификация строки|Запрет дубликатов|

Для собеседования: `PRIMARY KEY` — уникальный и не допускающий `NULL` идентификатор строки. `FOREIGN KEY` связывает таблицы, `UNIQUE` запрещает дубликаты, `CHECK` проверяет условие, `NOT NULL` запрещает отсутствие значения, а `DEFAULT` задаёт значение по умолчанию.

