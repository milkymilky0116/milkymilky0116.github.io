---
title: 서버에서 Graceful Shutdown은 왜 필요할까
date: 2024-11-03 01:04:43 +/-TTTT
categories: [Golang, Backend]
tags: [Golang, Backend]
mermaid: true # TAG names should always be lowercase
---
# 서론

Golang으로 작성된 여러 백엔드 서적을 읽으면서, "Graceful Shutdown (직역하면 우아하게 종료)" 라는 개념이 많이 등장하는 것을 보았습니다. 하지만 여러 책들을 읽다보니 서버를 우아하게 종료하는 것이 왜 필요한지, 어떤 이점을 가져다 주는지에 대해서는 다소 정보가 부족한것 같았습니다. "우아하게 종료하는" 매커니즘이 어떤 장점을 가져다줄까요? 그래서 오늘 글에서는 Graceful Shutdown을 구현한 서버와 그렇지 않은 서버를 테스트 해보면서 비교해보는 시간을 가져보겠습니다.

# 테스트 계획

테스트 계획은 간단합니다. 두가지 방식으로 나누어서 서버를 구동 해볼건데요, 앞서 설명드린 것처럼 Graceful 하게 종료되는 로직과 그렇지 않은 로직 두가지를 구현 해볼겁니다. 서버는 공통적으로 `/long` 이라는 3초간의 대기시간을 가지고 있는 핸들러를 가지고 있습니다.

먼저 Graceful 하지 않은 서버를 구현해보면서 "우아하게 종료하는 것"이 무엇인지 살펴봅시다.

Go에서 웹 서버를 생성하고 띄우는 것은 간단합니다. `http.Server`객체를 생성하고 `ListenAndServe` 메소드를 호출하기만 하면 되지요. 해당 구현에서는 테스트를 위해서 `ListenAndServe` 메소드를 goroutine으로 보냈습니다. 서버 goroutine이 테스트를 방해하면 안되니까요.

```go
// cmd/server/server.go

const TASK_COMPLETE = "Task Complete"

// 요청 핸들러입니다. 핸들러는 3초의 대기시간을 가지고 있습니다.
func longHandler(w http.ResponseWriter, r *http.Request) {
	time.Sleep(3 * time.Second)
	fmt.Fprintf(w, TASK_COMPLETE)
}

func routes() *http.ServeMux {
	mux := http.NewServeMux()
	mux.HandleFunc("GET /long", longHandler)
	return mux
}

func NewServer() *http.Server {
	srv := http.Server{
		Addr:    ":8080",
		Handler: routes(),
	}
	return &srv
}

func RunNotGracefulServer() *http.Server {
	srv := NewServer()
	go func() {
		// 서버를 실행합니다
		if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
			log.Fatalf("Server execution error: %v", err)
		}
	}()

	return srv
}
```

이제 테스트를 작성해보겠습니다. 테스트도 비교적 간단합니다. 띄워진 웹서버에 http 요청을 보냅니다. 이때 요청 도중 서버를 종료해보면 어떤 일이 일어날까요? 요청에는 3초가 걸리는데, 1초 대기한 뒤 서버를 종료시키도록 해보았습니다.

```go
// cmd/server/server_test.go

func TestNotGracefulShutdown(t *testing.T) {
	srv := RunNotGracefulServer()

	time.Sleep(500 * time.Millisecond)
	client := http.Client{
		Timeout: 5 * time.Second,
	}

	var wg sync.WaitGroup
	wg.Add(1)
	go func() {
		defer wg.Done()
		req, err := http.NewRequest(http.MethodGet, "http://localhost:8080/long", nil)
		if err != nil {
			t.Errorf("Fail to create http request : %v", err)
			return
		}
		resp, err := client.Do(req)
		if err != nil {
			t.Errorf("Expected request to success, but it fail")
			return
		}
		defer resp.Body.Close()
	}()
	time.Sleep(1 * time.Second)

	if err := srv.Close(); err != nil {
		t.Errorf("Server closing error : %v", err)
	}
	wg.Wait()
}
```

실패 메세지가 출력되는 것은 당연한 결과입니다. `http.Client`가 요청을 처리하는 부분에서 에러 로그가 출력이 되네요.

<img width="830" alt="스크린샷 2024-10-31 16 36 12" src="https://github.com/user-attachments/assets/e2fc4ce5-2e2c-4321-bffa-f74d6eabc253">

어떻게 보면 당연한 결과입니다. 요청을 보내던 도중 서버가 종료 되었으니 당연히 요청도 실패할 수 밖에 없습니다. 하지만 다시한번 곰곰히 생각해봅시다. 만일 프로덕션으로 올라가있는 서버라면 어떤 상황이 벌어질까요? 어떤 이유에서 서버에서 예기치 못한 에러가 발생해 패닉이 발생하거나, 혹은 예기치 못하게 서버를 잠시 종료해야하는 상황이 발생 할 수도 있습니다. 이때 여러분의 사용자들은 서비스를 이용하다가 요청이 마무리 되지 않고 빈 페이지를 보게 된다거나, 혹은 결제 시스템에서 오류가 발생해 결제는 됐는데 서비스를 이용하지 못하는 상황이 발생 할 수도 있습니다. Transaction이 중요한 시스템에서는 심각한 사고가 발생할 수도 있는 문제입니다. 

Graceful Shutdown은 바로 이러한 점에서 필요합니다. 서버가 종료되기 이전에 현재 들어온 요청들에 대해서는 모두 마무리하고 끝내자! 라는 개념이 바로 Graceful Shutdown 입니다. 우리의 서버는 사용자 경험을 보호하고 데이터를 안전하게 처리해야 할 의무가 있습니다. 그래서 예상치 못한 에러나 사고에도 대비가 되어 있어야 합니다. **갈땐 가더라도 우리의 서버는 일을 끝마치고 가야합니다.**

그렇다면 Graceful Shutdown은 어떻게 구현 할 수 있을까요? 음, 우선 정말 간단하게 로직을 생각해보았습니다.

<img width="729" alt="스크린샷 2024-10-31 16 58 18" src="https://github.com/user-attachments/assets/30de529d-16f6-47c2-93bb-db7d8865becd">

어떤 종료 메세지 (ctrl+c로 종료, 에러 등..)를 받으면 서버는 현재 요청을 중단하고, 남아있는 요청을 모두 처리한 뒤 서버를 종료합니다. 이를 어떻게 구현 할 수 있을까요?

우선 가장 마지막에 "요청을 중단하고, 남아있는 요청을 모두 처리 한 뒤 서버 종료" 하는 부분은 간단합니다. 이미 Go의 `net/http` 라이브러리에는 이러한 메소드가 구현이 되어 있습니다. 그래서 아래와 같이 구현하기만 하면 됩니다.

```go
// ...
if err := server.Shutdown(context.Background()); err != nil {
	log.Fatalf("Fail to shutdown server: %v", err)
}
```

여기서 잠깐, `Shutdown` 메소드는 `context` 라고 하는 패키지에서 어떤 파라미터를 받고 있습니다. `context` 란 무엇일까요? Go의 공식 문서에서 설명하는 것을 살펴봅시다.

	 Go 서버에서 각 들어오는 요청은 자체 goroutine에서 처리됩니다. 요청 처리기는 종종 데이터베이스나 RPC 서비스와 같은 백엔드에 접근하기 위해 추가적인 goroutine을 시작합니다. 특정 요청에 대해 작업하는 goroutine 집합은 일반적으로 최종 사용자 식별, 인증 토큰, 요청 기한과 같은 요청에 특화된 값에 접근해야 합니다. 요청이 취소되거나 시간이 초과되면, 해당 요청을 처리하는 모든 goroutine은 빠르게 종료되어 시스템이 사용 중인 리소스를 회수할 수 있어야 합니다.
	 구글에서는 요청 범위 값, 취소 신호, 기한을 API 경계를 넘어 요청을 처리하는 모든 goroutine에 쉽게 전달할 수 있도록 해주는 context 패키지를 개발했습니다. 이 패키지는 context라는 이름으로 공개되어 있습니다.

설명을 살펴보니 `context` 패키지는 저희의 목표에 잘 부합하는 것 같습니다. `context` 패키지는 각 goroutine 간의 자원을 효율적으로 공유하기 위해 설계된 패키지입니다. 예를 들어 특정 `context` 를 공유하는 어떤 작업들이 있을때, `context` 를 취소해서 하위 작업들 또한 종료되게 만들 수 있습니다. 이 말이 어렵다면 아래 예시를 통해 살펴봅시다.

```go
func main() {
	ctx, cancel := context.WithCancel(context.Background())
	go Task(ctx, "worker 1")

	go Task(ctx, "worker 2")

	go Task(ctx, "worker 3")

	time.Sleep(time.Second * 3)
	cancel()

	time.Sleep(time.Second * 1)
}

func Task(ctx context.Context, taskName string) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println("Task is canceled..")
			return
		default:
			fmt.Printf("%s is running..\n", taskName)
			time.Sleep(time.Second * 1)
		}
	}
}
```

위의 코드를 실행 시키면 아래와 같은 결과가 나옵니다.

```bash
worker 1 is running..
worker 2 is running..
worker 3 is running..
worker 2 is running..
worker 1 is running..
worker 3 is running..
worker 3 is running..
worker 2 is running..
worker 1 is running..
Task is canceled..
Task is canceled..
Task is canceled..
```

코드를 간단하게 설명 해볼게요. 우선 `main` 함수의 첫 줄에서는 `context` 객체를 생성하고 있습니다. 해당 예제에서는 `WithCancel` 이라는 메소드를 통해 생성 했는데요, 필요에 따라 다양한 옵션으로 `context`를 생성 할 수 있습니다. 예를 들면 Timeout 될때 `context` 가 취소 되게 할 수도 있고, 부모 `context` 의 데드라인에 맞추어 취소되게 할 수도 있습니다. 해당 예제에서는 `WithCancel` 메소드를 사용해서 저희가 직접 `context` 를 취소할 수 있도록 했습니다.

```go
ctx, cancel := context.WithCancel(context.Background())
// ...
time.Sleep(time.Second * 3)
cancel()
```

`Task` 메소드는 `context` 를 직접 파라미터로 받습니다. 그래서 `select` 문으로 `context` 가 취소되었는지 아닌지를 판단하고 있습니다. `context` 의 취소 신호는 `channel` 을 통해 수신되므로 `<-ctx.Done()` 을 통해 취소 신호를 받습니다. 취소 되었을 경우에는 즉시 함수를 종료 시킵니다.

해당 예제 코드를 전부 이해 못하셨어도 괜찮습니다. 중요한 것은 go의 `net/http` 패키지에서 `Shutdown` 메소드도 위의 코드와 비슷한 작동 방식으로 goroutine 들을 관리하고 있다는 것입니다. 

```go
func gracefulShutdown(server *http.Server, done chan bool, shutdownChan <-chan struct{}) {
	<-shutdownChan
	ctx, cancel := context.WithTimeout(context.Background(), time.Second*5)
	defer cancel()
	if err := server.Shutdown(ctx); err != nil {
		log.Fatalf("Fail to shutdown server: %v", err)
	}
	done <- true
}
```

저는 이런 방식으로 우아하게 종료하는 것을 구현 했습니다. `WithTimeout` 메소드를 이용해서 goroutine 들에게 취소 신호를 보냅니다. 만일 요청이 5초 이상의 처리 시간이 걸린다면 에러가 발생하게 됩니다. 

`shutdownChan`과 `done` 채널은 각각 goroutine 외부에서 취소 신호와 작업 완료 신호를 전달하는 역할을 합니다. 다만, 여기서는 테스트를 위해 간략하게 구현했습니다. 실제 서버 구현 시에는 `SIGTERM`, `SIGINT`와 같은 시그널을 받아 취소 요청을 처리하는 것이 좋습니다.

```go
ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
defer stop()

<-ctx.Done()
```

`gracefulShutdown` 메소드는 서버가 실행 될 때 별도의 goroutine에서 취소 신호를 대기합니다.

```go
func RunGracefulServer(done chan bool, shutdownChan <-chan struct{}) (*http.Server, error) {
	srv := NewServer()
	go func() {
		if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
			log.Fatalf("Server execution error: %v", err)
		}
	}()
	go gracefulShutdown(srv, done, shutdownChan)
	return srv, nil
}
```

이제 테스트 코드를 살펴봅시다. 테스트 코드는 이전과 크게 다르지 않습니다. 앞서 설명 드렸듯 취소 신호와 작업 완료 신호를 받는 채널을 파라미터로 주어야 하기 때문에 그와 관련된 코드가 몇줄 추가된 정도입니다.

```go
func TestGracefulShutdown(t *testing.T) {
	shutdownChan := make(chan struct{})
	done := make(chan bool, 1)
	_, err := RunGracefulServer(done, shutdownChan)
	if err != nil {
		t.Errorf("Fail to launch server : %v", err)
	}

	time.Sleep(500 * time.Millisecond)
	client := http.Client{
		Timeout: 5 * time.Second,
	}

	var wg sync.WaitGroup
	wg.Add(1)
	go func() {
		defer wg.Done()
		req, err := http.NewRequest(http.MethodGet, "http://localhost:8080/long", nil)
		if err != nil {
			t.Errorf("Fail to create http request : %v", err)
			return
		}
		resp, err := client.Do(req)
		if err != nil {
			t.Errorf("Fail to call http request : %v", err)
			return
		}
		defer resp.Body.Close()
		body, err := io.ReadAll(resp.Body)
		if err != nil {
			t.Errorf("Fail to read http body : %v", err)
			return
		}

		if string(body) != TASK_COMPLETE {
			t.Errorf("Expect : %s, but got %s", TASK_COMPLETE, string(body))
			return
		}
	}()

	time.Sleep(1 * time.Second)
	close(shutdownChan)
	wg.Wait()
	<-done
}
```

테스트가 통과 합니다 🎉

<img width="1704" alt="스크린샷 2024-11-03 00 34 06" src="https://github.com/user-attachments/assets/1b7f12d2-8d7b-4b33-b38a-dfbac6446c87">

# 다른 언어 / 프레임워크에서는 어떨까

이 글을 읽는 대다수의 독자분들은 아마 다른 언어, 혹은 프레임워크의 개발자일 것입니다. 다른 프레임워크에도 "우아하게 종료" 하는 개념이 있을까요? 물론입니다. 위에서 구현했던 비슷한 작업의 일들을 여러분 몰래 처리해주고 있습니다.

Java Spring 같은 경우에는 `application.properties` 파일에서 몇가지 설정을 추가하면 됩니다.

```properties
server.shutdown=graceful 
spring.lifecycle.timeout-per-shutdown-phase=30s
```

Spring에서는 안전한 자원 해제를 위해 `@PreDestroy` 어노테이션과 `DisposableBean` 인터페이스를 제공합니다. `@PreDestroy` 어노테이션은 Spring Bean이 제거되기 직전에 실행됩니다. `DisposableBean` 인터페이스는 `destroy()` 메소드를 구현하도록 요구해 애플리케이션 종료 시 자원을 정리하도록 할 수 있습니다.

```java
import javax.annotation.PreDestroy;
import org.springframework.stereotype.Component;

@Component
public class ResourceCleaner {

    @PreDestroy
    public void cleanUp() {
        System.out.println("Releasing resources...");
        // 데이터베이스 연결 종료, 파일 저장, 로그 정리 등
    }
}
```

```java
import org.springframework.beans.factory.DisposableBean;
import org.springframework.stereotype.Component;

@Component
public class ResourceManager implements DisposableBean {

    @Override
    public void destroy() {
        System.out.println("Cleaning up resources...");
        // 예: 데이터베이스 연결 해제, 파일 저장, 캐시 비우기
    }
}
```

이외에도 Node.js 에서는 `process.on('SIGTERM', callback)` 으로 취소 요청 신호를 받고 `http.Server.Close()` 함수를 호출해 우아하게 서버를 종료 할 수 있습니다.

Python에서는 프레임워크 차원에서 이를 지원하진 않지만 WSGI 서버에서 처리할 수 있습니다.

# 결론

서버는 사용자에게 안정성있는 경험을 줄 필요가 있습니다. 서버가 예기치 못한 상황에서 종료될때도 서버는 사용자의 요청을 매듭짓고 종료되어야 할 필요성이 있습니다. "Graceful Shutdown" 이라는 개념은 바로 여기에서 출발했습니다. 특히 마이크로 서비스 단위로 서버가 분리되고 인스턴스가 많아지는 현대의 클라우드 환경에서 서버를 우아하게 종료하는 것은 더이상 그냥 넘겨서는 안될 문제가 되었습니다.

Go를 통해 이를 직접 구현해보는 시간을 가지면서 여러분이 사용하는 프레임워크가 뒤에서 어떤 일을 처리해주고 있는지 잠시 엿보았습니다. 

[예제 소스코드 레포지토리](https://milkymilky0116.github.io/posts/Graceful_Shutdown/)
