В PostgreSQL есть несколько конструкций для проверки условий, выбора значений в зависимости от условий и обработки `NULL`.

## CASE WHEN THEN ELSE

**`CASE`** - условное выражение, которое возвращает результат в зависимости от выполнения условий. Аналог `if / else` в языках программирования.

### Синтаксис

```
CASE
    WHEN условие1 THEN результат1
    WHEN условие2 THEN результат2
    ELSE результат_по_умолчанию
END
```

Пример:

```
SELECT
    name,
    salary,
    CASE
        WHEN salary >= 150000 THEN 'Высокая'
        WHEN salary >= 100000 THEN 'Средняя'
        ELSE 'Низкая'
    END AS salary_level
FROM employees;
```

|name|salary|salary_level|
|---|---|---|
|Иван|180000|Высокая|
|Анна|120000|Средняя|
|Пётр|80000|Низкая|

**Как работает:**

- Условия проверяются по порядку.
- При первом истинном условии возвращается соответствующий результат.
- Если ни одно условие не истинно, возвращается `ELSE`.
- Если `ELSE` отсутствует и ни одно условие не истинно, возвращается `NULL`.

### Две формы CASE

**1. Поиск по условиям:**

```
SELECT
    CASE
        WHEN salary > 100000 THEN 'Высокая'
        WHEN salary <= 100000 THEN 'Низкая'
        ELSE 'Неизвестна'
    END
FROM employees;
```

**2. Сравнение с конкретными значениями:**

```
SELECT
    CASE department
        WHEN 'IT' THEN 'Разработка'
        WHEN 'HR' THEN 'Кадры'
        ELSE 'Другой отдел'
    END
FROM employees;
```

Вторая форма удобна, когда нужно сравнить одно выражение с несколькими значениями.

---

## COALESCE

**`COALESCE`** возвращает первое значение, которое не равно `NULL`.

### Синтаксис

```
COALESCE(value1, value2, value3, ...)
```

Пример:

```
SELECT
    name,
    COALESCE(phone, 'Не указан') AS phone
FROM employees;
```

Если `phone` равен `NULL`, вернётся `'Не указан'`.

Можно передавать несколько значений:

```
SELECT COALESCE(NULL, NULL, 'Привет', 'Мир');
```

Результат:

```
Привет
```

**Применение:**

- Подстановка значений по умолчанию при выборке.
- Обработка отсутствующих данных.
- Вычисления с возможными `NULL`.

Например:

```
SELECT COALESCE(bonus, 0) + salary AS total_income
FROM employees;
```

Если премия отсутствует, она считается равной нулю.

Важно: `COALESCE` возвращает значение с общим типом, к которому PostgreSQL может привести все аргументы.

---

## NULLIF

**`NULLIF(a, b)`** возвращает `NULL`, если `a = b`. В противном случае возвращает `a`.

### Синтаксис

```
NULLIF(value1, value2)
```

Примеры:

```
SELECT NULLIF(10, 10);
```

Результат: `NULL`.

```
SELECT NULLIF(10, 20);
```

Результат: `10`.

### Защита от деления на ноль

Допустим, нужно рассчитать среднюю стоимость товара:

```
SELECT total_amount / NULLIF(quantity, 0) AS avg_price
FROM orders;
```

Если `quantity = 0`, выражение `NULLIF(quantity, 0)` вернёт `NULL`. В результате деление на ноль не произойдёт, а результат вычисления будет `NULL`.

**Разница между COALESCE и NULLIF:**

- `COALESCE` заменяет `NULL` первым доступным значением.
- `NULLIF` превращает определённое значение в `NULL`, если оно совпадает с другим.

---

## AND, OR, NOT

Логические операторы используются для объединения и инвертирования условий.

### AND

Возвращает `TRUE`, если оба условия истинны.

```
SELECT *
FROM employees
WHERE salary > 100000
  AND department = 'IT';
```

Выберет сотрудников IT-отдела с зарплатой больше 100000.

### OR

Возвращает `TRUE`, если хотя бы одно условие истинно.

```
SELECT *
FROM employees
WHERE department = 'IT'
   OR department = 'HR';
```

Выберет сотрудников IT- или HR-отдела.

### NOT

Инвертирует логическое значение.

```
SELECT *
FROM employees
WHERE NOT department = 'IT';
```

Выберет сотрудников, у которых отдел не равен IT, за исключением строк, где `department` равен `NULL`: такие строки тоже не попадут в результат.

Можно записать короче:

```
WHERE department <> 'IT'
```

### Таблица истинности

|A|B|A AND B|A OR B|
|---|---|---|---|
|TRUE|TRUE|TRUE|TRUE|
|TRUE|FALSE|FALSE|TRUE|
|FALSE|TRUE|FALSE|TRUE|
|FALSE|FALSE|FALSE|FALSE|

`NOT TRUE` возвращает `FALSE`, а `NOT FALSE` возвращает `TRUE`.

**Важно:** в SQL есть три логических состояния: `TRUE`, `FALSE` и `NULL` (неизвестно). Поэтому обычная булева логика здесь имеет особенности.

Например:

```
SELECT NULL = NULL;
```

Результат: `NULL`, а не `TRUE`.

Для проверки на отсутствие значения используется `IS NULL`.

---

## IS NULL и IS NOT NULL

Используются для проверки наличия или отсутствия значения `NULL`.

```
SELECT *
FROM employees
WHERE phone IS NULL;
```

Выберет сотрудников без указанного телефона.

```
SELECT *
FROM employees
WHERE phone IS NOT NULL;
```

Выберет сотрудников, у которых телефон указан.

**Неправильно:**

```
WHERE phone = NULL
```

**Правильно:**

```
WHERE phone IS NULL
```

Операторы `=` и `<>` не предназначены для проверки `NULL`.

---

## IS DISTINCT FROM и IS NOT DISTINCT FROM

Позволяют сравнивать значения с учётом `NULL`, трактуя два `NULL` как равные.

### IS DISTINCT FROM

Возвращает `TRUE`, если значения различаются, включая случаи с `NULL`.

```
SELECT NULL IS DISTINCT FROM NULL;
-- FALSE

SELECT NULL IS DISTINCT FROM 10;
-- TRUE

SELECT 10 IS DISTINCT FROM 10;
-- FALSE
```

### IS NOT DISTINCT FROM

Возвращает `TRUE`, если значения равны, в том числе если оба равны `NULL`.

```
SELECT NULL IS NOT DISTINCT FROM NULL;
-- TRUE

SELECT NULL IS NOT DISTINCT FROM 10;
-- FALSE

SELECT 10 IS NOT DISTINCT FROM 10;
-- TRUE
```

**Применение:** сравнение полей, которые могут содержать `NULL`, без получения неопределённого результата.

---

## GREATEST и LEAST

Возвращают наибольшее и наименьшее значение соответственно.

### GREATEST

```
SELECT GREATEST(10, 25, 7, 40);
```

Результат: `40`.

### LEAST

```
SELECT LEAST(10, 25, 7, 40);
```

Результат: `7`.

Пример:

```
SELECT
    name,
    GREATEST(salary, 100000) AS adjusted_salary
FROM employees;
```

Здесь зарплата не будет меньше 100000.

**Особенность PostgreSQL:** `GREATEST` и `LEAST` игнорируют аргументы `NULL`, если есть хотя бы один аргумент, отличный от `NULL`. Если все аргументы равны `NULL`, результатом будет `NULL`.

---

## BETWEEN и NOT BETWEEN

Проверяют, находится ли значение в заданном диапазоне.

```
SELECT *
FROM employees
WHERE salary BETWEEN 50000 AND 150000;
```

Эквивалентно:

```
WHERE salary >= 50000
  AND salary <= 150000
```

**Границы включаются.**

`NOT BETWEEN` проверяет, что значение находится за пределами диапазона:

```
SELECT *
FROM employees
WHERE salary NOT BETWEEN 50000 AND 150000;
```

Если `salary` равен `NULL`, строка не попадёт ни в один из этих результатов, поскольку условие будет иметь значение `NULL`.

---

## IN и NOT IN

Проверяют, входит ли значение в заданный набор.

```
SELECT *
FROM employees
WHERE department IN ('IT', 'HR', 'Sales');
```

Эквивалентно:

```
WHERE department = 'IT'
   OR department = 'HR'
   OR department = 'Sales'
```

`NOT IN` проверяет, что значение не входит в набор:

```
SELECT *
FROM employees
WHERE department NOT IN ('IT', 'HR');
```

**Важная особенность:** если набор для `NOT IN` содержит `NULL`, результат может оказаться `NULL`, а не `TRUE`, для значений, которые не совпали ни с одним элементом.

Например:

```
SELECT 10 NOT IN (1, 2, NULL);
```

Результат: `NULL`.

Поэтому при работе с подзапросами часто безопаснее использовать `NOT EXISTS`.

---

## EXISTS и NOT EXISTS

Проверяют, возвращает ли подзапрос хотя бы одну строку.

### EXISTS

```
SELECT *
FROM departments d
WHERE EXISTS (
    SELECT 1
    FROM employees e
    WHERE e.department_id = d.id
);
```

Выберет отделы, в которых есть хотя бы один сотрудник.

### NOT EXISTS

```
SELECT *
FROM departments d
WHERE NOT EXISTS (
    SELECT 1
    FROM employees e
    WHERE e.department_id = d.id
);
```

Выберет отделы, в которых нет сотрудников.

`EXISTS` возвращает логическое значение. Содержимое `SELECT` внутри него обычно неважно: важно только наличие строк.

---

## ANY, SOME и ALL

Используются для сравнения значения с результатами подзапроса или набором значений.

### ANY и SOME

`ANY` возвращает `TRUE`, если условие выполняется хотя бы для одного значения. `SOME` - синоним `ANY`.

```
SELECT *
FROM employees
WHERE salary > ANY (
    SELECT salary
    FROM employees
    WHERE department = 'HR'
);
```

Выберет сотрудников, чья зарплата выше зарплаты хотя бы одного сотрудника HR.

### ALL

`ALL` возвращает `TRUE`, если условие выполняется для всех значений.

```
SELECT *
FROM employees
WHERE salary > ALL (
    SELECT salary
    FROM employees
    WHERE department = 'HR'
);
```

Выберет сотрудников, чья зарплата выше зарплаты каждого сотрудника HR.

**Важно:** если подзапрос не вернул ни одной строки, сравнение с `ANY` возвращает `FALSE`, а с `ALL` - `TRUE`. Если результаты содержат `NULL`, возможен результат `NULL`, когда истинность условия нельзя определить.

---

## NULL-обработка в агрегатах

Агрегатные функции тоже связаны с обработкой `NULL`.

Например:

```
SELECT
    COUNT(*) AS total_rows,
    COUNT(phone) AS employees_with_phone,
    AVG(salary) AS average_salary
FROM employees;
```

- `COUNT(*)` считает все строки.
- `COUNT(phone)` считает только строки, в которых `phone` не равен `NULL`.
- `AVG(salary)` рассчитывает среднее по ненулевым значениям `salary`.

Большинство агрегатных функций игнорируют `NULL`. Исключение - например, `array_agg`, которая может включать `NULL` в результат.

Если нужно заменить отсутствующее значение при агрегации:

```
SELECT AVG(COALESCE(bonus, 0))
FROM employees;
```

В этом случае отсутствующие премии учитываются как нулевые, что может изменить среднее значение.

---

## FILTER

`FILTER` позволяет применять условие к строкам, участвующим в конкретной агрегатной функции.

```
SELECT
    COUNT(*) AS total_employees,
    COUNT(*) FILTER (WHERE department = 'IT') AS it_employees,
    COUNT(*) FILTER (WHERE department = 'HR') AS hr_employees
FROM employees;
```

В отличие от `WHERE`, фильтрующего строки для всего запроса, `FILTER` влияет только на конкретный агрегат.

Это удобно, когда нужно посчитать несколько разных показателей за один запрос.

---

## NULLIF, COALESCE и CASE: сравнение

|Конструкция|Назначение|Пример|
|---|---|---|
|`CASE`|Выбор результата по условиям|`CASE WHEN salary > 100000 THEN 'High' ELSE 'Low' END`|
|`COALESCE`|Первое значение, не равное `NULL`|`COALESCE(phone, 'Unknown')`|
|`NULLIF`|Возвращает `NULL`, если значения равны|`NULLIF(quantity, 0)`|
|`GREATEST`|Наибольшее значение|`GREATEST(10, 20, 30)`|
|`LEAST`|Наименьшее значение|`LEAST(10, 20, 30)`|
|`IS NULL`|Проверка отсутствия значения|`phone IS NULL`|
|`IS DISTINCT FROM`|Сравнение с учётом `NULL`|`a IS DISTINCT FROM b`|

---

## Кратко для собеседования

- **`CASE`** - условное выражение, аналог `if / else`.
- **`COALESCE`** - возвращает первое значение, не равное `NULL`.
- **`NULLIF`** - возвращает `NULL`, если два значения равны.
- **`AND`, `OR`, `NOT`** - логические операторы.
- **`IS NULL`, `IS NOT NULL`** - проверка на `NULL`.
- **`IS DISTINCT FROM`** - сравнение значений с учётом `NULL`.
- **`GREATEST`, `LEAST`** - поиск максимального и минимального значения.
- **`BETWEEN`** - проверка диапазона, включая границы.
- **`IN`** - проверка вхождения в набор значений.
- **`EXISTS`** - проверка наличия строк в подзапросе.
- **`ANY`, `ALL`** - сравнение с одним или всеми результатами подзапроса.
- **`FILTER`** - фильтрация строк для конкретной агрегатной функции.

**Самое важное:** в PostgreSQL `NULL` означает неизвестное или отсутствующее значение. Сравнения с `NULL` обычно возвращают `NULL`, а не `TRUE` или `FALSE`. Поэтому нужно различать `= NULL`, `IS NULL` и `IS DISTINCT FROM`.

