# Интерфейс поля (IField)

Интерфейс `IField` определяет структуру, которую должны реализовывать поля в системе.

```typescript
export type FieldOptions = {
  name: string;                // Название поля
  field: string;               // Идентификатор поля
  rawType: string;             // Исходный тип данных
  type: string;                // Тип поля
  description?: string;        // Описание поля (опционально)
  interface?: string;          // Интерфейс поля
  uiSchema?: any;              // Схема пользовательского интерфейса
  possibleTypes?: string[];    // Допустимые типы данных
  defaultValue?: any;          // Значение по умолчанию
  primaryKey: boolean;         // Является ли первичным ключом
  unique: boolean;             // Уникальное ли поле
  allowNull?: boolean;         // Допускает ли null-значения
  autoIncrement?: boolean;     // Автоинкрементное ли поле
  [key: string]: any;          // Дополнительные свойства
};

export interface IField {
  options: FieldOptions;  // Конфигурационные параметры поля
}
```

## Свойства

### options

- **Тип**: `FieldOptions`



