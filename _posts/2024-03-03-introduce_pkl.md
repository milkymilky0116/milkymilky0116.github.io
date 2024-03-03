---
title: (번역) 애플에서 만든 새로운 Configuration 언어, Pkl🥒 을 소개합니다.
date: 2024-03-03 22:04:43 +/-TTTT
categories: [translation, pkl]
tags: [translation, pkl]
mermaid: true # TAG names should always be lowercase
---

# 소개 🥒

우리는 새로운 오픈 소스 프로젝트, Pkl (발음은 피클)을 소개하게 되어 기쁩니다. Pkl은 프로그램의 설정 (Configuration) 을 위한 새로운 프로그래밍 언어입니다.

개발자로서 프로그램의 설정 (Configuration)을 생각할 때, 흔히 정적 언어인 JSON, YAML, 혹은 단순한 속성 목록을 떠올리기 쉽습니다. 이러한 정적 언어들은 물론 그들만의 메리트가 있지만, 설정이 복잡해질 때에는 늪에 빠지기 쉽습니다. 예를 들어 그러한 언어의 부족한 표현식은 종종 코드 반복을 일으킵니다. 또한 그 언어들은 스스로 검증을 하는 단계를 거치지 않기 때문에 설정에서 에러를 발생시키기 매우 쉽습니다.

이러한 단점을 해결하기 위해, 가끔 그러한 포맷들은 특별한 로직을 더하기 위해 보조 툴들을 이용하기도 합니다. 예를 들어, 코드를 반복하지 않기 위해 참조를 해결하고 객체를 합치는 방법을 이해하는 특수한 속성이 도입되기도 합니다. 또는 유효성 검사에 대비해야 하는 필요성이 생긴 경우, 구성된 값이 예상된 타입과 일치하는지 검증하는 새로운 방법이 도입될 수도 있습니다. 얼마 지나지 않아 이러한 포맷들은 거의 프로그래밍 언어처럼 변하지만 작성하기에도, 이해하기에도 어려운 형태로 변하게 됩니다.

다른 관점에서, 범용 언어가 대신 사용될 수도 있습니다. Kotlin, Ruby, Javascript는 설정 데이터를 생성하는 DSL(Domain-specific language)의 기초가 될 수 있습니다. 이러한 언어들은 엄청나게 강력하지만, 설정을 작성하는 데 있어서 어려움을 겪을 수도 있습니다. 왜냐하면 이러한 언어들은 설정을 정의하고 검증하는데 만들어진 언어가 아니기 때문입니다. 추가적으로 이러한 DSL들은 그들 고유의 생태계에 묶여있습니다. Kotiln의 DSL을 Go로 작성된 애플리케이션의 설정 레이어에 적용시키기 어려운 것처럼 말입니다.

우리가 Pkl 언어를 만든 이유는, 우리는 설정이 정적 언어 (JSON, YAML...) 과 범용 언어 사이 그 어딘가에 위치한다고 생각하기 때문입니다. 우리는 그 두 개의 세계에서 최선을 선택하고 싶었습니다. 선언적이고, 쓰고 읽기에 쉽지만 범용 언어에서 차용한 기능들로 발전된 형태의 언어를 원했습니다. Pkl을 작성할 때, 여러분은 언어에서 기대하는 기능들을 쓸 수 있습니다. 클래스, 함수, 조건문, 반복문 등이 그 예시입니다. 여러분은 추상화 레이어를 만들 수 있고, 패키지로 만들어 코드를 공유하고 배포할 수 있습니다. 무엇보다도, 여러분은 Pkl을 다양한 설정 요구사항들에 따라 사용할 수 있습니다. Pkl은 정적 설정 파일을 어떤 포맷으로도 만들 수 있게 해주며, 또는 다른 애플리케이션의 런타임에 내장되게끔 할 수도 있습니다.

우리는 Pkl을 다음 중요한 세가지 목표를 가지고 설계했습니다.

- 배포전 검증 에러를 잡아내 안전성을 제공하기 위해

- 단순한 사용 사례에서 복잡한 사용 사례로 확장하기 위해

- IDE 기능의 서포트를 최대화 하여 작성하는 즐거움을 제공하기 위해

# Pkl을 간단하게 둘러보기 👀

우리는 Pkl을 개발자에게 친숙하고 배우기 쉬운 형태로 제작했습니다. 그래서 우리는 Pkl에 클래스, 함수, 반복문, 타입 어노테이션 같은 기능을 넣었습니다.

예시를 들어봅시다. 아래 예시는 가상의 웹 애플리케이션을 위해 설정 스키마를 Pkl로 작성한 파일입니다.

> 주의: 이 파일은 타입을 정의하지만, 데이터를 포함하지는 않습니다. 이러한 형태는 Pkl의 일반적인 패턴이며, 우리는 이것을 template 이라고 부르기로 했습니다.

_Application.pkl_

```
module Application

/// hostname은 서버가 응답하는 주소를 나타냅니다.
hostname: String

port: UInt16

/// environment는 배포할 환경을 나타냅니다.
environment: Environment

/// database는 애플리케이션에 연결될 데이터베이스를 나타냅니다.
database: Database

class Database {
  /// username은 데이터베이스의 유저 이름을 나타냅니다.
  username: String

  /// 데이터베이스를 위한 패스워드 입니다.
  password: String

  /// 데이터베이스의 원격 host 입니다.
  host: String

  /// 데이터베이스의 원격 port 입니다.
  port: UInt16

  /// 데이터베이스의 이름입니다.
  dbName: String
}

typealias Environment = "dev"|"qa"|"prod"
```

그리고 아래 파일은 설정이 정의되는 상황을 나타냅니다.

_localhost.pkl_

```
amends "Application.pkl"

hostname = "localhost"

port = 3599

environment = "dev"

database {
  host = "localhost"
  port = 5786
  username = "admin"
  password = read("env:DATABASE_PASSWORD")
  dbName = "myapp"
}
```

여기에서 수정을 가해서 같은 데이터에 기반하는 여러 변형들을 만들 수 있습니다. 예로 들어, 우리가 4개의 데이터베이스를 예방책으로 로컬에 돌리길 원한다고 가정해봅시다. for generator를 통해 4개의 변형을 만들 수 있고, 같은 베이스의 db를 사용하되 다른 포트에 할당하게끔 할 수 있습니다.

```
import "Application.pkl"

hidden db: Application.Database = new {
  host = "localhost"
  username = "admin"
  password = read("env:DATABASE_PASSWORD")
  dbName = "myapp"
}

sidecars {
  for (offset in List(0, 1, 2, 3)) {
    (db) {
      port = 6000 + offset
    }
  }
}
```

Pkl은 JSON, YAML, XML 등과 같은 일반적인 포맷으로 생성되게끔 할 수도 있습니다. 아래 예시는 Pkl 에서 YAML로 생성되는 과정을 보여줍니다.

```
$ export DATABASE_PASSWORD=hunter2
$ pkl eval --format yaml sidecars.pkl
```

```yaml
sidecars:
  - username: admin
    password: hunter2
    host: localhost
    port: 6000
    dbName: myapp
  - username: admin
    password: hunter2
    host: localhost
    port: 6001
    dbName: myapp
  - username: admin
    password: hunter2
    host: localhost
    port: 6002
    dbName: myapp
  - username: admin
    password: hunter2
    host: localhost
    port: 6003
    dbName: myapp
```

# 기본 제공 유효성 검사 (Built-in validation) 🧐

설정은 데이터에 관한 것입니다. 그리고 데이터는 유효해야만 합니다.

Pkl에서, 검증은 타입 어노테이션으로 쉽게 달성이 가능합니다. 그리고 타입 어노테이션으로 추가적으로 제약 조건을 걸어둘 수도 있습니다.

아래와 같은 제약 조건을 두는 예시를 살펴봅시다.

- `age`는 0에서 130 사이여야 합니다.
- `name`은 비어선 안됩니다.
- `zipCode`는 5개의 철자를 가지는 문자열이여야 합니다.

_Person.pkl_

```
module Person

name: String(!isEmpty)

age: Int(isBetween(0, 130))

zipCode: String(matches(Regex("\\d{5}")))
```

제약 조건 검증이 실패하면 평가 에러 (Evaluation error) 를 발생시킵니다.

_alessandra.pkl_

```
amends "Person.pkl"

name = "Alessandra"

age = -5

zipCode = "90210"
```

이 모듈은 다음과 같이 실패하게 됩니다 :

```
$ pkl eval alessandra.pkl
–– Pkl Error ––
Type constraint `isBetween(0, 130)` violated.
Value: -5

5 | age: Int(isBetween(0, 130))
             ^^^^^^^^^^^^^^^^^
at Person#age (file:///Person.pkl)

5 | age = -5
          ^^
at alessandra#age (file:///alessandra.pkl)

106 | text = renderer.renderDocument(value)
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
at pkl.base#Module.output.text (https://github.com/apple/pkl/blob/0.25.0/stdlib/base.pkl#L106)
```

제약 조건은 임의의 표현식입니다. 이를 통해 Pkl로 표현될 수 있는 모든 유형의 검증을 체크할 수 있습니다. 아래 예시는 홀수 길이의 문자열과, 첫번째 문자와 마지막 문자가 같아야 하는 타입을 정의하고 있습니다.

```
name: String(length.isOdd, chars.first == chars.last)
```

# 패키지를 공유하기 📦

Pkl을 패키지를 배포하고, 프로젝트의 의존성으로서 불러올 수 있는 기능을 제공하고 있습니다. 이러한 기능은 다른 프로젝트에서도 Pkl 코드를 사용할 수 있는 쉬운 방법을 제시합니다.

여러분만의 패키지를 만들어 Github이나 업로드 하기를 원하는 어느 공간에서든 배포 할 수 있습니다.

패키지는 절대경로 URI를 통해 불러 올 수 있습니다.

```
import "package://pkg.pkl-lang.org/pkl-pantry/pkl.toml@1.0.0#/toml.pkl"

output {
  renderer = new toml.Renderer {}
}
```

또는, 패키지는 프로젝트 종속적으로 관리 될 수도 있습니다. 프로젝트를 사용하여 종속성 그래프 내에서 동일한 종속성의 서로 다른 버전간의 충돌을 해결 할 수 있습니다. 이것은 또한 패키지를 보다 더 간단한 이름으로 불러올 수 있다는 것을 의미합니다.

_PklProject_

```
amends "pkl:Project"

dependencies {
  ["toml"] { uri = "package://pkg.pkl-lang.org/pkl-pantry/pkl.toml@1.0.0" }
}
```

_myconfig.pkl_

```
import "@toml/toml.pkl"

output {
  renderer = new toml.Renderer {}
}
```

패키지 세트는 Pkl 팀에서 관리되고 있습니다. Pkl 팀이 제공하는 패키지는 다음과 같습니다.

- [pkl-pantry](https://github.com/apple/pkl-pantry) — 다양한 패키지를 배포 할 수 있는 모노레포
- [pkl-k8s](https://github.com/apple/pkl-k8s) — Kubernetes descriptor 를 정의하기 위한 템플릿

# 언어 서포트

Pkl은 텍스트 결과물로 생성 할 수도 있고, Pkl에서 공식적으로 제공하는 언어 서포트를 통해 다른 언어로도 생성 할 수 있습니다.

Pkl이 언어로 통합될때, Pkl의 스키마는 타겟 언어의 클래스/struct로 생성이 됩니다. 아래 예시는 위에서 살펴보았던 _Application.pkl_ 이 Swift, Go, Java, Kotlin으로 생성되는 것을 보여줍니다. Pkl은 또한 타겟 언어의 문서 주석까지 포함하여 생성됩니다. _(역주: 해당 번역에서는 스크롤 관계상 Go 언어만 보여드리도록 하겠습니다.)_

_Application.pkl.go_

```go
package application

type Application struct {
	// The hostname that this server responds to.
	Hostname string `pkl:"hostname"`

	// The port to listen on.
	Port uint16 `pkl:"port"`

	// The environment to deploy to.
	Environment environment.Environment `pkl:"environment"`

	// The database connection for this application
	Database *Database `pkl:"database"`
}
```

_Database.pkl.go_

```go
// Code generated from Pkl module `Application`. DO NOT EDIT.
package application

type Database struct {
	// The username for this database.
	Username string `pkl:"username"`

	// The password for this database.
	Password string `pkl:"password"`

	// The remote host for this database.
	Host string `pkl:"host"`

	// The remote port for this database.
	Port uint16 `pkl:"port"`

	// The name of the database.
	DbName string `pkl:"dbName"`
}
```

_environment/Environment.go_

```go
// Code generated from Pkl module `Application`. DO NOT EDIT.
package Environment

import (
	"encoding"
	"fmt"
)

type Environment string

const (
	Dev  Environment = "dev"
	Qa   Environment = "qa"
	Prod Environment = "prod"
)

// String returns the string representation of Environment
func (rcv Environment) String() string {
	return string(rcv)
}
```

코드 생성을 이용하는 것은 Pkl을 애플리케이션에 통합하는 수많은 방법중 하나입니다. Pkl은 저수준 (Low-level)에서 Pkl의 평가 기능을 제어하는 Evaluator API를 제공하며, 사용자는 애플리케이션에 가장 적합한 추상화 레벨에서 Pkl과 상호작용을 할 수 있습니다.

# 역주의 Pkl에 대한 생각

Pkl은 최근에 나온 기술들 중 가장 흥미로운 기술이 아닐까 싶습니다. 애플에서 만든 오픈소스라는 점도 그렇고, 애플리케이션의 Configuration 에서 추상화 레이어를 한단계 더했다는 점도 흥미로웠습니다. 하지만 Pkl 공식문서와 블로그를 읽어보며 아직까지는 Pkl의 철학이나 존재 의의에 대해서 완전히 공감하기는 힘들다는 생각이 들었습니다.

첫번째, 애플리케이션의 Configuration을 위해 한단계 더 컴파일 하는 단계가 추가됩니다. 그리고 아직까지 Pkl 측에서 내세우고 있는 데이터 검증이나 패키지 기능을 위해 애플리케이션에서 이러한 컴파일 단계를 추가하는 것이 과연 옳을지에는 조금 의문이 듭니다. 자료를 읽어보면서 Pkl은 사이드 프로젝트, 혹은 소규모 프로젝트 보다는 보다 데이터 검증을 많이 거쳐야 하는 대규모 프로젝트에 올바르다는 생각이 들었습니다. Pkl이 제공하는 수 많은 기능들이 모든 상황에 필요하지는 않다라는 생각이 들었고, Pkl을 도입할때는 신중하게 생각해보는 것이 좋겠다는 생각이 들었습니다.

두번째, 추상화는 양날의 검입니다. Pkl은 추상화 레이어를 한단계 더해 Configuration에 더 많은 기능을 제공하지만, 이 뜻은 애플리케이션에서 예기치 못한 에러를 발생 시킬 수도 있다는 것을 의미합니다. 우리가 통제하기 어려운 추상화 레이어를 더하여 그러한 리스크를 지면서 Pkl이 제공하는 기능들을 가져가야 할까요? 생각해봐야 할 문제입니다.

그럼에도 불구하고 Pkl 자료를 번역하면서 흥미로운 부분들이 많다고 생각했고, 더 많은 사람들에게 소개해보면 좋을 것 같아서 Pkl의 공식 소개글을 번역하게 되었습니다. 애플리케이션의 Configuration과 그 과정에서 추상화 레이어를 추가하는 것에 대해서 생각해볼 수 있는 좋은 시간이였던 것 같습니다.
