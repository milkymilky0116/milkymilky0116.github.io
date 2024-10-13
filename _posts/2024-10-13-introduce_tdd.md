---
title: TDD와 미니멀리즘에 관한 이야기 -1-
date: 2024-10-11 22:04:43 +/-TTTT
categories: [TDD]
tags: [TDD]
mermaid: true # TAG names should always be lowercase
---

# 서론

웹 개발자라면 한번쯤은 TDD(Test-Driven Development) 방법론에 대한 이야기를 한번 쯤은 들어보았을 것이다. 하지만 그 방법론에 대한 명성과는 달리 TDD는 아주 낯설고 어렵게만 느껴진다. 그리고 TDD가 정말로 효용성있는 방법론인지 아닌지에 대해 지금도 넷상 어디에서 끊임없이 논의되고 있다. TDD란 무엇이며, 우리가 정말 알아두어야할 가치가 있는 방법론일까?

최근 TDD 스터디를 진행하면서 스터디원들이 이 방법론에 어떻게 접근해야 할지 모르겠다는 피드백을 많이 주었다. 프로그래밍 언어, 프레임워크, 혹은 어떤 개념을 공부하는 것과는 다르게 TDD에는 명확하게 정해진 정답이 없기 때문에 낯설어 하는 것이 어떻게 보면 당연할 것이다. 그래서 이 글은 지난 4개월간 Rust를 TDD로 공부하면서 미약하게나마 얻은 인사이트를 공유하고자 작성했다. TDD를 한번쯤이라도 여행하고자 하는 히치하이커들에게 조금이나마 도움이 되었으면 하는 바람으로 글의 서두를 작성하였다. 물론 최대한 언어, 프레임워크에 종속되지 않게 글을 쓰는 것이 목표이기 때문에 다양한 언어의 예제코드를 제시하며 글을 이어 나갈 것이다.

	 ⚠️ 이 글에서 설명하는 TDD 방법론은 백엔드 개발에 중점을 두고 작성하였다. 프론트엔드, 혹은 다른 분야에서는 통용되지 않는 이야기일 수도 있다.

# Ch 1. 우리가 놓치고 살아온 것들

여러분이 어떤 서비스의 서버 개발을 맡게 되었다고 가정해보자. 여러분이 건네 받은 요구사항 명세서의 가장 위쪽에는 `/health_check` 엔드포인트를 먼저 만들어 달라고 되어있다. 해당 엔드포인트로 요청을 보내면 HTTP 응답 코드는 `200 OK`를 반환해야 한다. 식은죽 먹기이다. 그렇지 않은가? 그렇다면 이것을 TDD로 어떻게 풀어낼 수 있을지 생각해보자. Spring 개발자들도 잠시 프레임워크에서 제공해주는 기능을 떼어놓고 생각해보자.

간단한 태스크 같다. 일단 호기롭게 테스트 코드를 먼저 작성해보기로 한다. 심플하게 생각해보면 다음과 같은 테스트 코드를 작성해야 할 것이다.

	테스트 코드 내부에서 웹 서버를 띄운다. 그리고 `/health_check` 엔드포인트로 요청을 보내어 200 응답이 오는지 확인한다.

이렇게만 보면 간단하다. 그래서 여러분은 일단 아는 지식으로 웹서버를 띄우는 함수를 작성한다. 일부러 최대한 미니멀하게 보여주고자 Go로 작성해보았다. Go를 잘 모르더라도 어떤 흐름인지는 이해가 가능 할 것이다.

먼저 테스트 코드를 작성하자.

```go
package main

import (
    "net/http"
    "net/http/httptest"
    "testing"
)

func TestHealthCheckHandler(t *testing.T) {
	run()
	// 띄워져있는 8080 포트로 요청을 보냅니다.
    req, err := http.NewRequest("GET", "http://localhost:8080/health_check", nil)
    if err != nil {
        t.Fatal(err)
    }

	// StatusCode를 검증합니다.
    if status := req.Response.StatusCode; status != http.StatusOK {
        t.Errorf("응답 상태 코드가 잘못되었습니다: 받음 %v, 예상 %v", status, http.StatusOK)
    }
}

```


그리고 구현 코드를 작성한다.

```go
package main

import (
    "fmt"
    "net/http"
)

func healthCheckHandler(w http.ResponseWriter, r *http.Request) {
	// 200 OK 코드로 응답을 보냅니다.
    w.WriteHeader(http.StatusOK)
}

func run() {
	// Handler (Spring이나 기타 프레임워크에서는 Controller)를 등록합니다.
	http.HandleFunc("/health_check", healthCheckHandler) 
	fmt.Println("서버가 포트 8080에서 실행 중입니다.") 

	// 8080 포트로 서버를 실행합니다.
	http.ListenAndServe(":8080", nil)
}

func main() {
	run()
}
```

여러분은 의기양양하게 테스트 코드를 돌려본다. 하지만.. 테스트 코드가 멈추지 않고 계속해서 실행된다. 이내 뭔가 잘못 되었음을 깨닫는다. 어떻게 하면 고칠 수 있을까?

TDD의 본질은 "가장 간단하게 문제를 해결 하는 것" 부터 시작한다. 여러가지 해결책이 있을 수 있겠지만, 웹서버를 백그라운드 스레드로 보내는 것이 가장 심플하면서 효과적인 해결 방법일 것이다. go에는 goroutine 이라는 강력한 경량 스레드 기능이 있기 때문에 이를 쉽게 구현 할 수 있다.

```go
package main

import (
	"net/http"
	"testing"
	"time"
)

func TestHealthCheckHandler(t *testing.T) {
	// 서버를 실행합니다.
	go func() {
		run()
	}()

	// 서버가 시작될 때까지 잠시 대기합니다.
	time.Sleep(1 * time.Second)

	// 요청을 보냅니다.
	resp, err := http.Get("http://localhost:8080/health_check")
	if err != nil {
		t.Fatal(err)
	}
	defer resp.Body.Close()

	// StatusCode를 검증합니다.
	if status := resp.StatusCode; status != http.StatusOK {
		t.Errorf("응답 상태 코드가 잘못되었습니다: 받음 %v, 예상 %v", status, http.StatusOK)
	}
}

```

Java로 비슷하게 생각해본다면 조금 더 복잡해진다.

```java
// ...

class DemoApplicationTests {

    @Test
    void testHealthCheck() throws Exception {
        // 서버를 별도의 스레드에서 실행하고 ApplicationContext를 가져옵니다.
        ExecutorService executor = Executors.newSingleThreadExecutor();
        Future<ConfigurableApplicationContext> future = executor.submit(() -> {
            return new SpringApplicationBuilder(DemoApplication.class)
                    .web(WebApplicationType.SERVLET)
                    .run();
        });

        // 서버가 시작될 때까지 대기합니다.
        ConfigurableApplicationContext context = future.get(10, TimeUnit.SECONDS);

        try {
            // HTTP GET 요청을 보냅니다.
            URL url = new URL("http://localhost:8080/health_check");
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            conn.setRequestMethod("GET");

            int statusCode = conn.getResponseCode();
            assertEquals(200, statusCode);

        } finally {
            // 테스트가 끝나면 서버를 종료합니다.
            context.close();
            executor.shutdownNow();
        }
    }
}

```

심히 복잡한 코드이다. 하지만 이러한 코드가 여러분이 사용하는 프레임워크가 몰래 뒤에서 해주던 일이다. 테스트 코드를 실행 할 때 웹서버를 백그라운드 스레드, 혹은 프로세스로 실행시킨다.

우리는 고작 `/health_check` 엔드포인트를 TDD로 작성하고자 한 것 뿐인데 많은 것을 얻었다. 테스트코드를 실행할 때 프레임워크가 뒤에서 어떤 일을 처리해주는지 알게 되었다. 어떻게 함수 실행을 백그라운드 스레드로 보내는지 알게되었다.

TDD는 위의 예제에서도 보았듯 결코 생산성을 높이는 방법론은 아니다. 여러분은 가장 작은 문제부터 테스트 코드를 작성하고, 개선하고, 다시 테스트 케이스를 늘려 또 다른 문제를 해결해야 하는 복잡한 과정을 거쳐야 한다. 하지만 TDD는 여러분에게 생각해볼 여지를 제공한다. 가령 위의 코드에서는 미처 커버하지 않았지만, 테스트 코드가 늘어날 수록 띄워야 하는 가상 웹서버도 많아질텐데 포트 충돌의 문제를 어떻게 해결 할 것인가? 테스트 코드가 종료될때 서버 스레드를 우아하게 종료시킬 수 있을까? 이러한 문제는 얼핏 보기에는 어려워 보인다. 하지만 최대한 심플한 방법을 생각해보고, 그 방법에 대해 문제점을 다시 거치는 과정은 여러분과 여러분이 제작하는 소프트웨어를 견고하게 만들 것이다.

TDD의 미학은 그래서 "미니멀리즘" 이라고 생각한다. 가장 간단한 문제에서 시작해서 가장 간단한 해결책을 생각하고 다시 그 방법속에서 문제를 발견하는 것이다. 설령 여러분이 이미 프레임워크에 친숙해져 프레임워크가 제공하는 추상화에 익숙해져 있더라도 이 과정은 의미있다. 프레임워크가 여러분들을 대신해 처리해주던 일들에 대해 생각해봄으로써 더 견고한 애플리케이션을 작성 할 수 있다.

다음 편에서는 DB 연동에 대해서도 생각해볼 것인데, 아래와 같은 사항들을 고려하면서 어떻게 좋은 테스트 코드를 작성할 수 있을지 생각해보자.

- 각 테스트 코드는 독립적이여야 한다. 사이드 이펙트를 최소화 하거나 없애야 한다.
- 테스트 코드를 작성하면서 가능한 모든 Edge-case를 생각해야 한다.
- 테스트 코드가 항상 우선시 되어야 한다. 리팩토링을 어느 시점에서 하면 좋을지 생각해보자.
