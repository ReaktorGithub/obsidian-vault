**TypeORM** - ORM для TypeScript/JavaScript, который позволяет работать с реляционной БД через классы, Entity и Repository вместо написания SQL для каждого запроса.

```
Entity
   ↓
Repository
   ↓
TypeORM
   ↓
Database
```

В NestJS TypeORM подключается через `@nestjs/typeorm`.

---

## Установка

```
npm install @nestjs/typeorm typeorm pg
```

`pg` нужен для PostgreSQL.

---

# Подключение

В `AppModule`:

```
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'postgres',
      password: 'password',
      database: 'movies',
      autoLoadEntities: true,
    }),
  ],
})
export class AppModule {}
```

`forRoot()` создает и настраивает подключение TypeORM для приложения.

---

## `forRootAsync()`

Используется, когда конфигурацию нужно получить через DI, например из `ConfigService`.

```
TypeOrmModule.forRootAsync({
  imports: [ConfigModule],
  inject: [ConfigService],

  useFactory: (config: ConfigService) => ({
    type: 'postgres',
    host: config.get('DB_HOST'),
    port: config.get<number>('DB_PORT'),
    username: config.get('DB_USER'),
    password: config.get('DB_PASSWORD'),
    database: config.get('DB_NAME'),
  }),
})
```

```
forRoot()
    ↓
конфигурация задана напрямую

forRootAsync()
    ↓
конфигурация создается через DI
```

---

# Entity

**Entity** - класс, который описывает таблицу БД.

```
@Entity('movies')
export class Movie {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column()
  year: number;
}
```

Соответствие:

```
Movie class → таблица movies
id          → колонка id
title       → колонка title
year        → колонка year
```

---

# Основные декораторы Entity

### `@Entity()`

Определяет класс как Entity.

```
@Entity('movies')
export class Movie {}
```

---

### `@PrimaryGeneratedColumn()`

Автоматически генерируемый primary key.

```
@PrimaryGeneratedColumn()
id: number;
```

---

### `@Column()`

Обычная колонка.

```
@Column()
title: string;
```

Можно указать параметры:

```
@Column({ unique: true })
email: string;
```

---

### `@CreateDateColumn()`

Дата создания записи.

```
@CreateDateColumn()
createdAt: Date;
```

TypeORM заполняет ее автоматически.

---

### `@UpdateDateColumn()`

Дата последнего изменения.

```
@UpdateDateColumn()
updatedAt: Date;
```

---

# Repository

**Repository** предоставляет методы для работы с конкретной Entity.

В NestJS:

```
@Injectable()
export class MovieService {
  constructor(
    @InjectRepository(Movie)
    private readonly movieRepository: Repository<Movie>,
  ) {}
}
```

Чтобы Repository был доступен:

```
@Module({
  imports: [
    TypeOrmModule.forFeature([Movie]),
  ],
  providers: [MovieService],
})
export class MovieModule {}
```

```
TypeOrmModule.forFeature([Movie])
              ↓
       Repository<Movie>
              ↓
         MovieService
```

---

# CRUD

## Create

```
const movie = this.movieRepository.create({
  title: 'Interstellar',
  year: 2014,
});

await this.movieRepository.save(movie);
```

`create()` создает объект Entity в памяти.

`save()` сохраняет его в БД.

---

## Read

Получить все:

```
const movies = await this.movieRepository.find();
```

Получить по ID:

```
const movie = await this.movieRepository.findOneBy({
  id: 1,
});
```

---

## Update

```
await this.movieRepository.update(
  1,
  {
    title: 'Interstellar 2',
  },
);
```

Или через Entity:

```
const movie = await this.movieRepository.findOneBy({
  id: 1,
});

movie.title = 'Interstellar 2';

await this.movieRepository.save(movie);
```

---

## Delete

```
await this.movieRepository.delete(1);
```

---

# `find` и фильтрация

```
const movies = await this.movieRepository.find({
  where: {
    year: 2024,
  },
});
```

Можно использовать операторы:

```
import { MoreThan } from 'typeorm';

const movies = await this.movieRepository.find({
  where: {
    year: MoreThan(2020),
  },
});
```

---

# Relations

TypeORM поддерживает:

```
OneToOne
OneToMany
ManyToOne
ManyToMany
```

---

## OneToMany / ManyToOne

Например:

```
Movie 1 ─────── N Review
```

### Movie

```
@Entity('movies')
export class Movie {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @OneToMany(
    () => Review,
    (review) => review.movie,
  )
  reviews: Review[];
}
```

### Review

```
@Entity('reviews')
export class Review {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  text: string;

  @ManyToOne(
    () => Movie,
    (movie) => movie.reviews,
  )
  @JoinColumn({ name: 'movie_id' })
  movie: Movie;
}
```

В БД внешний ключ будет находиться в `reviews`:

```
reviews
-------------------------
id | text | movie_id
-------------------------
1  | Good | 10
2  | Great| 10
3  | Bad  | 15
```

**`ManyToOne` - owning side связи**, потому что именно там хранится foreign key.

---

# Загрузка relations

Можно загрузить связанные данные:

```
const movie = await this.movieRepository.findOne({
  where: {
    id: 1,
  },
  relations: {
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
    },
  ],
}
```

---

# Eager

Можно указать автоматическую загрузку relation:

```
@OneToMany(
  () => Review,
  (review) => review.movie,
  {
    eager: true,
  },
)
reviews: Review[];
```

Теперь `reviews` будут автоматически загружаться вместе с `Movie`.

Использовать осторожно, так как relation будет загружаться при каждом запросе Entity.

---

# Cascade

Позволяет автоматически выполнять операции над связанными Entity.

```
@OneToMany(
  () => Review,
  (review) => review.movie,
  {
    cascade: true,
  },
)
reviews: Review[];
```

Например, сохранение `Movie` может автоматически сохранить новые `Review`.

---

# QueryBuilder

Когда обычных методов Repository недостаточно, используется `QueryBuilder`.

```
const movies = await this.movieRepository
  .createQueryBuilder('movie')
  .where('movie.year > :year', {
    year: 2020,
  })
  .orderBy('movie.year', 'DESC')
  .getMany();
```

Можно делать:

- сложные фильтры
- JOIN
- сортировку
- группировку
- агрегатные запросы

---

# Relations через QueryBuilder

```
const movies = await this.movieRepository
  .createQueryBuilder('movie')
  .leftJoinAndSelect(
    'movie.reviews',
    'review',
  )
  .getMany();
```

`leftJoinAndSelect` делает JOIN и добавляет связанные данные в результат.

---

# Pagination

Простой вариант:

```
const movies = await this.movieRepository.find({
  skip: 20,
  take: 10,
});
```

```
skip → сколько пропустить
take → сколько получить
```

Для сложной пагинации можно использовать QueryBuilder.

---

# DTO vs Entity

Это важно не путать.

### Entity

Описывает структуру данных в БД:

```
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;
}
```

### DTO

Описывает данные, которые приходят в приложение:

```
export class CreateUserDto {
  email: string;
}
```

```
HTTP request
     ↓
    DTO
     ↓
   Service
     ↓
   Entity
     ↓
  Database
```

---

# Repository vs EntityManager

### Repository

Работает с конкретной Entity:

```
movieRepository.find();
```

### EntityManager

Может работать с разными Entity:

```
manager.find(Movie);
manager.find(User);
```

Для обычного CRUD чаще используют Repository.

---

# Transactions

Транзакция позволяет выполнить несколько операций атомарно.

```
await this.dataSource.transaction(
  async (manager) => {
    await manager.save(movie);

    await manager.save(review);
  },
);
```

Если внутри транзакции возникает ошибка, изменения откатываются.

```
Operation 1 ──┐
Operation 2 ──┼── Transaction
Operation 3 ──┘
                  ↓
             success → COMMIT
             error   → ROLLBACK
```

---

# Migrations

Миграции используются для изменения структуры БД.

Например:

```
Было:

users
- id
- name

Стало:

users
- id
- name
- email
```

Создается migration, которая изменяет структуру БД.

В production обычно используют migrations вместо автоматического изменения схемы.

---

# `synchronize`

```
TypeOrmModule.forRoot({
  ...
  synchronize: true,
});
```

TypeORM автоматически пытается синхронизировать Entity со схемой БД.

Для production:

```
synchronize: false
```

Лучше использовать migrations, потому что автоматическая синхронизация может привести к нежелательным изменениям структуры БД.

---

# `forFeature()`

```
TypeOrmModule.forFeature([Movie])
```

Регистрирует Repository конкретной Entity в текущем модуле.

После этого можно:

```
@InjectRepository(Movie)
private readonly movieRepository: Repository<Movie>
```

---

# `forRoot()` vs `forFeature()`

```
forRoot()
    ↓
настройка TypeORM
подключение к БД

forFeature([Movie])
    ↓
Repository<Movie>
доступный в конкретном модуле
```

---

# `forRootAsync()`

```
TypeOrmModule.forRootAsync({
  imports: [ConfigModule],
  inject: [ConfigService],

  useFactory: (config: ConfigService) => ({
    type: 'postgres',
    host: config.get('DB_HOST'),
    port: config.get<number>('DB_PORT'),
  }),
});
```

Используется, когда конфигурация TypeORM зависит от других providers.

---

# DataSource

`DataSource` представляет подключение и конфигурацию TypeORM.

```
constructor(
  private readonly dataSource: DataSource,
) {}
```

Через него можно:

- получать EntityManager
- запускать transactions
- создавать QueryBuilder
- работать с подключением

---

# Lifecycle

Упрощенно:

```
NestJS
  ↓
TypeOrmModule.forRoot()
  ↓
DataSource
  ↓
Database connection
  ↓
Repository
  ↓
Service
  ↓
Controller
```

---

# Основные декораторы

```
@Entity()
@PrimaryGeneratedColumn()
@Column()

@OneToOne()
@OneToMany()
@ManyToOne()
@ManyToMany()

@JoinColumn()
@JoinTable()

@CreateDateColumn()
@UpdateDateColumn()
```

---

# Что спросить могут на собеседовании

### Что такое Entity?

Класс, который описывает структуру таблицы и отображается TypeORM на таблицу БД.

### Что такое Repository?

Объект для выполнения операций с конкретной Entity.

### `forRoot` vs `forFeature`?

`forRoot` настраивает TypeORM и подключение к БД. `forFeature` регистрирует конкретные Entity и их Repository в модуле.

### `OneToMany` vs `ManyToOne`?

`OneToMany` описывает связь с точки зрения главной сущности. `ManyToOne` описывает связь со стороны зависимой сущности и обычно содержит внешний ключ.

### Что такое QueryBuilder?

Инструмент TypeORM для построения сложных SQL-запросов через API TypeScript.

### Зачем нужны migrations?

Чтобы контролируемо изменять структуру БД между версиями приложения.

### Почему `synchronize: true` опасен в production?

TypeORM может автоматически изменить структуру БД на основе Entity. В production безопаснее управлять изменениями через migrations.

---

# Главная схема

```
                 NestJS
                    │
             TypeOrmModule
                    │
          ┌─────────┴─────────┐
          │                   │
       forRoot()         forFeature()
          │                   │
    DataSource           Repository
          │                   │
          └─────────┬─────────┘
                    ↓
                 Entity
                    ↓
                Database
```

**Главное:** `Entity` описывает таблицу, `Repository` работает с Entity, `DataSource` управляет подключением и транзакциями, `QueryBuilder` нужен для сложных запросов, а `forRoot` и `forFeature` подключают TypeORM на разных уровнях.
