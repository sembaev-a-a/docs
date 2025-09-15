# Менеджер источников данных 

`DataSourceManager` - это класс для управления несколькими экземплярами источников данных (`dataSource`).

## API методы

### add()
Добавляет экземпляр источника данных.

```typescript
add(dataSource: DataSource, options: any = {}): Promise<void>
```

### use()
Добавляет глобальное промежуточное программное обеспечение для экземпляра источника данных.

### middleware()
Получает промежуточное программное обеспечение текущего экземпляра DataSourceManager, которое можно использовать для обработки HTTP-запросов.

### afterAddDataSource()
Хук-функция, которая срабатывает после добавления нового источника данных.

```typescript
afterAddDataSource(hook: DataSourceHook)

type DataSourceHook = (dataSource: DataSource) => void;
```

### registerDataSourceType()
Регистрирует тип источника данных и соответствующий ему класс.

```typescript
registerDataSourceType(type: string, dataSourceClass: typeof DataSource)
```

### getDataSourceType()
Получает класс источника данных по его типу.

```typescript
getDataSourceType(type: string): typeof DataSource
```

### buildDataSourceByType()
Создает экземпляр источника данных на основе зарегистрированного типа и параметров.

```typescript
buildDataSourceByType(type: string, options: any): DataSource
```

