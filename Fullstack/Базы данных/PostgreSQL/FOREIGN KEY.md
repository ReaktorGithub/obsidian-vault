
В PostgreSQL внешний ключ (`FOREIGN KEY`) связывает таблицы и обеспечивает ссылочную целостность: значение в одной таблице должно ссылаться на существующую запись в другой.

## Добавить FOREIGN KEY при создании таблицы

Например, есть таблица пользователей `users` и таблица заказов `orders`.

```
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    amount NUMERIC(10, 2)
);
```

`user_id` — внешний ключ, который ссылается на `id` в таблице `users`.

## Добавить FOREIGN KEY в существующую таблицу

Используем `ALTER TABLE`:

```
ALTER TABLE orders
ADD CONSTRAINT fk_orders_users
FOREIGN KEY (user_id)
REFERENCES users(id);
```

- `ALTER TABLE orders` — изменяем таблицу `orders`.
- `ADD CONSTRAINT fk_orders_users` — добавляем ограничение с именем `fk_orders_users`.
- `FOREIGN KEY (user_id)` — указываем поле внешнего ключа.
- `REFERENCES users(id)` — указываем таблицу и поле, на которые ссылаемся.

**Важно:** типы данных связанных столбцов должны быть совместимы, а поле `users.id` должно быть первичным ключом (`PRIMARY KEY`) или иметь ограничение уникальности (`UNIQUE`).

## Пример работы

Добавим пользователя и заказ:

```
INSERT INTO users (name)
VALUES ('Олег');

INSERT INTO orders (user_id, amount)
VALUES (1, 1500.00);
```

Заказ будет связан с пользователем, у которого `id = 1`.

Если попытаться добавить заказ с несуществующим пользователем:

```
INSERT INTO orders (user_id, amount)
VALUES (999, 1500.00);
```

PostgreSQL выдаст ошибку, если пользователя с `id = 999` нет.

## Удаление внешнего ключа

```
ALTER TABLE orders
DROP CONSTRAINT fk_orders_users;
```
