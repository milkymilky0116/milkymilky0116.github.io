---
title: 에러 핸들링에 관한 식견 짧은 소고 - (feat. typescript, golang)
date: 2023-12-29 16:57:43 +/-TTTT
categories: [golang, typescript, essay]
tags: [golang,typescript,error handling]
mermaid: true # TAG names should always be lowercase
---

학부생 시절까지만 해도 에러 핸들링에 관해서 그다지 관심을 두지 않았었다. 프로그램은 요구사항 대로 잘 돌아가기 만 하면 장땡이라고 생각했고, "안정성 있는 코드"에 대해 깊게 생각하지 못했다. 하지만 최근 몇 년간 백엔드, 조금의 프론트를 공부하고 프로젝트를 해보면서 "어떻게 에러를 잘 처리해야 하는지"에도 관심이 많이 생기게 되었다. 그리고 이 글은 나름대로 여러 언어들로 개발하면서 겪었던 '에러 핸들링'에 관한 경험을 소개하려는 취지로 작성되었다. 물론 식견이 짧은 주니어 개발자의 글이라서 여러 부분에서 틀린 점이 존재할 수도 있으니 '이런 생각을 가지고 있구나~' 정도로 보고 넘어가 주시면 되겠다.

### try-catch 와 err != nil

Python/Typescript 개발자가 Golang을 처음 접했을 때, Golang의 에러 핸들링 방식을 보고 충격받았었다. 그 방식이 슬쩍 보기에는 너무나도 단순하고 예뻐 보이지 않았기 때문이다. 그래서 이런 의문이 들었다. 왜 golang은 try-catch를 도입하지 않았을까?

우선 아래와 같은 상황을 가정하자.
1. `foo()` 와 `bar()`라는 함수 두 가지를 써야 하는 상황이고, 두 함수 모두 에러가 생길 수 있는 여지가 있다.
2. `foo()` 와 `bar()`는 integer(혹은 number)를 리턴한다.
3. 에러를 최대한 잘 핸들링 하고 싶다.

Typescript로 코드를 작성하면 이런 형태일 것이다.

```typescript
function foo():number {
	//...
	return 1
}

function bar():number {
	//...
	return 2
}

async function main() {
	try {
		const foo = await foo();
		const bar = await bar();
	}catch(error){
		// some error handling...
	}
}
```

하지만 Golang 으로 작성한다면 이런 형태일것이다.

```go
func foo() (error,int) {
	//...
	return nil, 1
}

func bar() (error,int) {
	return nil, 2
}

func main() {
	err, foo := foo()
	
	if err != nil {
		// some error handling...
	}
	
	err, bar := bar()
	
	if err != nil {
		// some error handling...
	}
}
```


언뜻 보기에 Golang의 방식은 전혀 우아하지 않다. error를 인자로 넘겨주는 Golang의 방식보다 try-catch 블록으로 함수를 감싸는 Typescript의 방식이 더 간결해 보이고 언어-디자인적으로 봤을 때 꽤 예뻐 보일지도 모르겠다.  
  
Golang의 에러 핸들링 방식은 try-catch에 익숙해져 있는 개발자가 보기에 많은 의문을 남긴다. 만약 에러가 발생할 수 있는 함수들이 수십 가지라면? 코드 전체가 `if err != nil`로 채워질 것이고 코드는 꽤.. 못생겨 보일 것이다.

```go

func main() {
	_, err := somefunctionA()
	
	if err != nil {
		//some error handling..
	}
	
	_, err = somefunctionB()
	
	if err != nil {
		//some error handling..
	}
	
	_, err = somefunctionC()
	
	if err != nil {
		//some error handling..
	}
	
	_, err = somefunctionD()
	
	if err != nil {
		//some error handling..
	}
	
	//...
}
```


![reddit](https://raw.githubusercontent.com/milkymilky0116/obsidian-backup/main/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-12-28%20%EC%98%A4%ED%9B%84%207.36.08.png)
*Golang의 에러 핸들링 방식은 전혀 우아해 보이지 않고, 난잡해 보이기까지 한다.*

### err != nil 은 못생겼다. 하지만...

상황을 하나 더 추가해 보자. 이번에는 `foo()` 와 `bar()`에서 발생할 수 있는 에러를 각각 다르게 처리하고 싶다. 그리고 `foo()`와 `bar()`의 값을 합한 값을 리턴 하고 싶다.  
  
Typescript로 해결하려고 보니, 점점 머리가 아파지기 시작한다. try-catch 블록을 나눠서 해결하려고 하니, foo와 bar의 값을 합친 값을 리턴할 수가 없다. 왜냐하면 try-catch 블록 안에서 선언된 변수는 해당 scope 에서만 존재할 수 있기 때문이다.

```typescript
async function foo():number {
	//...
	return 1
}

async function bar():number {
	//...
	return 2
}

async function main():Promise<number> {

	try {
		const foo : Promise<number> = await foo();
	}catch(error){
		// some foo error handling...
	}
	
	try {
		const bar : Promise<number> = await bar();
	}catch(error){
		// some bar error handling...
	}
	
	return foo + bar; // error: foo와 bar가 선언되지 않음
}
```

그렇다면 코드를 개선해보자.

```typescript
function foo():number {
	//...
	return 1
}

function bar():number {
	//...
	return 2
}

async function main():Promise<number> {
	let foo:Promise<number | undefined> = undefined;
	let bar:Promise<number | undefined> = undefined;
	
	try {
		foo = await foo();
	}catch(error){
		// some foo error handling...
	}
	
	try {
		bar = await bar();
	}catch(error){
		// some bar error handling...
	}
	
	return foo + bar;
}
```

상황이 점점 좋지 않게 흘러간다. try-catch 블록 바깥에서 변수를 사용하기 위해 복잡한 타입을 가진 변수를 선언해야 하고, 그 변수가 number도 될 수 있고 undefined도 될 수 있는 상황이 되어버렸다. 만일 로직이 복잡해지거나 `foo`,`bar` 변수를 사용해야 하는 경우, 변수가 undefined인 경우까지 고려해서 에러가 생기지 않도록 핸들링 해줘야 하는 작업까지 필요할 수도 있다.  
  
물론 try-catch 블록을 한 번만 쓰고, catch 블록에서 에러 분기를 나누어 해결하는 방법도 있다.

```typescript
//...
async function main():Promise<number> {
	try {
		const foo:number = await foo();
		const bar:number = await bar();
		
		return foo + bar;
		
	}catch(error){
		switch (error) {
		case error instanceof fooError:
			//some foo error handling..
		case error instanceof barError:
			//some bar error handling..
		}
	}
}
```

하지만 이렇게 catch 블록에서 에러 분기를 나누어 해결하는 방식에는 몇가지 문제점이 있다.

- **정확히** 어떤 에러가 발생하는지 파악해야 한다. 그런데 내가 직접 짠 함수가 아닌 라이브러리 단에서 발생하는 에러라면? 발생할 수 있는 에러들을 전부 검수하고 분기를 나누는 과정이 결코 쉽지는 않을 것이다.

- 코드의 직관성이 떨어진다. 위 코드 예시에서는 에러 인스턴스의 이름이 함수 이름을 포함하고 있다고 가정해서 읽기 쉽지만, 대부분의 에러들은 그렇지 않다. 어떤 함수에서 에러가 발생할 수 있는지 코드만 보고서는 파악하기 쉽지 않다.

그렇다면 Golang에서 이 문제를 해결해 보자. Golang에서 위의 문제를 해결하는 것은 매우 간단하다.

```go
func foo() (error, int) {
	//...
	return nil, 1
}

func bar() (error, int) {
	//...
	return nil, 2
}

func main() int {
	err, foo := foo()
	
	if err != nil {
		// some foo error handling...
	}
	
	err, bar := bar()
	
	if err != nil {
		// some bar error handling...
	}
	
	return foo + bar
}
```

이 코드의 이점은 다음과 같다.
- 어떤 함수에서 에러가 발생할 수 있는지 쉽게 알 수 있다.

- 특정 함수에서 발생하는 에러를 쉽게 다룰 수 있다. 정확하게 어떤 에러가 발생하는지 파악하지 않아도 되고, 에러 핸들링이 변수의 타입 시그니처에 영향을 미치지도 않는다.

- 코드가 직관적이다! Golang을 모르는 사람이 보더라도 코드가 어떤 일을 수행하고 있는지 명확하게 알 수 있다.

이러한 연유에서, `if err != nil` 로 에러를 핸들링 하는 것이 처음에는 썩 예쁘게 보이지 않더라도 try-catch 에 비해 많은 이점을 가져다 주는 것을 알 수 있다.

![reddit](https://raw.githubusercontent.com/milkymilky0116/obsidian-backup/main/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-12-29%20%EC%98%A4%ED%9B%84%202.55.41.png)


> (의역)
>마지막으로, 그리고 가장 중요한 것은 예외들이 자주 남용되고 엉성한 코딩 관행으로 이어질 수 있다는 것입니다. 
>예외들은 선언되지 않고 문서화되지 않기 때문에 방법이나 함수가 잠재적으로 던질 수 있는 모든 예외들을 알아챌 수 없습니다. 또는 하나의 예외가 수많은 다른 오류에 사용될 수도 있는데, 예를 들어 잘못된 파일 이름과 전체 디스크 모두에 대해 "SaveFailedException"을 던질 수도 있습니다. 예외들은 종종 단순히 무시되고 스택에서 다시 넘겨져 맨 위에서만 처리되거나, 그냥 완전히 무시되고 일부 일반 예외가 기록되도록 합니다.

> Go 개발자들은 대신 오류가 "발생했을 때" 처리하는 것이 가장 좋다고 주장합니다. 그래서 모든 오류에 대해 생각하고, 코드의 모든 오류 사례(ErrFileNotFound, ErrDiskFull 등)를 문서화할 수 있습니다. 이것은 또한 Golang이 자바나 C++ 앱보다 훨씬 빠르게 컴파일할 수 있다는 것을 의미합니다. (사실 예외 처리보다 더 큰 이유가 있지만, 예외 처리도 그 점중 하나입니다.).


[Reddit - 왜 Golang은 try-catch를 채택하지 않았나요?](https://www.reddit.com/r/golang/comments/9t7con/why_no_trycatch_in_golang_whats_the_theory_behind/)

### 누가 에러 핸들링 하라고 칼들고 협박함?

![누칼협](https://raw.githubusercontent.com/milkymilky0116/obsidian-backup/main/%EC%97%90%EB%9F%AC%20%ED%95%B8%EB%93%A4%EB%A7%81.jpeg)


그런데 다시 근본적인 질문으로 돌아가자. 우리에게 정말 에러 핸들링이 필요할까? 그냥 프로그램은 잘 돌아간다면 장땡 아닐까?  
  
그런데 문제는, 에러는 어디에나 산재해있을 수 있다는 것이다. 그리고 그 에러가 여러분이 짠 코드에서 나오지 않을 수도 있다.  
  
단순하게 string을 JSON으로 변환해 주는 함수에 대해서 생각해 보자.

```typescript
JSON.parse('{"id":1, "name":"milky"}');
```

종종 많은 사람들이 `JSON.parse()`에 발생할 수 있는 에러에 대해 생각하지 않고 넘어가는 경우를 보아왔다. 당연히 파라미터가 잘 주어질 것이라고 생각하고, 당연히 string을 JSON으로 변환하는 로직이 잘 작동할 것이라고 생각한다.

하지만 이런 경우에 대해서 생각해 봐야 한다.

- **정말** 파라미터가 잘 주어질 것이라는 기대가 있을까? 만일 `}` 을 뺀 string이 주어진다면? 그 이외에 문법적인 오류가 있는 string이 주어진다면?
- 어떠한 연유에서, *정말 알 수 없는 이유로* string을 JSON으로 변환하는 과정에서 에러가 발생한다면?

첫 번째 이유는 그렇다 치는데, 두 번째 이유가 조금 황당해 보일지도 모르겠다. 우리가 정말 그런 상황을 고려해야 할까? 대답은 그럴 수도 있고, 아닐 수도 있다. 하지만 안정성을 필요로 하는 프로그램을 짜야 한다면, 반드시 고려해야 한다.

아닐수도 있는 경우는 다음과 같다.

- 정말 오래 유지 보수되고 커뮤니티에 의해 안정성이 있다고 평가받은 라이브러리의 함수는 그냥 에러 핸들링을 하지 않고 넘어갈 수도 있다.
- 내가 짜려고 하는 프로그램에 굳이 안정성이 필요하지 않다. 간단하게 짜려는 프로그램인데 모든 케이스를 고려하고 싶지 않다.

하지만 만일 프로그램이 오랜 시간 동안 돌아가야 하고, 어떤 상황에도 잘 돌아갈 수 있게끔 안정되어야 하고, 만일 에러가 발생하면 복구해서 서비스를 원활하게 돌아가게 하고 싶은 등 "믿을 수 있을만한" 프로그램을 설계 중이라면 고려해야 한다.

이러한 이유에서, try-catch를 굳이 강제하지 않는 Typescript와 달리 Golang은 문법 구조적으로, 그리고 관습적으로 에러 핸들링을 장려한다. Golang 개발자 팀이 try-catch를 넣지 않은 이유는 바로 이러한 이유에서이다. 에러 핸들링은 프로그램의 신뢰성에 있어서 중대한 문제이기 때문에, 그러한 관습을 문법적인 구조에서부터 장려하는 Golang의 에러 핸들링 방식은 예쁘게 보이진 않더라도 "믿을 수 있는" 코드를 짜기에 매우 좋은 방법이다. 게다가 읽기도 쉽지 않은가.

```go
//...
	args := os.Args[1:]
	
	config, err := cli.ParseArgs(args)
	if err != nil {
		log.Println(err)
	}
	
	questions, err := quiz.ParseCsv(config.FileName, config)
	
	if err != nil {
		log.Println(err)
	}
	
	result, err := questions.ParseUserInput()
	
	if err != nil {
		log.Println(err)
	}
//...
```


### 결론

물론 Golang의 에러 핸들링 방식이 완벽하지는 않다. 조금 더 모던한 프로그래밍 언어로 오면 에러 핸들링에 대해 여러 가지 방법으로 접근하고 있는 것을 알 수 있다.

개인적으로 Rust의 `Result<T,E>` 타입과 match 표현식이 매우 마음에 들었다. Golang 보다 더 직관적이고, 코드를 파악하기 매우 쉽게 느껴졌다.

```rust
fn main() {
    let data_result = File::open("data.txt");

    let data_file = match data_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the data file: {:?}", error),
    };

    println!("Data file", data_file);
}
```

배워본 적은 없지만, Zig 라는 프로그래밍 언어에서는 `if err != nil` 을 `try` 라는 키워드로 대체해서 가독성을 높였다고 한다.

```zig
pub fn main() !void {  
	const input = -10;  
	const number = try maybeError(input);  
	std.debug.print("Number: {}\n", .{number});  
}
```


최신의 프로그래밍 언어로 올수록 try-catch 블록보다는 다른 방식으로 해결하려는 경향이 많이 보이고, 이러한 방향성에 개인적으로는 매우 동감한다.

최근 Typescript(Nest.js) 와 Golang 두 언어로 동시에 프로젝트를 진행하면서 에러 핸들링에 대한 차이점을 피부로 느낄 수 있었고, 개인적인 생각을 글 속에 녹여보았다. 결론적으로는 Golang에서의 경험이 더 좋았다고 생각하고, 더 안정성 있는 프로그램을 구성하기에 좋은 환경이었던 것 같았기에 `if err != nil` 이 더 좋은 방식인 것 같다고 조심스럽게 의견을 내비치며 글을 마무리 짓겠다.
