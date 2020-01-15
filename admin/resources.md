# Resources

Ресурсы - это то, что можно администрировать через пользовательский интерфейс QOR Admin, часто это модель GORM (таблица БД).

## добавление ресурса в QOR Admin

Например, имеем модель, по которой создана таблица в БД:
```go
// GORM-backend model
type User struct {
  gorm.Model
  Email     string
  Password  string
  Name      sql.NullString
  Gender    string
  Role      string
  Addresses []Address
}
```
Добавляем ее в интерфейс Qor Admin:

```go
user := Admin.AddResource(&User{}, &admin.Config{Menu: []string{"User Management"}})
```

## Resource Configuration

Для настройки ресурса доступные опции в `admin.Config`:

| Name       | Type              | Default | Description                                                                                         |
| ---        | ---               | ---     | ---                                                                                                 |
| Name       | string            |         | Отображаемое имя ресурса                                                                        |
| Menu       | []string          |         | Настройка меню для ресурса, см. [Menus](/admin/theming_and_customization.md#menus) |
| Permission | *roles.Permission |         | Управление доступом для ресурса, см. [Roles](/admin/authentication.md#authorization-for-resource)  |
| Themes     | []ThemeInterface  |         | Установка индивидуальную тему для ресурса [индивидуальной темы](/admin/theming_and_customization.md#themes) для ресурса              |
| Priority   | int               |         | Определяет порядок сортировки записей в меню, производиться как по возрастанию (ASK)                                                |
| Singleton  | bool              | false   |Задать ресурс - это один объект или несколько объектов. e.g. "SEO setting" vs "Users". При установке в true вместо таблицы будет отображаться форма для редактирования записи.           |
| Invisible  | bool              | false   | Установливает, отображается ли ресурс в меню                                                         |
| PageCount  | int               | 20      | Определяет, сколько записей будет отображаться на странице                                   |
