---
title: (정리) HashTable
date: 2023-09-21 20:01:05 +/-TTTT
categories: [TIL, CodePractice]
tags: [python, codepractice, algorithm]
math: true
mermaid: true # TAG names should always be lowercase
---

# Hash Table

## 정의

Hash Table은 Hash Function을 거친 값들을 index로 삼아 key, value와 함께 저장하는 자료구조이다.

## Direct Hash Table

Direct Hash Table이란 키의 개수만큼의 크기를 가진 Hash Table을 말한다.

- 장점: 키 값이 서로 같아 충돌하는 문제가 생기지 않는다.
- 단점: 사용하지 않는 값들도 테이블이 만들어져야 하므로 메모리의 낭비가 심하다.

## 충돌

대게 Hash Table은 Hash 값보다 작은 크기를 가지므로 필연적으로 충돌 문제가 생긴다.

### 해결방법

- Seperate Chaining 기법
  - value 값을 Linked List로 연결하여 한 버킷당 들어갈 수 있는 값들을 제한하지 않으면서 값들을 넣는 방법
  - 이때 특정 값을 찾는데 O(n) 만큼의 시간이 걸린다.
- Open Addressing 기법
  - Linear Probing : 특정 키가 중복되면 해당 Hash값에서 고정폭만큼 옮겨 다음 Hash값에 값을 넣는것
  - Quadratic probing : 위의 방식에서 고정폭을 제곱수만큼 늘리는 방식. 1^2, 2^2, 3^2.... 와 같은 순으로 고정폭을 늘린다.
  - Double hashing : 충돌이 일어나는 것을 방지하기 위해 또 다른 Hash Function을 도입하여 하나는 값의 Hash를 찾기 위해, 또 하나는 고정폭을 얻는 방식으로 문제를 해결한다.

### 시간복잡도

Hash Table은 탐색, 삽입, 삭제에 있어서 _O(1)_ 만큼의 시간을 가진다.

하지만 충돌이 빈번하게 일어나 최악의 경우에는 _O(n)_ 만큼의 시간을 가질 수도 있다.

---

## 문제 1.

매개변수 nums에 길이가 n인 수열이 주어지면 수열의 원소중에서 빈도수가 1인 가장 큰 숫자 를 반환하는 프로그램을 작성하세요. 빈도수 1인 숫자가 없을 경우 -1를 반환하세요.

### 풀이 1.

```python
def solution(nums: list[int]) -> int:
direct_hash = [0] * 1001
max_val = -1
for i in nums:
    direct_hash[i] += 1
for index, cnt in enumerate(direct_hash):
    if cnt == 1 and index > max_val:
        max_val = index
return max_val
```

Direct Hash Table로 풀어본 문제이다. 적당한 범위의 값들이 제공된다면 괜찮겠지만, 큰 수가 주어질경우 메모리에서 심각한 낭비를 보게 될것이다.

### 풀이 2.

```python
def solution(nums: list[int]) -> int:
    frequency_table = {}
    for n in nums:
        if n not in frequency_table:
            frequency_table[n] = 1
        else:
            frequency_table[n] += 1

    max_value = -1
    for key, val in frequency_table.items():
        if val == 1:
            max_value = max(key, max_value)
    return max_value
```

Dictionary 자료구조를 이용해서 풀어보았다.

## 문제 2.

자기 분열수란 배열의 원소 중 자기 자신의 숫자만큼 빈도수를 갖는 숫자를 의미합니다. 만약 배열이 [1, 2, 3, 1, 3, 3, 2, 4] 라면
1의 빈도수는 2,
2의 빈도수는 2,
3의 빈도수는 3,
4의 빈도수는 1입니다.
여기서 자기 자신의 숫자와 같은 빈도수를 갖는 자기 분열수는 2와 3입니다.
매개변수 nums에 자연수가 원소인 배열이 주어지면 이 배열에서 자기 분열수 중 가장 작은 수를 찾아 반환하는 프로그램을 작성하세요. 자기 분열수가 존재하지 않으면 -1를 반환하세요.

### 풀이

```python
def solution(nums: list[int]) -> int:
    frequency_table = {}
    for num in nums:
        if num not in frequency_table:
            frequency_table[num] = 1
        else:
            frequency_table[num] += 1
    for key, value in frequency_table.items():
        if key == value:
            return key
    return -1
```

위의 문제와 조건만 달라졌다 뿐이지 거의 코드가 동일하다.

## 느낀점

코테에 점점 스며들고 있다....
