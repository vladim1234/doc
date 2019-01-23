# Интеграция с WEB frameworks

Администратор QOR имеет возможность интеграции с большинством веб-фреймворков golang. 
Несколько примеров того, как их интегрировать.

##  Интеграция с HTTP ServeMux

```go
mux := http.NewServeMux()
Admin.MountTo("/admin", mux)
http.ListenAndServe(":9000", mux)
```

##  Интеграция с Beego

```go
mux := http.NewServeMux()
Admin.MountTo("/admin", mux)

beego.Handler("/admin/*", mux)
beego.Run()
```

##  Интеграция с Gin

```go
mux := http.NewServeMux()
Admin.MountTo("/admin", mux)

r := gin.Default()
r.Any("/admin/*resources", gin.WrapH(mux))
r.Run()
```

##  Интеграция с gorilla/mux

```go
adminMux := http.NewServeMux()
Admin.MountTo("/admin", adminMux)

r := mux.NewRouter()
r.PathPrefix("/admin").Handler(adminMux)

http.Handle("/", r)
```
