## Общая конфигурация

Вы можете настроить Admin с `AdminConfig` структурой при инициализации, используя следующие параметры:

```go
type AdminConfig struct {
  SiteName       string
  DB             *gorm.DB
  Auth           Auth
  SessionManager session.ManagerInterface
  I18n           I18n
  AssetFS        assetfs.Interface
  *Transformer
}
```

* <a id="sitename"></a>SiteName

  По умолчанию, для Site Name установлено значение `Qor Admin`, вы можете использовать `SiteName` для изменения названия сайта.

  ```go
  Admin := admin.New(&admin.AdminConfig{SiteName: "Qor Example"})
  ```

  Совет: [You can inject stylesheets, javascripts into your admin site with your site's title](/admin/theming_and_customization.md#global-stylesheet--javascript)

* DB

  `DB` is a GORM DB connection, требуестся для определения соединения с БД

* Auth

  Auth используется для [Authentication](/admin/authentication.md)

* SessionManager

  Admin использует SessionManager для управления cookies, flash messages, learn to [customize it](/admin/session_manager.md)

* I18n

  [Internationalization](/admin/i18n.md) solution for Admin

* <a id="assetfs"></a>AssetFS

  AssetFS определяет, как искать шаблоны при рендеринге страниц, см. [view paths]((/admin/theming_and_customization.md) для более подробной информации, при развертывании вашего сайта в производство, вы можете сделать ваше приложение автономно исполняемым, см. [Deploy to production](/admin/deploy.md).

* Transformer

  [Transformer for RESTFul API](/admin/restful_api.md#transformer)

## Dashboard

QOR Admin по умолчанию предоставляет страницу панели инструментов  с фиктивным текстом.. Если вы хотите настроить панель, вы можете создать файл `dashboard.tmpl` в [QOR view paths](/admin/theming_and_customization.md#view-paths), QOR Admin загрузит его как шаблоны Golang при отображении страницы панели инструментов.

Если вы хотите отключить dashboard, вы можете перенаправить ее на другую страницу, например:

```go
Admin.GetRouter().Get("/", func(c *admin.Context) {
  http.Redirect(c.Writer, c.Request, "/admin/clients", http.StatusSeeOther)
})
```
