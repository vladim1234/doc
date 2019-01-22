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
| Themes     | []ThemeInterface  |         | Set [customized theme](/admin/theming_and_customization.md#themes) for the resource               |
| Priority   | int               |         | Control the display sequence in menu, ordered by ASC                                                |
| Singleton  | bool              | false   | Set the resource is a single object or multiple objects. e.g. "SEO setting" vs "Users"              |
| Invisible  | bool              | false   | Set whether the resource is visible in menu                                                         |
| PageCount  | int               | 20      | Pagination setting, Set how many records shall be shown per page                                    |
