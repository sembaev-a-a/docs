# Базовый интерфейс (BaseInterface)

## Обзор

`BaseInterface` является базовым классом для всех типов интерфейсов. Пользователи могут расширять этот класс для реализации собственной логики интерфейсов.

```typescript
class CustomInterface extends BaseInterface {
  async toValue(value: string, ctx?: any): Promise<any> {
    // Пользовательская логика преобразования в значение
  }

  toString(value: any, ctx?: any) {
    // Пользовательская логика преобразования в строку
  }
}
// Регистрация интерфейса
db.interfaceManager.registerInterfaceType('customInterface', CustomInterface)
```

## Интерфейсы

### toValue(value: string, ctx?: any): Promise<any>

Преобразует внешнюю строку в фактическое значение интерфейса, которое может быть напрямую передано в репозиторий для операций записи.

### toString(value: any, ctx?: any)

Преобразует фактическое значение интерфейса в строковый тип, который может использоваться для экспорта или отображения.

