---
title: HTMX를 사용하며 느낀 몇가지 생각들
date: 2024-01-11 01:53:43 +/-TTTT
categories: [htmx, essay]
tags: [htmx]
mermaid: true # TAG names should always be lowercase
---

React를 위시한 프론트엔드 생태계와 커뮤니티가 나날이 커지고 있는 시대의 흐름속에서, 홀로 시대를 역행하는 듯한 라이브러리를 발견했다. [HTMX](https://htmx.org)라고 불리는 이 라이브러리의 간단한 소개글들과 공식 문서를 읽어보면서, 꽤 매력적인 라이브러리 라는 생각이 들었다. 자바스크립트를 크게 곁들이지 않고도, React를 사용하면서 겪었던 수많은 종속성의 늪을 겪지 않고도 웹 애플리케이션을 만들 수 있다니. 당장 안써볼 수 없겠다는 생각이 들어 지난 몇일간 HTMX와 함께 간단하게 프로젝트를 만들었다. 이 글은 HTMX를 사용하면서 느꼈던 몇가지 충격 포인트에 대해서 이야기 해보려고 한다.

![](https://github.com/milkymilky0116/obsidian-backup/blob/main/image/HTMX/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202024-01-09%20%EC%98%A4%ED%9B%84%208.30.00.png?raw=true)

### 한마디로 정의하기 어려운 HTMX

HTMX는 겉으로 보기에는 매우 심플한 라이브러리이다. HTML의 태그에 요청을 보낼 주소와 요청을 받은 뒤 행동을 간단한 문법과 함께 명시해두면 많은 코드를 적지 않고도 React로 만든 웹앱과 비슷한 기능을 하는 웹 애플리케이션을 만들 수 있다. 아래와 같이 말이다.

```html
<button hx-post="/clicked" hx-swap="outerHTML">
  <!-- /clicked 로 요청을 보내면 HTML 요소를 swap 하는 행동을 한다.-->
  Click Me
</button>
```

잠깐, 주석을 보고 이상한 점을 느끼지 않았는가. HTML 요소를 swap 한다니?

겉보기에는 평범한 프론트엔드 라이브러리처럼 보이지만, HTMX를 사용하기 위해서는 우리가 줄곧 많이 사용해왔던 데이터 API(JSON 과 같은) 에서 생각을 벗어날 필요성이 있다. HTMX는 HTML 요소를 반환하는 하이퍼미디어 형태의 API와 궁합이 잘 맞기 때문에, HTMX는 궁극적으로 서버의 역할까지 바꿔놓는 역할을 한다. 그래서 HTMX는 한마디로 정의하기 어렵다. 표면상으로는 프론트엔드 라이브러리이지만, 궁극적으로 서버의 역할까지 바꿔놓으니 말이다.

여기서부터 여러가지 의문이 드는 독자분들이 있을것이다.

1. 그렇다면 HTMX를 사용하기 위해서는 기존의 데이터 API를 모조리 바꿔야 하나?
2. 서버에서 HTML을 반환하는 형식이라면, 서버에서도 그 많은 HTML 컴포넌트들을 관리해야 하나?
3. 애초에, 하이퍼미디어 API의 존재 의의가 뭐지?

1번 질문부터 살펴보자. 정답은 HTMX를 사용하려면 어쩔수 없다. 물론 현재 잘 서비스 되고 있는 데이터 API와 클라이언트 앱을 굳이 HTMX로 마이그레이션 하기 위해 모조리 뜯어고쳐서 리스크를 감수하는 회사는 많이 없을 것이다. 하지만, 여기에 그런 리스크를 감수하고 HTMX로 마이그레이션 한 감명깊은 사례가 하나 있다.

https://youtu.be/3GObi93tjZI?si=aounjSrgyuUJ_yr2

![스크린샷 2024-01-10 오후 7.46.51.png](https://github.com/milkymilky0116/obsidian-backup/blob/main/image/HTMX/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202024-01-10%20%EC%98%A4%ED%9B%84%207.46.19.png?raw=true)
![스크린샷 2024-01-10 오후 7.46.19.png](https://github.com/milkymilky0116/obsidian-backup/blob/main/image/HTMX/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202024-01-10%20%EC%98%A4%ED%9B%84%207.46.51.png?raw=true)

React에서 HTMX + 웹 컴포넌트로 마이그레이션 하면서, 메모리 사용량과 빌드 시간, 심지어 코드 베이스까지 획기적으로 줄일 수 있었다. 데모 시연 영상도 꽤 멋있으니 한번쯤 보시는 걸 추천한다.

이 영상의 의의는 React나 React를 위시한 프론트엔드 프레임워크들 만이 정답이 아니라는 것을 보여준 것에 있다고 생각한다. 현대의 클라이언트 웹 애플리케이션들은 많은 종속성을 필요로하고, 라이브러리들의 트렌드가 순식간에 바뀌며, 6개월이나 1년만 지나도 "구식" 애플리케이션이 되기 쉬운 양상을 보이고 있다. 만일 오래 서비스되고 유지보수 하기 쉬운 애플리케이션을 만들고 싶다면? 종속 라이브러리들을 업데이트 하자마자 애플리케이션이 실행이 안되고, 새로 유지보수 해야하는 상황이 싫다면? 어쩌면 그런 상황속에서 React가 "유일한 방법"은 아닐 수 있다는 것이다.

### 상태관리 : 하이퍼미디어 API의 존재 의의

**"상태관리(State Management)"** 라는 단어를 프론트엔드 개발자들은 지겹도록 들었을 것이다. 프론트엔드 생태계에는 Redux, Recoil, Zustand 와 같은 수 많은 상태관리 라이브러리들이 존재하고, 아마 이런 라이브러리들을 공부하느라 진땀 뺀 경험이 프론트엔드 개발자라면 한번쯤은 있을 것이다.

프론트엔드 세상에서는 "상태(State)" 라는 단어를 컴포넌트들 간의 통신 혹은 그 자체의 상태, 혹은 props라는 인자의 넘김으로 보통 이해하고 계실것이라 생각한다. 하지만 좀 더 시야를 넓혀서 서버와 클라이언트를 아우르는 시선으로 바라보자. 그리고 잠깐 오래전으로 시계를 돌려보자. React가 존재하기 이전, jQuery가 이제 막 세상에 등장해서 웹 시장을 뒤흔들어 놓고 있을때로 말이다.

이때의 클라이언트 애플리케이션의 형태는 어땠을까? 예상 가능하겠지만 이때의 클라이언트 애플리케이션은 지금처럼 독자적인 형태가 아니였다.

![server-client(jquery).png](<https://github.com/milkymilky0116/obsidian-backup/blob/main/image/HTMX/server-client(jquery).png?raw=true>)

이때의 클라이언트는 MVC(Model-View-Controller) 혹은 MTV(Model-View-Template) 이라는 구조 아래에서 서버에 비해 작은 부분을 차지하고 있었다. 클라이언트는 지금과 달리 큰 구조를 가지고 있지 않았다. 상태를 가지고 있는 주체는 클라이언트에 비해 큰 구조를 가진 서버였고, 클라이언트는 서버의 상태를 반영해주는 하나의 거울처럼 역할했다. 그리고 그 사이를 우리가 익히 알고있는 jQuery나 템플릿 엔진들이 다리를 놓아주는 역할을 했다. 현대의 웹 애플리케이션 구조를 보기 전에, 다시한번 이 말을 곰곰히 생각해보자.

> _클라이언트는 서버가 가지고 있는 상태의 반영이다._

예를 들어 간단한 To-do 앱을 만든다고 생각해보자. 사용자가 특정한 to-do를 삭제하는 요청을 클라이언트를 통해 보내면, 서버에서는 그 요청을 받아들이고 to-do를 삭제한다. 그 이후 삭제되고 남은 to-do들의 상태를 클라이언트가 반영해야 한다. 클라이언트는 서버에 특정한 상태를 변경하도록 요청하고, 또 그 상태를 반영하는 거울같은 역할을 하는 것이다. 이 사실은 변하지 않고 현대의 웹 애플리케이션으로 넘어와도 통용된다. 클라이언트와 서버간의 역할은 오래전부터 지금까지 변한 것이 없다. 변한 것이 있다면, 클라이언트 애플리케이션이 독자적으로 커지고 서버와 클라이언트 간의 분리가 일어났다는 것이다.

![server-client(now).png](<https://github.com/milkymilky0116/obsidian-backup/blob/main/image/HTMX/server-client(now).png?raw=true>)

위의 사진은 여러분이 익히 알고 계시는 그 구조이다. 서버에 요청을 날리면 JSON 혹은 GraphQL같은 데이터 포맷의 형식으로 클라이언트에게 응답하고, 클라이언트는 그 데이터를 가공해서 사용자에게 보여지는 적절한 형태로 바꾼다. 한마디로 현대의 웹 애플리케이션은 서버와 클라이언트 간의 디커플링이 일어난 것이다. 서버와 클라이언트는 어느 한쪽에 종속된 것 없이 명확하게 분리되어 있다. (물론 최근의 웹 프레임워크들은 서버 영역까지 발을 넓히고 있지만 그 얘기 까지 하면 너무 복잡하니..) 그리고 그 사이를 JSON, GraphQL, tRPC 등등의 형태로 된 중간 다리들이 이어주고 있다.

이렇게 서버와 클라이언트간의 분리가 일어나면서 "상태" 또한 두가지 영역에서 모두 처리해야할 문제가 되었다. 서버야 그렇다 치지만, 클라이언트는 이제 서버에서 받아온 요청을 다시 재가공해서 어떠한 형태로 저장해야할 필요성이 생겼다. 컴포넌트 중심의 웹 개발로 트렌드가 바뀌면서 클라이언트는 각각의 컴포넌트들에게 상태를 전달해야 하는 상황이 되었고, 그것을 여러분이 잘 알고 있는 상태관리 라이브러리들로 해결 한 것이다.

이렇게 분리가 일어나면서 장점과 단점이 모두 공존하게 되었다. 우선 장점부터 이야기 하자면, 모바일 기기의 등장으로 인해 하나의 서비스가 여러개의 클라이언트를 가지게 되는 경우가 많아졌다. 이러한 상황에서 서버와 클라이언트간의 분리는 꽤 합리적인 선택이다. 만일 클라이언트가 서버에 의존하는 고전적인 방식이였다면, 수많은 클라이언트들의 개수만큼 서버를 다시 개발해야하는 상황이 발생 했을지도 모른다.

반면 이러한 분리는 앞에서도 언급했듯 서버와 클라이언트 각각의 상태를 독자적으로 관리해야 한다는 것을 의미하기도 한다. 즉, 이 말은 서버의 상태를 받아오는 클라이언트의 상태 관리가 제대로 이루어지지 않을 경우, 서버와 클라이언트간의 불균형이 생길 수도 있다는 것이다. 클라이언트에서 신경써야 할 부분들이 늘어나고, 파생되는 문제들을 해결하기 위해 점점 더 클라이언트 서비스가 많은 종속 라이브러리들을 필요로 하면서 현재의 프론트엔드 생태계가 형성이 된 것이다.

> 첨언) 그래서 Next.js나 Nuxt.js 같은 "풀스택" 라이브러리들이 서버로 점점 더 영역을 넓혀가는데는 이러한 이유가 있지 않을까 라는 생각이 든다. 점점 더 과거의 PHP를 닮아가는 것 같기도 하고...

![nextjs-sql.png](https://github.com/milkymilky0116/obsidian-backup/blob/main/image/HTMX/nextjs-sql.png?raw=true)
_이... 이건 좀...._

정리를 하자면 다음과 같다.

- 하이퍼미디어 API
  - 서버와 클라이언트 결합도가 높을때 사용하기 좋음
  - 상태를 저장하는 주체는 오직 서버이고, 클라이언트는 그것을 단지 반영하기만 함
- 데이터 API
  - 서버와 클라이언트의 분리가 명확하게 일어나야 할때 사용하기 좋음
  - 서버와 클라이언트 각각 상태 관리를 해야하고, 이 과정에서 불균형이 일어날 수도 있음

### HTMX: 상태관리를 위한 최적의 솔루션?

위의 챕터를 읽었다면 HTMX가 어떤 생각으로부터 비롯된 라이브러리인지 대충 눈치 챈 독자도 있을 것이다. HTMX는 겉으로 보았을때 프론트엔드 라이브러리를 표방하고 있지만, 서버와 전체적인 웹 애플리케이션이 작동되는 방식을 바꿔놓는다. HTMX는 서버에서 하이퍼미디어 API를 받아오길 원하고, 클라이언트와 서버간의 응집도를 높히려고 한다. HTMX는 _"클라이언트는 서버가 가지고 있는 상태의 반영 "_ 이라는 철학을 가지고 클라이언트 단을 현대의 웹앱의 형태를 갖추면서도 최대한 심플하게 관리되는 것을 원한다. HTMX는 프론트엔드 라이브러리이지만, 결과적으로 상태관리를 위한 최적의 솔루션이 된것이다.

아무리 그래도 상태관리를 위해 서버에서 HTML 컴포넌트를 관리해야 하는게 결과적으로 유지보수의 어려움으로 이어지지 않을까? 이 말은 맞을수도 있고 아닐수도 있다. 서버에서 날 것 그대로의 HTML 문자열을 보내는 형식으로 개발하게 된다면 당연히 어려움이 뒤따라올 것이다. 하지만 HTMX와 함께 템플릿 엔진을 적극적으로 활용해보니, 생각했던 것보다 좋은 개발자 경험을 얻을 수 있다는 사실을 알게 되었다. 또한 [Web Components](https://developer.mozilla.org/en-US/docs/Web/API/Web_components)라는 자바스크립트의 기술을 활용하면, React보단 어려워도 컴포넌트 기반의 개발을 충분히 구현해 나갈 수 있다. 아래 코드는 필자가 짠 컨트롤러 코드중 한 부분이다. 생각보다 간단해보이지 않는가? status code와 함께 템플릿을 렌더링하는 요청을 반환하면, HTMX는 그 요청을 받고 타겟으로 지정해둔 다른 HTML 요소와 바꾸면서 DOM을 업데이트 한다.

```go
// ...
func (h *HomeHandler) Add(ctx echo.Context) error {
	context := ctx.FormValue("context")
	if context == "" {
		errorMsg := components.ErrorMsg("invalid form value")
		return utils.Render(ctx, errorMsg, http.StatusInternalServerError)
	}
	state, ok := ctx.Get("state").(*types.State)
	if !ok {
		ctx.Error(errors.New("context error"))
	}
	log.Println(state)
	h.TodoService.Add(state.Length+1, context)

	todosComponents := components.Todos(h.TodoService.List())
	return utils.Render(ctx, todosComponents, http.StatusOK)
}
//...
```

```html
<form
  class="flex items-center gap-4 mb-6"
  hx-post="/add"
  hx-target="#todos"
  hx-swap="outerHTML"
  hx-target-5*="#msg"
>
  ...
</form>
```

HTMX의 작동 방식을 더 쉽게 이해하기 위해, HTMX에서 요청을 받고 다른 HTML 요소와 swap 하는 과정을 그림으로 그려보면 다음과 같다.

![HTMX-working.png](https://github.com/milkymilky0116/obsidian-backup/blob/main/image/HTMX/HTMX-working.png?raw=true)

HTML attribute에 명시된 hx-post 요청으로 서버에 `/add` 로 요청을 보내면 , 서버는 그 반환으로 어떤 HTML 요소를 가져온다. HTML 요소를 응답받은 form 태그는 역시 HTML attribute에 명시된 hx-target과 hx-swap 을 보고 특정 요소를 특정한 방식으로 업데이트 하거나 바꿔치기 한다. 그림에서는 target이 `id: todos` 인 HTML 요소를 `outerHTML` 방식으로 바꿔달라고 하고 있다. 여기서 `outerHTML`은 요소 전체를 바꾼다는 것을 의미한다. 물론 `outerHTML` 뿐만 아니라 그 안의 자식 요소로 들어가는 등의 여러가지 swap 방식도 존재한다.

이러한 여러가지 방식을 조합해서 다양하게 DOM을 조작 할 수 있고, React에 못지 않은 웹 앱을 구성하는 것도 가능하다. 아래 데모는 HTMX + Golang 서버를 이용해서 만든 간단한 to-do 앱이다.

![demo.gif](https://github.com/milkymilky0116/obsidian-backup/blob/main/image/HTMX/demo.gif?raw=true)

### HTMX를 써보며 어려웠던 점들

1. "상태관리"에 대한 방법을 다시 생각해야만 했다.

앞서 말했듯, 하이퍼미디어 API 방식을 이용한 웹 애플리케이션은 상태를 쥐고 있는 주체가 서버라고 언급했다. 서버에서는 주로 DB에 상태저장을 하는데, 이 부분이 처음 HTMX를 개발할때 난해하게 느껴질 수도 있다. 예를 들어서 위의 데모처럼 수정버튼을 눌렀을때 input으로 바뀌게 하고 싶은데, 그러한 상태까지 DB에 저장해야 할까? 영구적인 변화가 아니라 잠시동안만 유지되는 DOM 조작을 하고 싶다면 어떻게 해야할까?

이 부분은 좀 더 생각해야할 부분이다. 위의 데모에서는 간단한 트릭으로 새로고침 하면 edit 상태가 풀리게끔 해결했지만, 기본적으로 HTMX는 모든 행동이 서버의 엔드포인트에 요청하게끔 되어있기 때문에 만일 DOM 조작을 해야하는 경우가 많아진다면, 그 모든 조작에 대해서 엔드포인트를 만들어야 하고 상태를 어디엔가 저장해야 할 것이다. 어쩌면 DB에 굳이 저장해야할 일이 아닌데도 상태를 저장하고 새로운 필드를 만들어야 한다면, 꽤 번거로운 일이 될수도 있다.

이것에 대한 솔루션은 적용 해보지는 않았지만 몇가지 떠오른 방법을 이야기 하자면, 첫번째는 Golang 한정으로 context 패키지를 이용하는 것이다. Golang에서 context는 애플리케이션의 life cycle 동안 특정한 상태를 저장할 수 있는 패키지이다. `context.WithValue()` 메소드를 통해 애플리케이션이 돌아가고 있을때만 상태를 저장 할 수 있고, 이를 통해 DB에 모든 상태를 저장해야 했던 번거로움을 덜 수 있을 것이다.

두번째는 [Alpine.js](https://alpinejs.dev/)를 활용하는 것이다. Alpine.js는 HTMX와 비슷한 라이브러리이지만, 로컬변수를 선언해 DOM의 상태를 조작할 수 있다는 큰 장점이 있는 라이브러리이다. 그렇다면 짧게 상태가 유지되어야 하는 경우에 Alpine.js를 활용해 서버 엔드포인트에 요청하지 않고도 DOM을 조작할 수 있어 매우 편리할 것이다. 실제로 검색해보니 HTMX와 Alpine.js를 같이 활용하는 사례를 많이 보았고, 두 라이브러리가 함께 쓰이면 꽤 좋은 시너지를 내지 않을까 싶어 좀 더 탐구 해볼 생각이다.

2. 에러 핸들링은 어떻게 해야할까

form을 다루다보면 적절하지 못한 요청에 대해서도 처리해야 하는 경우가 종종 있다. 로그인을 할때 중복된 이메일을 검증 한다던지, 패스워드가 틀렸을때를 생각해보면 이해하기 쉬울 것이다. HTMX에서는 어떻게 해결 할 수 있을까?

정답은 기본 HTMX 기능만을 활용해서 해결하기는 어렵고, HTMX 팀에서 공식적으로 지원하는 익스텐션을 활용하면 쉽게 해결할 수 있다. [response-target](https://htmx.org/extensions/response-targets/) 을 이용하면 status code에 따라 다른 타겟을 지정해서 DOM 조작이 가능하다. 아래 코드를 보면 이해하기 쉬울 것이다.

```html
<div hx-ext="response-targets">
  <div id="response-div"></div>
  <button
    hx-post="/register"
    hx-target="#response-div"
    hx-target-5*="#serious-errors"
    hx-target-404="#not-found"
  >
    Register!
  </button>
  <div id="serious-errors"></div>
  <div id="not-found"></div>
</div>
```

만일 정상적인 응답이 올 때와 에러 응답이 왔을때, swap 방식을 다르게 해서 처리해야 하는 경우라면 어떨까? (에러가 생겼을때 플래시 메세지를 띄우는 것처럼 말이다.) 그럴때는 [multi-swap](https://htmx.org/extensions/multi-swap/)을 활용하면 된다. 여러 타겟을 지정할 수 있고, 각각의 타겟에 따라서 swap 방법을 다르게 하는 것도 물론 가능하다.

```html
<body hx-boost="true" hx-ext="multi-swap">
  <!-- simple example how to swap #id1 and #id2 from /example by innerHTML (default swap method) -->
  <button hx-get="/example" hx-swap="multi:#id1,#id2">
    Click to swap #id1 and #id2 content
  </button>

  <!-- advanced example how to swap multiple elements from /example by different swap methods -->
  <a
    href="/example"
    hx-swap="multi:#id1,#id2:outerHTML,#id3:beforeend,#id4:delete"
    >Click to swap #id1 and #id2, extend #id3 content and delete #id4 element</a
  >

  <div id="id1">Old 1 content</div>
  <div id="id2">Old 2 content</div>
  <div id="id3">Old 3 content</div>
  <div id="id4">Old 4 content</div>
</body>
```

### 결론

![javascript-rising-stars.png](https://github.com/milkymilky0116/obsidian-backup/blob/main/image/HTMX/javascript-rising-stars.png?raw=true)

[2023 Javascript Rising Stars](https://risingstars.js.org/2023/en#section-framework) 라는 자바스크립트 라이브러리 순위를 살펴보면, HTMX가 다른 쟁쟁한 라이브러리들과 함께 10위권에 안착 해 있음을 볼 수 있다. HTMX가 이처럼 주목받는 이유는, 피로감이 극심해져가는 프론트엔드 생태계에서 훌륭한 대안으로 제시될 수 있었기 때문에 그런 것이 아닐까 싶다. 그것도 꽤 신선하고 심플한 방법으로 말이다.

무엇보다 HTMX는 백엔드 개발자에게도 의의가 큰 라이브러리이다. 기존의 JSON, GraphQL 등등의 데이터 API 에서 벗어나 하이퍼미디어 API 로 웹 애플리케이션을 설계하는 방식에 있어서 다시 생각해볼 수 있는 기회를 주고, 그 방법이 심지어 꽤 괜찮다는 것까지 확인 할 수 있다.

HTMX가 React를 완벽히 대체할 수 있는가? 앞서 말했듯 그것은 아니다. 하지만 개발할때 매우 즐거웠는가? 그렇다. 앞으로 사이드 프로젝트에서 계속 쓸 의향이 있는가? 매우매우매우 그렇다. HTMX는 그 방향성에 신선한 충격을 받았기에 앞으로도 미래가 기대되는 라이브러리이다.

HTMX 팀의 시조를 마무리로 HTMX 소개글을 마치겠다.

> javascript fatigue:  
> longing for a hypertext  
> already in hand

> 피곤한 자바스크립트  
> 하이퍼텍스트의 갈망  
> 이미 내 손안에
