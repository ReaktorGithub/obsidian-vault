Получить данные:

```
SELECT * FROM users;
```

Конкретные поля:

```
SELECT id, name
FROM users;
```

С условием:

```
SELECT *
FROM users
WHERE age > 18;
```

Несколько условий:

```
SELECT *
FROM users
WHERE age > 18
  AND city = 'Moscow';
```
