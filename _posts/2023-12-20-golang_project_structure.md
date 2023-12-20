---
title: Golang으로 프로젝트 구조 설계해보기
date: 2023-12-20 19:18:43 +/-TTTT
categories: [golang, structure]
tags: [golang]
mermaid: true # TAG names should always be lowercase
---

![gopher](https://github.com/milkymilky0116/obsidian-backup/assets/84823612/5ab5d678-2844-4e68-a2f3-938a2ec0d091)

최근에 Golang으로 사이드 프로젝트를 하나 시작했다. 실제로 Golang으로 본격적인 프로젝트를 진행해 본 적은 처음이었기에 프로젝트 세팅부터 많은 난관이 있었다. 그중 가장 어려웠던 것은 '프로젝트 구조 설계'였다. (그리고 지금도 여전히 고민 중이다..)

Golang과 Golang의 여러 웹 라이브러리들 (gin, echo, fiber, 혹은 기본 라이브러리 등등)은 프로젝트 구조를 강제하지 않는다. Handler를 프로젝트에 어디에 위치 시킬지, 어떤 식으로 라우터를 취합해서 서버를 실행시킬지는 전적으로 사용자의 손에 달려있다. 그리고 Golang의 프로젝트 구조에 대해 여러 아티클을 찾아 본 결과, 사람들이 관습적으로 사용하는 구조는 있어도 뚜렷한 '정답'은 없었다. 프로젝트의 구조는 전적으로 프로젝트의 성격, 스펙에 달려있고 그 프로젝트에 맞는 구조를 설계해야 하는 것은 오롯이 개발자의 몫이다.

이 글은 사이드 프로젝트를 만들어보면서 내가 나름대로 프로젝트 구조에 대해 고민하고 왜 이렇게 구성했는지 정리해 보기 위해 쓰였다. 당연히 이 글에 나온 프로젝트 설계가 정답이라고 생각하지 않고, 아마 Golang을 실제로 현업에서 쓰시는 분들 입장에서는 갸우뚱할만한 포인트들도 많을 것이라고 생각한다. 그럼에도 프로젝트를 설계하면서 가졌던 여러 생각들을 공유하는 것은 꽤 괜찮은 일이라고 생각하고, 나중에 또 다른 프로젝트를 시작할 때 더 발전시킬 수 있는 기회가 될 것이라고 생각해서 글을 쓰게 되었다.

### 관습에서 시작하기

위에서도 언급했듯 Golang의 프로젝트 설계에는 정답이 없다. 하지만 많은 Golang 개발자들이 따라가는 관습적인 큰 틀은 존재한다.

https://github.com/golang-standards/project-layout

스타가 무려 43.5k나 찍혀있는 이 레포지토리는 많은 Golang 개발자들, 혹은 서적이나 튜토리얼에서 따라가고 있는 일반적인 구조이고 프로젝트 설계에 어려움을 겪고 있다면 충분히 참고 할만한 레포지토리이다.

물론 레포지토리의 README를 자세히 읽어보면 알 수 있듯, 이 레포지토리의 구조가 완전한 '정답'은 아니다. README의 서두를 보면 알 수 있듯 처음 Golang을 배우는 사람들은 매우 심플한 것 (가령 `main.go` 하나부터 시작해 보는 것)에서 시작하는 것이 좋다고 언급하고 있고, 프로젝트가 커지고 관리법이 필요하다고 여겨질 때 도입해 보라고 추천하고 있다. 그리고 개발하면서 필요하다고 여겨지는 것들만 남기고 나머지는 삭제해도 무방하다고 언급하고 있다. 위에서도 언급했듯, Golang의 프로젝트 구조는 프로젝트의 성격과 스펙에 따라서 많이 달라질 수 있다. 그러니 위의 레포지토리나 지금 보고 있는 글과 같이 프로젝트 설계에 대해 '참고' 하는 것은 좋아도, 절대적인 '정답'으로 여겨서는 안된다는 것이다.

여러 아티클, 서적, 튜토리얼들을 참고해본 결과 주요하게 쓰이는 구조는 다음과 같았다.

### `/cmd`

프로젝트에서 최종적으로 실행되는 메인 애플리케이션을 담는 디렉토리이다. cmd 폴더에 라우터나 핸들러를 넣는 경우도 봤는데, 개인적으로 생각하기에는 cmd 폴더에 담겨있는 파일은 최대한 작게 유지하는 것이 좋다고 생각한다. 코드가 지나치게 난잡해질 수도 있고, 애플리케이션의 주요한 로직을 담는 것은 cmd 폴더의 취지와는 맞지 않는다고 생각하기 때문이다.

### `/internal`

internal 폴더에는 다른 사람들이 임포트 하기 원치 않는 주요한 애플리케이션 코드들이 들어가면 좋다. 주로 핵심적인 비즈니스 로직이나 그 프로젝트 자체에서만 쓰이는 코드들을 internal 폴더에 넣어서 정리하면 프로젝트의 구조가 매우 깔끔해진다. internal 폴더는 `Go 1.4` 부터 컴파일러가 캐치 할 수 있도록 되었기 때문에 프로젝트에 internal 디렉토리를 넣는 것은 꽤 고려해볼 만한 일이다.

### `/pkg`

pkg 폴더는 internal과 반대로 외부에서 사용해도 괜찮은 코드들이 들어가면 좋다.

### 프로젝트 소개를 시작하기 전에

아래에 서술될 프로젝트는 [Puddlee backend](https://github.com/PUDDLEEE/puddleee_back/tree/feat/17/auth) 에서 볼 수 있다.

해당 프로젝트에서 쓰인 라이브러리들은 다음과 같다.

- [Cobra](https://github.com/spf13/cobra)- CLI 툴
- [Viper](https://github.com/spf13/viper)- configuration 툴
- [Gin](https://github.com/gin-gonic/gin) - http 라이브러리
- [ent](https://entgo.io/)- 데이터베이스 ORM
- [mockery](https://github.com/vektra/mockery)- 테스팅을 위한 mocking 툴
- [swag](https://github.com/swaggo/swag)- Golang을 위한 swagger 라이브러리

### 프로젝트 시작점

애플리케이션을 어떻게 시작하면 좋을까? 필자는 위에서도 언급했듯 cmd 디렉토리를 비롯한 프로젝트의 엔트리 포인트는 최대한 작게 유지하는 것이 좋다고 생각한다. 애플리케이션이 실행될 `main.go`와 cmd 디렉토리는 전적으로 프로그램이 실행될 로직들만 담는 것이 좋다고 생각해서, 해당 프로젝트의 `main.go`와 cmd 디렉토리는 매우 심플하게 관리되고 있다.

**main.go**

```go
package main

import (
	"github.com/PUDDLEEE/puddleee_back/cmd"
	_ "github.com/PUDDLEEE/puddleee_back/docs"
)
//...

func main() {
	cmd.Execute()
}
```

Execute 메소드는 Cobra 라이브러리를 활용해서 CLI 커맨드를 호출할 수 있도록 만들어져 있다. cmd 디렉토리 안에 `root.go` 는 애플리케이션의 설명을 (일반적인 -h, -help 플래그와 같이) , `serve.go`는 실제로 애플리케이션이 실행될 커맨드를 담고 있다.

**root.go**

```go
package cmd

import "github.com/spf13/cobra"

var rootCmd = &cobra.Command{
	Use:   "puddlee",
	Short: "Puddlee is a Chat Service for developer.",
}

func init() {
	rootCmd.AddCommand(serveCmd)
}

func Execute() error {
	return rootCmd.Execute()
}
```

**serve.go**

```go
package cmd

import (
	"github.com/PUDDLEEE/puddleee_back/internal/app"
	"github.com/spf13/cobra"
)

var serveCmd = &cobra.Command{
	Use:   "serve",
	Short: "Run Puddlee Backend Server",
	Long: `Run Puddlee Backend Server on local.
	configuration file lies on the config directory.`,
	Run: func(cmd *cobra.Command, args []string) {
		app.InitApp()
	},
}
```

`serve.go` 코드를 살펴보면, `app.InitApp()` 으로 프로젝트가 실행이 되고 있다는 것을 알 수 있다. `app` 패키지는 internal 디렉토리 안에 있고, 애플리케이션이 실행될 수 있도록 각종 설정 (db 설정, 라우터 설정 등등...) 을 관리하는 역할을 하고 있다.

### 이 많은 패키지들을 어떻게 한데 묶어야 할까?

현재 구현중인 프로젝트에서, internal 폴더의 구조는 다음과 같다.

```
📂internal
┣ 📂 app
┣ 📂 auth
┣ 📂 db
┣ 📂 errors
┣ 📂 interceptors
┣ 📂 jwtAuth
┣ 📂 mail
┣ 📂 middlewares
┣ 📂 room
┗ 📂 users
```

폴더만 봐도 알 수 있듯, internal 디렉토리에는 애플리케이션에서 가장 핵심적인 로직들이 담겨있다. 이 많은 패키지들을 어떻게 한데 묶어서 실행시킬 수 있을까?

![golang-project drawio](https://github.com/milkymilky0116/obsidian-backup/assets/84823612/36ec417b-ef4b-4e4f-9a3a-8bd62ab74412)

핵심은 패키지들이 `InitXXX()` 로 initializing 되고 있고, 모든 패키지들이 App 으로 전달되고 있는 구조라는 것이다. App에서는 Config, Routes, DB 를 관리하기 위해 따로 struct를 만들어 관리하고, 메소드 `InitApp()` 으로 서버를 실행시키는 구조를 설계했다.

Application struct의 구조는 다음과 같다.

```go
type Application struct {
	Routes Routes
	Config *config.Config
	Client *ent.Client
}
```

이런 식으로 struct를 만들게 되면, 애플리케이션의 전반적인 설정을 관리하기 쉬워지게 된다. 어떤 컨트롤러에서 Application struct 내의 Config나 Client를 필요로 할 경우 그 필드를 인자로 넘길수 있게 된다. 해당 컨트롤러에서 `initConfig`나 `initDB` 등을 번거롭게 호출할 필요성이 없어지고, dependency에 관한 관리가 더 쉬워지게 된다.

`internal/app/init.go`은 위에서 설명한 점을 구현하고 있다.

**internal/app/init.go**

```go
package app

import (
	"fmt"
	"net/http"

	"github.com/PUDDLEEE/puddleee_back/internal/db"
	"github.com/PUDDLEEE/puddleee_back/pkg/config"
)

func InitApp() {
	app := &Application{
		Config: config.InitConfig(),
	}
	app.Client = db.InitDB()
	defer app.Client.Close()
	app.initRoutes()
	app.run()
}

func (app *Application) run() {
	addr := fmt.Sprintf("%s:%d", app.Config.Server.Host, app.Config.Server.Port)
	srv := &http.Server{
		Addr:    addr,
		Handler: app.Routes.Router,
	}
	srv.ListenAndServe()
}
```

### Singleton 패턴으로 패키지 initializing 하기

위의 그래프에서 보듯 Config, DB, Routes는 `InitXXX()`로 초기화 되고 있다. 각각의 패키지는 앱을 실행할때 단 한번만 initialize 되야하는 상황이다. 이러한 상황에서 Singleton 패턴보다 더 적절한 패턴은 없어보인다.

Golang에서 Singleton 패턴은 `sync` 패키지를 이용해 구현 할 수 있다. `sync.Once` 타입을 이용하면 애플리케이션 내에서 어떤 상황을 단 한번만 호출할 수 있도록 도움 받을 수 있다.

**internal/app/routes.go**

```go
package app

//...

var once sync.Once
var routes Routes

func (app *Application) initRoutes() {
	if routes.Router == nil {
		once.Do(func() {
			app.setRoutes()
		})
	} else {
		log.Fatal("Router already configure")
	}
	app.Routes = routes
}

//...
```

Config의 initialize는 이렇게 구현되었다.

**pkg/config/config.go**

```go
package config

import (
	"log"
	"sync"

	"github.com/spf13/viper"
)

var once sync.Once

var config *Config

func InitConfig() *Config {
	if config == nil {
		once.Do(func() {
			err := setConfig()
			if err != nil {
				log.Fatal(err)
			}
		})
	} else {
		log.Fatal("Config already set.")
	}
	return config
}

func setConfig() error {
	viper.AddConfigPath("./config")
	viper.SetConfigType("yaml")
	if err := viper.ReadInConfig(); err != nil {
		return err
	}
	if err := viper.Unmarshal(&config); err != nil {
		return err
	}
	return nil
}

```

그리고 Application struct의 필드를 활용해서 컨트롤러들을 initalize 할수도 있다. 아래의 코드에서는 `setRoutes()` 를 `Application` 의 메소드로 만들어서, Application struct 안에 있는 필드들을 활용 할 수 있도록 만들었다. 다만 아래 코드는 더 깔끔하게 처리할 여지가 있어서 좀 더 리팩토링이 필요한 상황이다.

```go
// ...
func (app *Application) setRoutes() {
	routes.Router = gin.Default()
	routes.Router.Use(middlewares.ErrorHandler())
	routes.Router.Use(cors.Default())
	v1 := routes.Router.Group("/api/v1")
	userController := user.InitializeController(context.Background(), app.Client)
	roomController := room.InitializeController(context.Background(), app.Client)

	mailParams := &mail.MailServiceParams{}
	mailParams.From = app.Config.Mail.From
	mailParams.Port = app.Config.Mail.Port
	mailParams.Server = app.Config.Mail.Server
	mailParams.Password = app.Config.Mail.Password

	authController := auth.InitializeController(context.Background(), app.Client, app.Config.Jwt.SecretKey, *mailParams)
	routes.addSwagger(v1)
	routes.addUser(v1, userController)
	routes.addRoom(v1, roomController)
	routes.addAuth(v1, authController)
}

//...
```

### Service, Controller, Repository 구조

Service, Controller, Repository 구조를 채택해서 얻는 이점은, 비즈니스 로직에 대해 관심사를 분리시켜 더 유지보수 가능한 코드를 작성 가능하다는 것이다.

- Repository: DB에서 데이터를 가공하는 역할
- Service: 핵심적인 비즈니스 로직을 담는 역할
- Controller: 실제로 API가 호출될때 에러 핸들링 등을 담당하는 역할

User, Room, Auth.. 와 같은 애플리케이션의 구조는 모두 아래와 같이 동일하다.

```
📂 user
┣ 📂 dto
┣ 📎 controller.go
┣ 📎 controller_test.go
┣ 📎 initController.go
┣ 📎 repository.go
┣ 📎 repository_test.go
┣ 📎 service.go
┗ 📎 service_test.go
```

Controller, Service, Repository 에 대한 dependency는 `initController.go` 에서 처리하고 있다.

**internal/user/initController.go**

```go
func InitializeController(ctx context.Context, client *ent.Client) *UserController {
	userRepository := NewUserRepository()
	userService := NewService(userRepository, ctx, client)
	userController := NewController(userService)
	return userController
}
```

여기서 `NewService()`, `NewController()` 에서는 직접적으로 struct를 받지 않는다. 왜냐하면 테스팅을 위해서 mocking한 인자를 받아야 할 수도 있기 때문이다. 그래서 대신 `pkg/interfaces` 에 선언된 인터페이스를 인자로 받고, Type Assertion을 통해 발생할 수 있는 에러를 처리해주고 있다.

**internal/user/service.go**

```go
func NewService(repo interfaces.IUserRepository, ctx context.Context, client *ent.Client) *UserService {
	if userRepository, ok := repo.(*UserRepository); ok {
		return &UserService{userRepository: userRepository, ctx: ctx, client: client}
	}

	if userRepository, ok := repo.(*mocks.IUserRepository); ok {
		return &UserService{userRepository: userRepository, ctx: ctx, client: client}
	}
	return nil
}
```

**pkg/interfaces/IUserRepository.go**

```go
package interfaces

import (
	"context"

	"github.com/PUDDLEEE/puddleee_back/ent"
	userdto "github.com/PUDDLEEE/puddleee_back/internal/user/dto"
)

type IUserRepository interface {
	Create(context.Context, *ent.Client, userdto.CreateUserDTO) (*ent.User, error)
	FindOneById(context.Context, *ent.Client, int) (*ent.User, error)
	FindOneByEmail(context.Context, *ent.Client, string) (*ent.User, error)
	Update(context.Context, *ent.Client, int, userdto.UpdateUserDTO) (*ent.User, error)
	Delete(context.Context, *ent.Client, int) error
}
```

Service, Repository에 명시된 메소드들은 `pkg/interfaces` 에서 정의된 인터페이스의 구조체를 따르고 있고, 이를 Decorator 패턴이라고 한다. 앞서 말했듯 본래 인자와 테스팅을 위한 mocking한 인자 모두 받아야 하기 때문에 이를 위해서 Decorator 패턴을 적용했다.

### 결론

아직 프로젝트는 완성되지 않았지만, 약 1~2주간의 기간동안 프로젝트를 세팅하면서 어떻게 하면 더 괜찮은 구조로 구성할 수 있을지 고민했고 결론적으로 지금 까지 나온 결과물에 만족한다!

물론 더 나아질 수 있는 여지가 있겠지만 코드를 명확하게 관심사대로 분리할 수 있었고, 프로젝트의 확장성도 용이해서 꽤 만족스럽게 개발 할 수 있었다.

하지만 여기서 끝이 아니라고 생각하고, 프로젝트 구조에 관한 자료를 찾아보면서 더 나은 개선 방향을 찾아볼것이다.
