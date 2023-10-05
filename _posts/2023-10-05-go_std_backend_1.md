---
title: Go 기본 라이브러리 백엔드 - 개발기록 1
date: 2023-10-05 19:21:43 +/-TTTT
categories: [TIL, Summary]
tags: [golang, backend, standard library]
mermaid: true # TAG names should always be lowercase
---

## 프로젝트 구조 (23/10/05 기준)

```
📂cmd
┣ 📂cli
┃┗ 📎cli.go
┣ 📂db
┃┗ 📎db.go
┣ 📂web
┃ ┣ 📎api.go
┃ ┣ 📎application.go
┃ ┣ 📎helpers.go
┃ ┣ 📎middlewares.go
┃ ┣ 📎models.go
┃ ┣ 📎routes.go
┗ ┗ 📎template.go
📂internal
┣ 📂models
┃ ┣ 📎gists.go
┗ ┗ 📎users.go
📂ui
┣ 📂html
┃ ┣ 📂 pages
┃ ┃ ┣ 📄 home.tmpl.html
┃ ┃ ┗ 📄 view.tmpl.html
┃ ┃ 📂 partials
┃ ┃ ┗ 📄 nav.tmpl.html
┃ ┗ 📄 base.tmpl.html
┣ 📂static
┃ ┗📂css
┗ ┗ ┗📄 style.css
📎 main.go
✏ test.http
```

## CLI 커스텀 플래그 추가 (API / Template Mode)

이번 프로젝트에서는 일반적인 API와 홈페이지가 보이는 템플릿 모드 두가지를 지원해보고 싶었습니다.

이전에는 단순히 DB의 Config를 사용자에게 받는 것까지만 구현했지만, 여기서 더 나아가서 API 및 템플릿 두가지 모드를 지원 할 수 있도록 기능을 개선해봤습니다.

### 📎cli.go

```go
package cli

import (
	"flag"
	"io"
)

type Config struct {
	IsTemplateMode bool
	User           string
	Password       string
	Dbname         string
	Port           int
}

func ParseArgs(w io.Writer, args []string) (Config, error) {
	c := Config{}
	fs := flag.NewFlagSet("Gist Application", flag.ContinueOnError)
	fs.SetOutput(w)
	fs.BoolVar(&c.IsTemplateMode, "tmp", false, "Whether running application on Template Mode")
	fs.StringVar(&c.User, "user", "root", "Database Username")
	fs.StringVar(&c.Password, "password", "", "Database User Password")
	fs.StringVar(&c.Dbname, "dbname", "", "Database Name")
	fs.IntVar(&c.Port, "port", 3306, "MySQL port")

	err := fs.Parse(args)
	if err != nil {
		return c, err
	}
	return c, nil
}
```

`cli.go`는 사용자에게 받는 파라미터를 파싱해서 Config 모델을 넘겨주는 역할을 하고 있습니다. `flag.Parse()` 함수를 통해 하나의 파라미터만 받을 수도 있지만, `flag.NewFlagSet()`을 이용하면 여러가지의 인수를 받을수 있습니다.

`flag.NewFlagSet`이 가지는 장점은 다음과 같습니다.

- 자동으로 -help (-h) 파라미터를 파싱해서 사용자에게 Instruction을 보여줍니다.
- 여러 파라미터를 받아 보다 유연한 CLI Application을 만들 수 있습니다.
- Type casting 메소드가 지원되서 코드를 작성하기에 편합니다.

### 📎application.go / 📎 main.go

```go
// 📎application.go
type Application struct {
	ServerLogger   *log.Logger
	ErrorLogger    *log.Logger
	Gists          *models.GistModel
	Users          *models.UserModel
	TemplateCache  map[string]*template.Template
	IsTemplateMode bool
}
```

`📎application.go`는 전반적인 애플리케이션의 config를 설정하는 역할을 하고 있습니다. `Application` 모델에 `IsTemplateMode` 라는 필드를 정의해서 애플리케이션이 현재 템플릿 모드로 돌아가고 있는지 아닌지 판단을 할 수 있도록 했습니다.

`IsTemplateMode`는 Middleware에서 `Content-Type`을 정할때나, 라우팅의 모드를 바꿀때 사용합니다.

```go
// 📎main.go
c, err := cli.ParseArgs(os.Stderr, os.Args[1:])
	if err != nil {
		infoLog.Println(err)
		os.Exit(1)
	}

...

	app := &web.Application{
		ServerLogger:   infoLog,
		ErrorLogger:    errorLog,
		Gists:          &models.GistModel{DB: db},
		Users:          &models.UserModel{DB: db},
		IsTemplateMode: c.IsTemplateMode,
	}

	mux := app.InitRoutes()
	defaultMode := "API Mode"
	if app.IsTemplateMode {
		defaultMode = "Template Mode"
		templateCache, err := web.NewTemplateCache()
		if err != nil {
			errorLog.Fatal(err)
		}
		app.TemplateCache = templateCache
	}
```

`main.go`에서는 파싱한 파라미터를 `Application`모델에 넘겨주는 역할을 수행합니다.

### 📎routes.go

```go
// 📎routes.go
package web

import (
	"net/http"

	"github.com/gorilla/mux"
)

func (app *Application) InitRoutes() *mux.Router {
	mux := mux.NewRouter()
	mux.PathPrefix("/static/").Handler(http.StripPrefix("/static/", http.FileServer(http.Dir("./ui/static"))))
	if app.IsTemplateMode {
		mux.HandleFunc("/", app.TemplateHome)
		mux.HandleFunc("/gists/view/{id:[0-9]+}", app.TemplateViewOneGists)
		mux.HandleFunc("/gists/create", app.TemplateCreateGist)
	} else {
		s := mux.PathPrefix("/api").Subrouter()
		s.HandleFunc("/", app.ApiHome)
		s.HandleFunc("/gists/view", app.ApiViewLatestGists)
		s.HandleFunc("/gists/view/{id:[0-9]+}", app.ApiViewOneGists)
		s.HandleFunc("/gists/create", app.ApiCreateGist)
	}
	mux.NotFoundHandler = http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		app.NotFound(w)
	})
	mux.Use(app.RecoverPanics)
	mux.Use(app.HTTPLogger)
	mux.Use(app.ContentTypeHeader)
	mux.Use(app.SecureHeaders)
	return mux
}
```

`routes.go`는 애플리케이션의 route를 관리해주는 역할을 하고 있습니다. `Application` 모델의 메소드로 정해놓은 덕택에 손쉽게 두가지 모드의 route를 관리할 수 있습니다.

## DB Connect / Access Layer

이번 프로젝트에서는 최대한 기본 라이브러리를 이용해서 백엔드를 구현하는 것이 목표이기 때문에 DB access와 sql 모두 Go의 기본 라이브러리를 활용해서 구현했습니다.

### 📎db.go

```go
// 📎db.go
package db

import (
	"database/sql"
	"fmt"

	"github.com/go-sql-driver/mysql"
	"github.com/milkymilky0116/go-std-backend/cmd/cli"
)

func DBinit(c cli.Config) (*sql.DB, error) {
	cfg := mysql.Config{
		User:      c.User,
		Passwd:    c.Password,
		Net:       "tcp",
		Addr:      fmt.Sprintf("127.0.0.1:%d", c.Port),
		DBName:    c.Dbname,
		ParseTime: true,
	}
	db, err := sql.Open("mysql", cfg.FormatDSN())
	if err != nil {
		return nil, err
	}
	pingErr := db.Ping()
	if pingErr != nil {
		return nil, pingErr
	}
	return db, nil
}
```

DB를 활용하려면 당연히 DB에 연결하는 과정도 필요하겠죠. CLI를 통해 받은 파라미터들을 다시 MySQL의 Config에 넘겨주고, DB를 연결합니다.

추가적으로 핑 테스트도 해서 DB와 제대로 연결이 됐는지 확인한 뒤에 db를 반환합니다.

### 📎gists.go

```go
// 📎gists.go
package models

import (
	"database/sql"
	"errors"
	"time"
)

type Gist struct {
	Id        int       `json:"id"`
	Title     string    `json:"title"`
	Content   string    `json:"content"`
	Writer    User      `json:"writer"`
	CreatedAt time.Time `json:"created_at"`
	UpdatedAt time.Time `json:"updated_at"`
}

type GistParam struct {
	Title   string `json:"title"`
	Content string `json:"content"`
	Writer  int    `json:"writer"`
}

type GistModel struct {
	DB *sql.DB
}

func (m *GistModel) FindOne(id int) (*Gist, error) {
	var result Gist
	var gistId, writerId int
	var title, content string
	var created_at, updated_at time.Time
	statement := "SELECT * FROM gist WHERE id = ?"
	row := m.DB.QueryRow(statement, id)

	if err := row.Scan(&gistId, &title, &content, &writerId, &created_at, &updated_at); err != nil {
		if errors.Is(err, sql.ErrNoRows) {
			return &result, sql.ErrNoRows
		}
		return &result, err
	}
	var userModel = &UserModel{DB: m.DB}
	user, err := userModel.findUser(writerId)
	if err != nil {
		return &result, err
	}
	result = Gist{
		Id:        gistId,
		Title:     title,
		Content:   content,
		Writer:    *user,
		CreatedAt: created_at,
		UpdatedAt: updated_at,
	}
	return &result, nil
}

func (m *GistModel) FindMany(limit int) ([]*Gist, error) {
	gists := []*Gist{}
	statement := "SELECT id FROM gist ORDER BY created_at DESC LIMIT ?"
	rows, err := m.DB.Query(statement, limit)
	if err != nil {
		return nil, err
	}
	defer rows.Close()
	for rows.Next() {
		var gistId int
		gist := &Gist{}
		if err := rows.Scan(&gistId); err != nil {
			return nil, err
		}
		gist, err = m.FindOne(gistId)
		if err != nil {
			return nil, err
		}
		gists = append(gists, gist)
	}
	if err = rows.Err(); err != nil {
		return nil, err
	}
	return gists, err
}

func (m *GistModel) Insert(params GistParam) (int, error) {
	statement := "INSERT INTO gist (title, content, writer) VALUES (?, ?, ?)"
	result, err := m.DB.Exec(statement, params.Title, params.Content, params.Writer)
	if err != nil {
		return 0, err
	}
	id, err := result.LastInsertId()
	if err != nil {
		return 0, err
	}
	return int(id), nil
}
```

`gists.go` 에서는 Gist의 모델과 SQL 쿼리를 수행하는 DB Access Layer를 정의합니다.

SQL 쿼리가 single row 일때는 `QueryRow()` 메소드를,
SQL 쿼리가 multiple row 일때는 `Query()` 메소드를,
`DELETE, UPDATE` 와 같이 반환하는 값이 없을때는 `Exec()` 메소드를 사용합니다.

받아온 데이터들을 go의 값으로 받기 위해서 반드시 `row.Scan()` 메소드를 사용해서 값들을 받아오는데 이상 없는지 에러 체크를 해줘야 합니다.
