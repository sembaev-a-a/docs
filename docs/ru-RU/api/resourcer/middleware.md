# Промежуточное програмное обеспечение

`Middleware` в NocoBase похожи на Koa, но обладают расширенными возможностями для упрощения расширения функциональности.  
Определённые ПО можно вставлять и использовать в различных местах, например, в роутере ресурсов (resourcer), а момент их вызова определяется разработчиком.

## Конструктор

### Сигнатура

```js
constructor(options: Function)
constructor(options: MiddlewareOptions)
```

### Параметры

| Имя       | Тип                  | По умолчанию | Описание                                 |
|-----------|----------------------|--------------|------------------------------------------|
| `options` | `Function`           | —            | Функция-обработчик `middleware`               |
| `options` | `MiddlewareOptions`  | —            | Объект настроек `middleware`                  |
| `options.only`    | `string[]`     | —            | Разрешены только указанные действия       |
| `options.except`  | `string[]`     | —            | Исключённые действия (не будут обработаны)|
| `options.handler` | `Function`     | —            | Функция-обработчик                        |

### Примеры

**Простое определение:**

```js
const middleware = new Middleware((ctx, next) => {
  await next();
});
```

**Определение с параметрами:**

```js
const middleware = new Middleware({
  only: ['create', 'update'],
  async handler(ctx, next) {
    await next();
  },
});
```

## Методы экземпляра

### `getHandler()`

Возвращает скомпонованную цепочку обработчиков.

#### Пример

Следующий `middleware` при запросе выведет сначала `1`, затем `2`:

```js
const middleware = new Middleware((ctx, next) => {
  console.log(1);
  await next();
});

middleware.use(async (ctx, next) => {
  console.log(2);
  await next();
});

app.resourcer.use(middleware.getHandler());
```

---

### `use()`

Добавляет функцию-`middleware` в текущую цепочку. Используется для расширения функциональности мидлвара.  
См. пример в `getHandler()`.

#### Сигнатура

```js
use(middleware: Function)
```

#### Параметры

| Имя        | Тип        | По умолчанию | Описание                     |
|------------|------------|--------------|------------------------------|
| `middleware` | `Function` | —            | Функция-обработчик мидлвара   |

---

### `disuse()`

Удаляет ранее добавленную функцию-`middleware` из текущей цепочки.

#### Сигнатура

```js
disuse(middleware: Function)
```

#### Параметры

| Имя        | Тип        | По умолчанию | Описание                     |
|------------|------------|--------------|------------------------------|
| `middleware` | `Function` | —            | Функция-обработчик мидлвара   |

#### Пример

В этом примере при запросе будет выведено только `1`. Вывод `2` из `fn1` выполняться не будет:

```js
const middleware = new Middleware((ctx, next) => {
  console.log(1);
  await next();
});

async function fn1(ctx, next) {
  console.log(2);
  await next();
}

middleware.use(fn1);

app.resourcer.use(middleware.getHandler());

middleware.disuse(fn1);
```

---

### `canAccess()`

Проверяет, должен ли текущий `middleware` быть вызван для указанного действия. Обычно используется внутри `resourcer`.

#### Сигнатура

```js
canAccess(name: string): boolean
```

#### Параметры

| Имя     | Тип      | По умолчанию | Описание               |
|---------|----------|--------------|------------------------|
| `name`  | `string` | —            | Название действия      |

---

## Другие экспорты

### `branch()`

Создаёт "ветвящийся" `middleware`, позволяющий выбирать разные обработчики в зависимости от контекста.

#### Сигнатура

```js
branch(map: { [key: string]: Function }, reducer: Function, options): Function
```

#### Параметры

| Имя                 | Тип                         | По умолчанию       | Описание                                                                 |
|---------------------|-----------------------------|--------------------|--------------------------------------------------------------------------|
| `map`               | `{ [key: string]: Function }` | —                  | Таблица соответствия обработчиков; ключи определяются функцией `reducer`  |
| `reducer`           | `(ctx) => string`             | —                  | Функция, вычисляющая ключ ветви на основе контекста                       |
| `options?`          | `Object`                      | —                  | Дополнительные настройки ветвления                                        |
| `options.keyNotFound?`   | `Function`               | `ctx.throw(404)`   | Обработчик, вызываемый, если ключ не найден                              |
| `options.handlerNotSet?` | `Function`               | `ctx.throw(404)`   | Обработчик, вызываемый, если для ключа не задан обработчик                |

#### Пример

Определяет, как действовать дальше при аутентификации, в зависимости от параметра `authenticator` в URL:

```js
app.resourcer.use(
  branch(
    {
      password: async (ctx, next) => {
        // Обработка аутентификации по паролю
      },
      sms: async (ctx, next) => {
        // Обработка аутентификации по SMS
      },
    },
    (ctx) => {
      return ctx.action.params.authenticator ?? 'password';
    }
  )
);
```
