**Prisma** - ORM для Node.js и TypeScript, который предоставляет типобезопасный клиент для работы с БД.

Основные части:

```
schema.prisma
     ↓
Prisma Client
     ↓
Service
     ↓
Database
```

---

## Установка

```
npm install prisma @prisma/client
```

Инициализация:

```
npx prisma init
```

Создаются:

```
prisma/
  schema.prisma

.env
```

---

## `schema.prisma`

Основной файл, где описываются модели и их связи.

```
model User {
  id    Int    @id @default(autoincrement())
  name  String
  email String @unique
}
```

Модель `User` соответствует таблице `User` в БД.

---

## Поля

```
model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  age       Int?
  createdAt DateTime @default(now())
}
```

Основные типы:

```
String
Int
Float
Boolean
DateTime
```

`?` означает необязательное поле:

```
age Int?
```

`@id` - первичный ключ.

`@default()` - значение по умолчанию.

`@unique` - уникальное значение.

---

## Prisma Client

После изменения schema нужно сгенерировать клиент:

```
npx prisma generate
```

В коде:

```
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();
```

Теперь можно выполнять запросы:

```
const users = await prisma.user.findMany();
```

---

# CRUD

## Create

```
const user = await prisma.user.create({
  data: {
    name: 'Alex',
    email: 'alex@test.com',
  },
});
```

## Read

```
const users = await prisma.user.findMany();
```

Один пользователь:

```
const user = await prisma.user.findUnique({
  where: {
    id: 1,
  },
});
```

## Update

```
await prisma.user.update({
  where: {
    id: 1,
  },
  data: {
    name: 'Bob',
  },
});
```

## Delete

```
await prisma.user.delete({
  where: {
    id: 1,
  },
});
```

---

# `findUnique` vs `findFirst`

`findUnique` используется для поиска по уникальному полю:

```
prisma.user.findUnique({
  where: {
    id: 1,
  },
});
```

`findFirst` ищет первую подходящую запись:

```
prisma.user.findFirst({
  where: {
    name: 'Alex',
  },
});
```

---

# Фильтрация

```
const users = await prisma.user.findMany({
  where: {
    age: {
      gt: 18,
    },
  },
});
```

Основные операторы:

```
equals
not
in
notIn
lt
lte
gt
gte
contains
startsWith
endsWith
```

---

# Сортировка

```
const users = await prisma.user.findMany({
  orderBy: {
    name: 'asc',
  },
});
```

---

# Pagination

```
const users = await prisma.user.findMany({
  skip: 20,
  take: 10,
});
```

```
skip - сколько пропустить
take - сколько получить
```

---

# Relations

Например:

```
Movie 1 ─────── N Review
```

В Prisma:

```
model Movie {
  id      Int      @id @default(autoincrement())
  title   String
  reviews Review[]
}

model Review {
  id      Int    @id @default(autoincrement())
  text    String
  movieId Int

  movie Movie @relation(
    fields: [movieId],
    references: [id]
  )
}
```

`Movie` содержит массив `Review`.

`Review` содержит `movieId` и связь с `Movie`.

---

# Получение связанных данных

```
const movie = await prisma.movie.findUnique({
  where: {
    id: 1,
  },
  include: {
    reviews: true,
  },
});
```

Результат:

```
{
  id: 1,
  title: 'Interstellar',
  reviews: [
    {
      id: 1,
      text: 'Great movie',
      movieId: 1
    }
  ]
}
```

---

# `include` vs `select`

`include` добавляет связанные данные:

```
include: {
  reviews: true,
}
```

`select` позволяет выбрать конкретные поля:

```
select: {
  id: true,
  title: true,
}
```

Например:

```
const movie = await prisma.movie.findUnique({
  where: { id: 1 },
  select: {
    id: true,
    title: true,
    reviews: {
      select: {
        text: true,
        rating: true,
      },
    },
  },
});
```

---

# Prisma в NestJS

Обычно создают отдельный `PrismaService`:

```
@Injectable()
export class PrismaService extends PrismaClient {}
```

И модуль:

```
@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

В сервисе:

```
@Injectable()
export class UserService {
  constructor(
    private readonly prisma: PrismaService,
  ) {}

  findAll() {
    return this.prisma.user.findMany();
  }
}
```

Получается:

```
Controller
    ↓
UserService
    ↓
PrismaService
    ↓
Prisma Client
    ↓
Database
```

---

# Migrations

После изменения `schema.prisma` создают миграцию:

```
npx prisma migrate dev --name add-user
```

Миграция:

```
schema.prisma
      ↓
migration
      ↓
Database
```

Для production:

```
npx prisma migrate deploy
```

---

# Prisma Studio

Графический интерфейс для просмотра и изменения данных:

```
npx prisma studio
```

По смыслу похож на Beekeeper Studio, но Prisma Studio ориентирован именно на работу с Prisma-моделями.

---

# Transactions

Позволяют выполнить несколько операций как одну транзакцию:

```
await prisma.$transaction([
  prisma.user.create({
    data: {
      name: 'Alex',
      email: 'alex@test.com',
    },
  }),

  prisma.user.create({
    data: {
      name: 'Bob',
      email: 'bob@test.com',
    },
  }),
]);
```

Если транзакция не может быть выполнена целиком, изменения откатываются.

---

# Что важно знать на собеседовании

- `schema.prisma` - описание моделей и связей.
- `Prisma Client` - автоматически генерируемый типобезопасный клиент.
- `PrismaService` - обычно NestJS-обертка над `PrismaClient`.
- `findMany()` - получить несколько записей.
- `findUnique()` - получить уникальную запись.
- `create()` / `update()` / `delete()` - CRUD.
- `where` - фильтрация.
- `select` - выбрать конкретные поля.
- `include` - получить связанные данные.
- `skip` / `take` - pagination.
- `orderBy` - сортировка.
- `$transaction()` - транзакции.
- `prisma generate` - генерация Prisma Client.
- `prisma migrate` - миграции БД.
- `prisma studio` - GUI для просмотра данных.

### Главное отличие от TypeORM

```
TypeORM:

Entity → Repository → Database

Prisma:

schema.prisma → Prisma Client → Database
```

В TypeORM схема обычно описывается через Entity и декораторы TypeScript, а в Prisma - декларативно в `schema.prisma`.
