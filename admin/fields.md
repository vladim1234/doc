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
Отображение для имени атрибута в ресурсе. Обычно это не устанавливается, и оно по умолчанию совпадает с `Name`.

Обычно это требуется, когда вы хотите использовать QOR Admin в качестве [RESTFul API Service](../admin/restful_api.md) и выставлять поля с другим именем, например:



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

  `Valuer` определяет, как извлекать значение поля из `object`, в результате он возвращает объект golang, QOR обычно отображает шаблон поля по-разному в зависимости от его значения и состояния.
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

Вы можете настроить QOR Admin для отображения «виртуальных» полей - полей, которые не являются атрибутами базы данных. 
Определите их как Meta для вашего ресурса, чтобы определить его для виртуального поля, при этом Valuer должен быть определен  (см. [Настройка Meta](#customize-meta)), поэтому администратор QOR знает, как отобразить его для конечного пользователя.

```go
product.Meta(&admin.Meta{Name: "MainImageURL", Valuer: func(record interface{}, context *qor.Context) interface{} {
  if p, ok := record.(*models.Product); ok && len(p.Images) > 0 {
    return p.Images[0].URL
  }
  return ""
}})
```

Если вы хотите использовать виртуальное поле в `NewAttrs`,` EditAttrs`, `ShowAttrs`, вам необходимо:

* Определить meta `Type`

  Тогда QOR Admin знает, какой шаблон использовать при рендеринге.

* определить мата `Setter`

  Тогда QOR Admin знает, как сохранить значение формы

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

