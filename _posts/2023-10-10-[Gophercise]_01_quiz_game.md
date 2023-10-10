---
title: (Gophercises) 01.Quiz Game
date: 2023-10-10 16:53:43 +/-TTTT
categories: [TIL, Summary]
tags: [golang, cli, gophercises]
mermaid: true # TAG names should always be lowercase
---

## Exercise #1: Quiz Game

[문제링크](https://courses.calhoun.io/lessons/les_goph_01)

### 문제 설명

본 과제는 CSV 파일에서 퀴즈들을 불러와서 유저들에게 문제를 맞추게 하는 프로그램을 작성하는 것입니다.

문제를 풀때 유저가 틀리거나 잘못된 값을 넣어도 프로그램을 종료 시키는 것이 아니라, 틀린것으로 처리해서 유저가 얼마나 답을 맞췄는지 보여줘야해요.

csv 파일은 기본적으로 `problems.csv` 이지만 유저가 flag를 통해서 파일 이름을 넣을수 있게 프로그램을 작성해야 합니다.

csv 파일은 다음과 같은 포맷입니다.

```csv
5+5,10
7+3,10
1+1,2
8+3,11
1+2,3
8+6,14
3+1,4
1+4,5
5+1,6
2+3,5
3+3,6
2+4,6
5+2,7
```

또한, 퀴즈에는 타이머도 있어야 합니다. 타이머는 기본적으로 30초이지만 flag를 통해 유저가 커스터마이징 할 수 있게 해줘야 합니다.

타이머가 넘어가는 순간 퀴즈를 멈추고 결과값을 출력해야 합니다.

- 보너스

1.  유저가 답을 넣을때 공백, 대문자 등이 추가되더라도 틀린것으로 간주하지 않게 해보세요.

### 내 풀이

우선 해당 과제의 구조는 다음과 같습니다.

```
📂cli
┗📎 cli.go
📂quiz
┗📎 quiz.go
📎 main.go
✏ test.http
```

### 📎 cli.go

`cli.go` 에서는 단순히 유저에게 flag를 받아서 config를 넘겨주는 역할을 하고 있습니다.

문제의 요구사항대로 filename과 timer를 각각 flag로 받고 있습니다.

```go
package cli

import "flag"

type Config struct {
	FileName string
	Timer    int
}

func ParseArgs(args []string) (Config, error) {
	var c Config
	fs := flag.NewFlagSet("Quiz Game", flag.ContinueOnError)
	fs.StringVar(&c.FileName, "filename", "problems.csv", "enter your csv file name")
	fs.IntVar(&c.Timer, "timer", 30, "Time Limit to solving quiz")
	err := fs.Parse(args)
	if err != nil {
		return c, err
	}
	return c, nil
}
```

### 📎 quiz.go

#### ParseCSV

```go
type Quiz struct {
	Quiz   map[string]string
	Result int
	index  int
	config cli.Config
}

func ParseCsv(filename string, config cli.Config) (*Quiz, error) {
	var q Quiz
	q.Quiz = map[string]string{}
	file, err := os.Open(filename)
	if err != nil {
		return nil, err
	}
	defer file.Close()
	reader := csv.NewReader(file)
	data, err := reader.ReadAll()
	if err != nil {
		return nil, err
	}
	r, _ := regexp.Compile("[0-9]+")
	for _, row := range data {
		q.Quiz[row[0]] = strings.TrimSpace(r.FindString(row[1]))
	}
	q.config = config
	return &q, nil
}
```

`ParseCSV` 에서는 파일을 받아서 map 형태로 파싱하는 역할을 맡고 있습니다. Quiz 라는 모델을 만들어서 값을 다시 반환하는 형태로 작성되었습니다.

#### ParseUserInput

```go
func (q *Quiz) ParseUserInput() (*Quiz, error) {
	q.index = 1
	for question, answer := range q.Quiz {
		var userAnswer string

		channel := make(chan bool)
		errs := make(chan error, 1)
		outputQuestion := fmt.Sprintf("Problem #%d %s : ", q.index, question)

		go func() {
			fmt.Print(outputQuestion)
			reader := bufio.NewReader(os.Stdin)
			result, err := reader.ReadString('\n')
			if err != nil {
				errs <- err
			}
			result = result[:len(result)-1]
			userAnswer = strings.TrimSpace(result)
			if err != nil {
				errs <- err
			}
			channel <- true
		}()

		timeoutChan := time.After(time.Duration(q.config.Timer) * time.Second)
		select {
		case <-channel:
			userAnswer = strings.ToLower(userAnswer)
			answer = strings.ToLower(answer)
			if userAnswer == answer {
				q.Result++
			}
			q.index++

		case <-timeoutChan:
			err := errors.New("timeout exceed")
			return q, err
		}

	}
	return q, nil
}
```

이 부분이 가장 핵심이자 어려운 부분인데요, 유저에게 input을 받는 역할을 하고 있는 메소드입니다.

문제를 받아서 입력받게 하는건 그렇게 어렵지 않지만, 거기에 타이머를 붙여 timeout을 구현하는 것은 쉽지 않았습니다.

저는 timeout을 goroutine을 이용해서 구현했습니다.

```go
	for question, answer := range q.Quiz {
		var userAnswer string

		channel := make(chan bool)
		errs := make(chan error, 1)
		outputQuestion := fmt.Sprintf("Problem #%d %s : ", q.index, question)

		go func() {
			fmt.Print(outputQuestion)
			reader := bufio.NewReader(os.Stdin)
			result, err := reader.ReadString('\n')
			if err != nil {
				errs <- err
			}
			result = result[:len(result)-1]
			userAnswer = strings.TrimSpace(result)
			if err != nil {
				errs <- err
			}
			channel <- true
		}()

		timeoutChan := time.After(time.Duration(q.config.Timer) * time.Second)
```

퀴즈 리스트를 돌면서 사용자에게 문제를 출력하고 값을 받게 하는데, 이때 timeout을 위해 goroutine을 이용했습니다.

goroutine 안에서 문제 출력과 사용자에게 input을 받게 합니다. 이때 채널을 두개 만들어서 정상적으로 문제의 답이 입력되면 `channel`로, 에러가 발생하면 `errs`로 가게 로직을 짰습니다.

그리고 timeout 되는 경우도 처리해줘야 하므로, goroutine 이후에 `time.After`를 이용해서 timeout 될때의 채널도 만들어주었습니다.

```go
		select {
		case <-channel:
			userAnswer = strings.ToLower(userAnswer)
			answer = strings.ToLower(answer)
			if userAnswer == answer {
				q.Result++
			}
			q.index++

		case <-timeoutChan:
			err := errors.New("timeout exceed")
			return q, err
		}
```

그리고 이후에 이렇게 채널별로 오는 값들의 케이스에 따라서 예외 처리를 해주고 있습니다. 여기서 `errs`는 일부러 처리를 안했는데요, 잘못된 값이 와도 그냥 넘겨야 하는 문제의 요구사항이 있었기 때문에 넣지 않았습니다.

정상적으로 문제가 입력됐다면 사용자가 입력한 값과 문제를 비교해서 Quiz 모델안의 `result`와 `index` 값을 갱신합니다.

timeout이 됐다면 그대로 에러를 반환해서 전체 메소드를 종료시킵니다.

### 결과

![스크린샷 2023-10-10 오후 5 44 02](https://github.com/milkymilky0116/gophercises-practice/assets/84823612/d38b086b-135e-4d69-b22d-2f77af0ceb48)
![스크린샷 2023-10-10 오후 5 43 32](https://github.com/milkymilky0116/gophercises-practice/assets/84823612/2c828c69-12ea-4e27-beb6-2529eca91b0e)
