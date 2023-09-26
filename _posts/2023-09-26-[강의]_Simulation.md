---
title: (코테) 시뮬레이션 문제 5가지
date: 2023-09-20 00:10:11 +/-TTTT
categories: [TIL, CodePractice]
tags: [python, codepractice, algorithm]
math: true
mermaid: true # TAG names should always be lowercase
---

# 문제 1. Pond

매개변수 nums에 n행 n열의 이차원 배열에 격자판 정보가 주어집니다.
각 격자에는 그 지역의 높이가 쓰여있습니다. 각 지역은 상하좌우 인접한 지역의 숫자가 모두 자신보다 클 경우 이 지역을 웅덩이 지역이라고 합니다.
격자의 가장자리는 1000으로 초기화 되었다고 가정한다.
만약 5\*5 이차원 배열의 격자판 정보다 아래와 같다면 총 웅덩이의 수는 5개입니다.

주어진 격자에 웅덩이가 몇 개 있는지 찾아 그 개수를 반환하는 프로그램을 작성하세요.

## 풀이

```python
def solution(nums: list[list[int]]) -> int:
    dx: list[int] = [-1, 0, 1, 0]
    dy: list[int] = [0, 1, 0, -1]
    answer = 0
    for i, _ in enumerate(nums):
        for j, _ in enumerate(nums[i]):
            flag = True
            for k in range(4):
                if (
                    i + dx[k] >= 0
                    and i + dx[k] < 5
                    and j + dy[k] >= 0
                    and j + dy[k] < 5
                ):
                    if nums[i][j] >= nums[i + dx[k]][j + dy[k]]:
                        flag = False
                        break

            if flag:
                answer += 1
    return answer
```

`dx,dy`에서 매치되는 인덱스의 좌표는 상하좌우의 방향을 나타낸다.

2차원 배열이기 때문에 이중 for 문을 돌려서 해당 값에 인접하는 네 방향의 값이 자신보다 모두 큰지를 판별하면 된다.

이때 `index out of range` 에러가 나면 안되기 때문에 해당 좌표에 `dx,dy`를 더한 값이 격자 밖을 안넘어가는지 판별하는 조건문이 필요하다.

만일 네 방향 모두 자신보다 크다면 answer에 +1을 해준다.

# 문제 2. 청소로봇 (ver 1)

청소 로봇은 다음 규칙에 따라 이동합니다.

1. 'U' 명령은 로봇이 위쪽으로 한 칸 이동합니다.
2. 'R' 명령은 로봇이 오른쪽으로 한 칸 이동합니다.
3. 'L' 명령은 로봇이 왼쪽으로 한 칸 이동합니다.
4. 'D' 명령은 로봇이 아래쪽으로 한 칸 이동합니다.
   매개변수 moves에 청소 로봇에 명령을 내린 문자들이 차례대로 나열된 명령 문자열이 주어지 면 이 명령 문자열을 청소 로봇이 모두 수행했을 때 최종 위치를 반환하는 프로그램을 작성하 세요. 격자판 밖으로 벗어나는 명령은 주어지지 않습니다.

## 풀이

```python
def solution(moves: str) -> list[int]:
    init_cords = [0, 0]
    for move in moves:
        if move == "U":
            init_cords[0] -= 1
        elif move == "R":
            init_cords[1] += 1
        elif move == "L":
            init_cords[1] -= 1
        elif move == "D":
            init_cords[0] += 1
    return init_cords
```

약간 간단 무식하게 풀었다. 해당하는 문자열이 이동하는 만큼 좌표를 더해준다.
강의에서는 앞선 풀이처럼 `dx,dy` 리스트를 만들고 좀 더 간결하게 리팩토링을 했던데 나중에 다시 시도해봐야겠다.

# 문제 3. 청소로봇 (ver 2.)

청소 로봇은 다음 규칙에 따라 이동합니다.

1. 'U' 명령은 로봇이 위쪽으로 한 칸 이동합니다.
2. 'R' 명령은 로봇이 오른쪽으로 한 칸 이동합니다.
3. 'L' 명령은 로봇이 왼쪽으로 한 칸 이동합니다.
4. 'D' 명령은 로봇이 아래쪽으로 한 칸 이동합니다.
5. 만약 로봇이 명령을 수행할 경우 격자판 밖으로 나가는 경우라면 로봇은 해당 명령을 수행 하지 않고 무시합니다.
   매개변수 n에 격자판 크기가 주어지고, moves에 청소 로봇에 명령을 내린 문자들이 차례대로 나열된 명령 문자열이 주어지면 청소 로봇이 최종적으로 멈춘 위치를 반환하는 프로그램을 작 성하세요.

## 풀이

```python
def solution(moves: str, size: int) -> list[int]:
    init_cords = [0, 0]
    for move in moves:
        if move == "U":
            if init_cords[0] - 1 >= 0:
                init_cords[0] -= 1
        elif move == "R":
            if init_cords[1] + 1 < size:
                init_cords[1] += 1
        elif move == "L":
            if init_cords[1] - 1 >= 0:
                init_cords[1] -= 1
        elif move == "D":
            if init_cords[0] + 1 < size:
                init_cords[0] += 1
    return init_cords
```

이번에는 격자판으로 나갈수도 있다는 조건이 있기 때문에 격자판 안에서 움직이는 경우만 이동 할 수 있도록 앞선 코드에 조건만 부여해서 풀었다.

# 문제 4. 로봇의 이동

로봇은 다음 규칙에 따라 이동합니다.

1. 'G' 명령을 주면 보고 있는 방향으로 한 칸 이동합니다. 격자 밖으로 나가는 명령은 하지 않습니다.
2. 'R' 명령을 주면 오른쪽으로 90도 회전합니다.
3. 'L' 명령을 주면 왼쪽으로 90도 회전합니다.
   매개변수 moves에 로봇에 명령을 내린 문자들이 차례대로 나열된 명령 문자열이 주어지면 이 명령 문자열을 로봇이 모두 수행했을 때 최종 위치를 반환하는 프로그램을 작성하세요.

## 풀이

```python
def solution(moves: str) -> list[int]:
    init_cords = [0, 0]
    init_direction = 1
    dx = [-1, 0, 1, 0]
    dy = [0, 1, 0, -1]
    for move in moves:
        if move == "G":
            init_cords[0] += dx[init_direction]
            init_cords[1] += dy[init_direction]
        elif move == "R":
            init_direction = (init_direction + 1) % 4
        elif move == "L":
            init_direction = (init_direction - 1) % 4
    return init_cords
```

이번 문제 같은 경우에는 '90도 회전'이라는 조건이 추가 되었기 때문에 조금 더 다르게 생각해야할 필요성을 느꼈다.

`dx,dy` 인덱스를 설정하고 R과 L이 들어올 시 회전하는 효과를 주도록 초기 방향 변수를 설정했다.

그리고 인덱스 안에서 계속 회전 할 수 있도록 나머지 연산을 이용해 `out of range` 에러가 안나도록 했다.

# 문제 5. 위험 지역

n\*n 이차원 배열에 특정 지역의 지뢰정보가 지도로 주어집니다. 만약 아래와 같이 5\*5의 지도에 지뢰정보가 주어지면

위에 지도에서 1은 지뢰가 매설된 지역이고, 0은 빈땅입니다.
위에 지도에서 지뢰가 매설된 격자와 상하좌우 대각선으로 인접한 8개의 빈땅 격자를 위험지 역입니다. 위에 지도에서 위험지역은 총 14개입니다.
매개변수 board에 특정지역의 지뢰정보가 담겨진 지도가 주어지면 이 지역에 위험지역이 총 몇 개 있는지 반환하는 프로그램을 작성하세요.

## 풀이

```python
def solution(nums: list[list[int]]) -> int:
    is_danger_zone = {}
    dx = [-1, -1, -1, 0, 1, 1, 1, 0]
    dy = [-1, 0, 1, 1, 1, 0, -1, -1]
    answer = 0
    for x, _ in enumerate(nums):
        for y, _ in enumerate(nums[x]):
            is_danger_zone[(x, y)] = False

    for x, _ in enumerate(nums):
        for y, _ in enumerate(nums[x]):
            if nums[x][y] == 1:
                for i in range(8):
                    if (
                        x + dx[i] >= 0
                        and x + dx[i] < len(nums)
                        and y + dy[i] >= 0
                        and y + dy[i] < len(nums)
                    ):
                        if nums[x + dx[i]][y + dy[i]] != 1:
                            is_danger_zone[(x + dx[i], y + dy[i])] = True
    for _, value in is_danger_zone.items():
        if value:
            answer += 1
    return answer
```

나는 이 문제는 좀 이상하게 푼 것 같아서 다시 풀어볼 생각이다.. 일단 for 문을 3개 돌리는게 비효율적인것 같기 때문에...

일단 앞서 공부한 Hashtable을 이용해서 풀었다. 격자와 똑같은 배열의 boolean hashtable을 만들고, 1번 pond 문제와 비슷하게 순회하는 값이 1이면 인접한 8방향을 모두 `True`로 만들어준다.

이번에는 대각선까지 포함한 방향을 탐색해야 하기 때문에 `dx,dy`의 값을 8개를 만들었다.

이때 주의해야 할 점은, 해당하는 1의 위치는 `True`로 만들면 안되기 때문에 값을 검사하는 조건 절이 필요하다.

이렇게 만든 boolean hashtable의 `True`값을 전부 카운트 한다.

## 느낀점

사실 강의에서 추가적으로 풀어보라고 한 [(백준)14503번 로봇청소기](https://www.acmicpc.net/problem/14503)는 정답 코드를 보고서도 잘 이해가 가지 않아서 다시 풀어보려고 한다...

한없이 어려워지면 또 어려워지는게 Simulation 문제들이라 많은 연습이 필요할 듯 싶다.
