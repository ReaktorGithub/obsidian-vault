`this` - это ссылка на объект, в контексте которого была вызвана функция. Его значение определяется способом вызова функции, а не местом её объявления.

### Правила определения this

1. Обычный вызов функции - `this` равен `undefined` в strict mode и глобальному объекту (`window` в браузере) без strict mode.
    
2. Вызов через объект - `this` ссылается на объект слева от точки.
    
3. Стрелочная функция - не имеет собственного `this`, а берёт его из внешней области видимости (лексический `this`).
    
4. call, apply, bind - позволяют явно задать значение `this`.
    

### Примеры

Вызов через объект:

```
const user = {
  name: 'Oleg',
  sayName() {
    console.log(this.name);
  }
};

user.sayName(); // 'Oleg'
```

Потеря контекста:

```
const user = {
  name: 'Oleg',
  sayName() {
    console.log(this.name);
  }
};

const fn = user.sayName;
fn(); // undefined (strict mode)
```

Стрелочная функция:

```
const user = {
  name: 'Oleg',
  sayName: () => {
    console.log(this.name);
  }
};

user.sayName(); // не 'Oleg'
```

Стрелочная функция не получает `this` от объекта `user`, потому что берёт его из внешней области видимости.

### Управление this

```
function sayName() {
  console.log(this.name);
}

const user = { name: 'Oleg' };

sayName.call(user);  // Oleg
sayName.apply(user); // Oleg

const boundFn = sayName.bind(user);
boundFn(); // Oleg
```

- `call` - вызывает функцию с указанным `this`, передавая аргументы по отдельности.
    
- `apply` - вызывает функцию с указанным `this`, передавая аргументы массивом.
    
- `bind` - возвращает новую функцию с закреплённым `this`, не вызывая её сразу.
    

Главное: `this` определяется контекстом вызова функции. Исключение - стрелочные функции, которые наследуют `this` из внешней области видимости.

