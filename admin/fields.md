# Fields

## Настройка видимых полей

По умолчанию Qor отображает все поля определенные в структуре. Если специально объявить поля, будут видны только определенные поля, и они будут представлены в указанном порядке:
```go
// Add resources `Order`, `Product` to Admin:
order := Admin.AddResource(&models.Order{})
product := Admin.AddResource(&models.Product{})
```
Показать перечисленные атрибуты на основной странице:
```go
order.IndexAttrs("User", "PaymentAmount", "ShippedAt", "CancelledAt", "State", "ShippingAddress")
```
показать все атрибуты на основной странице, кроме `State`:
```go
order.IndexAttrs("-State")
```
Показать перечисленные атрибуты на странице добавления:
```go
order.NewAttrs("User", "PaymentAmount", "ShippedAt", "CancelledAt", "State", "ShippingAddress")
```
показать все атрибуты, кроме `State` на странице добавления:
```go
order.NewAttrs("-State")
```
Поля на новой форме можно разбить на несколько разделов, например Basic Information и Organization:
```go
// Structure the new form to make it tidy and clean with `Section`
product.NewAttrs(
  &admin.Section{
    Title: "Basic Information",
    Rows: [][]string{
      {"Name"},
      {"Code", "Price"},
    }
  },
  &admin.Section{
    Title: "Organization",
    Rows: [][]string{
      {"Category", "Collections", "MadeCountry"},
    }
  },
  "Description",
  "ColorVariations",
}
```
Показать перечисленные атрибуты на странице редактирования:
```go
order.EditAttrs("User", "PaymentAmount", "ShippedAt", "CancelledAt", "State", "ShippingAddress")
```
Показать перечисленные атрибуты на странице отображения:
```go
order.ShowAttrs("User", "PaymentAmount", "ShippedAt", "CancelledAt", "State", "ShippingAddress")
```
Если ShowAttrs не были настроены, страница показа не будет сгенерирована, вместо нее будет показана форма редактирования.

## Настройка поля для вложенных ресурсов

```go
order := Admin.AddResource(&models.Order{})
orderItemMeta := order.Meta(&admin.Meta{Name: "OrderItems"})
orderItemResource := orderItemMeta.Resource

orderItemResource.EditAttrs("ProductCode", "Price", "Quantity")
```

## Meta

Meta предназначена для обработка (преобразования) полей перед их отображением или после отправки из формы. По умолчанию поля ресурса отображаются на основе его типов и отношений. при необходимости  вы можете настроить рендеринг, переписав meta определение.

Существует несколько предопределенных типов `Meta`, в том числе `string`, `password`, `date`, `datetime`, `rich_editor`, `select_one`, `select_many`, ([более полный список](#common_meta_types)).

[QOR Admin](../admin/README.md) автоматически выберет тип для `Meta` на основе типа данных поля. Например, если тип поля `time.Time`, [QOR Admin](../admin/README.md) он будет определен как `datetime` тип.

Например, для поля «Gender» качестве элемента выбора в форме пользователя, который включает в себя два варианта: "Male", "Female".

```go
user.Meta(&admin.Meta{Name: "Gender", Config: &admin.SelectOneConfig{Collection: []string{"Male", "Female"}}})
```

### Настройка Meta

Если конфигурация поля по умолчанию не соответствует вашим потребностям, вы хотите настроить поле:

```go
type Meta struct {
    Name            string
    FieldName       string
    Label           string
    Type            string
    Setter          func(object interface{}, metaValue *resource.MetaValue, context *qor.Context)
    Valuer          func(object interface{}, context *qor.Context) (value interface{})
    FormattedValuer func(object interface{}, context *qor.Context) (formattedValue interface{})
    Permission      *roles.Permission
    Config          MetaConfigInterface
    Collection      interface{}
    Resource        *Resource
}
```

* Name

  Имя поля, которое требуется перезаписать.
Например, перезапишем поле «Gender» качестве элемента выбора в форме пользователя, который включает в себя два варианта: "Male", "Female".
  ```go
  user.Meta(&admin.Meta{Name: "Gender", Config: &admin.SelectOneConfig{Collection: []string{"Male", "Female"}}})
  ```

* FieldName

  Mapping to the attribute name in the resource, Usually, This is no need to set and same with `Name` by default.

  This is usually needed when you want to use QOR Admin as [RESTFul API Service](../admin/restful_api.md) and expose fields with a different name, e.g:

  ```go
  // Generate JSON { "Code": "value-of-ExternalCode", ... }
  order.Meta(&admin.Meta{Name: "Code", FieldName: "ExternalCode"})
  ```

* Type

  Тип отображения атрибута, например: `select_one`, `password`. [полный список предопределенных мета-типов](#common_meta_types)

* Label

 Название атрибута в форме и заголовок таблицы для страницы индекса.

  По умолчанию в качестве названия «address» используется «Address». С помощью этой опции вы можете установить другое название для «address».

* Setter

  ```go
  // Setter's type
  func(record interface{}, metaValue *resource.MetaValue, context *qor.Context)
  ```

`Setter` определяет, как декодировать значение полученное из формы в поле, по умолчанию` Setter` генерируется администратором QOR, он получает значение формы из `metaValue` и декодирует это значение в` record` в зависимости от типа поля.

* <a id="valuer"></a>Valuer

  ```go
  // Value's type
  func(record interface{}, context *qor.Context) (result interface{})
  ```

  `Valuer` определяет, как извлекать значение поля из `object`, it returns a golang object as result, QOR usually will render field's template differently based on its value and state.

* FormattedValuer

  ```go
  // FormattedValuer's type
  func(record interface{}, context *qor.Context) (result interface{})
  ```

`FormattedValuer` похож на `Valuer`, но обычно возвращает форматированную строку как результат, он будет показан конечному пользователю на странице индекса и API


* Resource

  Это может быть использовано для настройки атрибутов во вложенной форме, обычно это не нужно устанавливать, подробнее: [как настроить атрибут во вложенной форме](../metas/collection-edit.md).

* Permission

  Определяет настройку полномочий пользователя для этого атрибута, см. [Authentication](/admin/authentication.md#authorization-for-fields) for detail.

* Config

  Конфигурация текущего типа атрибута, например: `Config: &admin.SelectOneConfig{Collection: []string{"Male", "Female", "Unknown"}}`. 

### Virtual Field

You can configure QOR Admin to display "virtual" fields - fields that are not database attributes. Just define them as `Meta` to your resource, to define it for a virtual field, `Valuer` is must be required (refer [Customize Meta](#customize-meta)), so QOR Admin knows how to display it to end user.

```go
product.Meta(&admin.Meta{Name: "MainImageURL", Valuer: func(record interface{}, context *qor.Context) interface{} {
  if p, ok := record.(*models.Product); ok && len(p.Images) > 0 {
    return p.Images[0].URL
  }
  return ""
}})
```

If you want to use the virtual field in `NewAttrs`, `EditAttrs`, `ShowAttrs`, you have to:

* Define meta's `Type`

  Then QOR Admin knows which template to use when rendering it

* Define meta's `Setter`

  Then QOR Admin knows how to save form's value

e.g:

```go
user.Meta(&admin.Meta{Name: "Password",
    Type:   "password",
    Valuer: func(interface{}, *qor.Context) interface{} { return "" },
    Setter: func(record interface{}, metaValue *resource.MetaValue, context *qor.Context) {
        if newPassword := utils.ToString(metaValue.Value); newPassword != "" {
            bcryptPassword, _ := bcrypt.GenerateFromPassword([]byte(newPassword), bcrypt.DefaultCost)
            record.(*models.User).EncryptedPassword = string(bcryptPassword)
        }
    },
})
```

## Common Meta types

{% include "/admin/common_meta_types.md" %}

