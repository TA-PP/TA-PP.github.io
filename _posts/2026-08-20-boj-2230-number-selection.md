---
title: "[백준 2230] 수 고르기 - Python 정렬과 투 포인터, 종료 조건 반례"
date: 2026-08-20 10:00:00 +0900
categories: [Coding Test, Baekjoon]
tags: [python, sorting, two-pointer, array, debugging, baekjoon]
---

## 문제 정의

[백준 2230번 수 고르기](https://www.acmicpc.net/problem/2230)는 정수 수열에서 두 수를 골랐을 때, 두 수의 차이가 `M` 이상인 경우 중 가장 작은 차이를 구하는 문제다.

```text
선택한 두 수: A, B
조건: |A - B| >= M
목표: 조건을 만족하는 |A - B|의 최솟값
```

문제에서 같은 수를 고르는 것도 허용한다. 따라서 `M = 0`이면 같은 위치의 값을 두 번 선택해 차이 `0`을 만들 수 있다.

주요 제한은 다음과 같다.

- `1 <= N <= 100,000`
- `0 <= M <= 2,000,000,000`
- `0 <= |A[i]| <= 1,000,000,000`
- 차이가 `M` 이상인 두 수를 항상 고를 수 있다.

예를 들어 수열이 `[1, 2, 3, 4, 5]`이고 `M = 3`이라면 차이가 3 이상인 쌍 중 `(1, 4)`, `(2, 5)`의 차이 3이 가장 작으므로 정답은 3이다.

## 접근 및 설계

두 수를 선택하는 모든 조합을 확인하면 `O(N²)`이 필요하다. `N`이 최대 100,000이므로 완전 탐색 대신 정렬과 투 포인터를 사용한다.

### 1. 수열 정렬

입력 수열을 오름차순으로 정렬한다.

```python
numbers.sort()
```

정렬하면 `left <= right`일 때 다음 관계가 성립한다.

```text
numbers[left] <= numbers[right]

두 수의 차이
= numbers[right] - numbers[left]
```

절댓값을 직접 계산하지 않아도 항상 0 이상의 차이를 얻을 수 있다.

### 2. 차이가 M보다 작은 경우

```python
if difference < M:
    right += 1
```

조건을 만족하려면 차이를 키워야 한다. 정렬된 배열에서 `right`를 오른쪽으로 옮기면 더 크거나 같은 값을 가리키므로 차이가 커질 수 있다.

### 3. 차이가 M 이상인 경우

```python
else:
    answer = min(answer, difference)
    left += 1
```

현재 차이는 정답 후보다. 후보를 저장한 뒤 더 작은 차이를 찾기 위해 `left`를 오른쪽으로 옮긴다. 왼쪽 값이 커지면 같은 `right`에 대한 차이가 작아진다.

### 처음 작성한 코드

처음에는 다음과 같이 `left = 0`, `right = 1`에서 시작하고 두 포인터가 서로 다를 때만 반복했다.

```python
left = 0
right = 1
answer = []

while left < right and right < N:
    temp = numbers[right] - numbers[left]

    if temp >= M:
        answer.append(temp)
        left += 1
    elif temp < M:
        right += 1

print(min(answer))
```

차이에 따른 포인터 이동 방향은 맞지만 반복문의 종료 조건 때문에 탐색이 너무 일찍 끝날 수 있다.

### 반례: left와 right가 만났다고 탐색이 끝난 것은 아니다

```text
N = 3, M = 6
numbers = [1, 10, 17]
```

처음 작성한 코드의 흐름은 다음과 같다.

```text
left = 0, right = 1
difference = 10 - 1 = 9

9 >= 6이므로 후보 9 저장
left를 1로 이동

left == right가 되어 while 종료
출력: 9
```

하지만 `(10, 17)`의 차이는 7이므로 실제 정답은 7이다. `left`와 `right`가 만났을 때 탐색 전체를 종료할 것이 아니라, 포인터 관계를 다시 맞춘 뒤 아직 확인하지 않은 오른쪽 구간을 계속 탐색해야 한다.

### 반례: M이 0이고 N이 1인 경우

문제는 같은 수를 고를 수 있으므로 다음 입력의 정답은 0이다.

```text
N = 1, M = 0
numbers = [5]
```

처음 코드에서는 `right = 1`이므로 반복문에 한 번도 진입하지 않는다. `answer`가 빈 리스트인 상태에서 `min(answer)`를 호출하면 `ValueError`도 발생한다.

이를 처리하려면 두 포인터를 모두 0에서 시작해야 한다.

```python
left = 0
right = 0
```

포인터가 엇갈리면 `right`를 `left` 위치로 맞춰 정렬된 순서를 유지한다.

```python
if left > right:
    right = left
```

## 사용한 자료구조

### 리스트(List)

입력되는 `N`개의 정수를 리스트에 저장한다.

```python
numbers = [int(input()) for _ in range(N)]
```

리스트는 `sort()`로 정렬할 수 있고, 두 포인터의 인덱스로 각 원소에 `O(1)`에 접근할 수 있다.

### 두 개의 인덱스 포인터

```python
left = 0
right = 0
```

- `left`: 현재 작은 수를 가리키는 포인터
- `right`: 현재 큰 수를 가리키는 포인터

두 포인터는 오른쪽으로만 이동한다. 이미 확인해 후보에서 제외한 구간으로 돌아가지 않기 때문에 정렬 이후 탐색을 선형 시간에 끝낼 수 있다.

### 최솟값 변수

처음 코드에서는 모든 후보를 리스트에 저장한 뒤 마지막에 `min()`을 호출했다.

```python
answer.append(temp)
print(min(answer))
```

하지만 필요한 값은 후보 전체가 아니라 최솟값 하나뿐이다. 따라서 무한대로 초기화한 변수 하나를 갱신하는 편이 적절하다.

```python
answer = float('inf')
answer = min(answer, difference)
```

이렇게 하면 후보 저장에 필요한 추가 공간을 `O(N)`에서 `O(1)`로 줄이고 빈 리스트에서 `min()`을 호출하는 문제도 없앨 수 있다.

## 사용한 알고리즘

### 정렬

수열을 오름차순으로 정렬해 포인터 이동에 따라 차이가 어떻게 바뀌는지 예측할 수 있게 만든다.

```text
정렬 전: [10, 1, 17]
정렬 후: [1, 10, 17]
```

정렬되지 않은 상태에서는 `right`를 이동했을 때 값이 커진다는 보장이 없으므로 투 포인터 이동 기준을 세울 수 없다.

### 투 포인터(Two Pointer)

정렬한 리스트에서 두 포인터가 가리키는 값의 차이에 따라 한쪽 포인터를 이동한다.

```python
while left < N and right < N:
    difference = numbers[right] - numbers[left]

    if difference < M:
        right += 1
    else:
        answer = min(answer, difference)
        left += 1

    if left > right:
        right = left
```

핵심은 다음 두 가지 단조성이다.

1. `left`를 고정하고 `right`를 증가시키면 차이는 작아지지 않는다.
2. `right`를 고정하고 `left`를 증가시키면 차이는 커지지 않는다.

이 성질 덕분에 차이가 작을 때는 `right`, 조건을 만족했을 때는 `left`만 이동하며 정답 후보의 경계를 찾을 수 있다.

### 왜 조건을 만족하면 left를 이동하는가

현재 차이가 `M` 이상이면 이미 유효한 후보를 찾은 상태다. 같은 `right`에서 차이를 더 작게 만들 수 있는 방법은 `numbers[left]`를 더 큰 값으로 바꾸는 것이다.

```text
numbers = [1, 4, 8, 12]
M = 6

left=0, right=2 -> 8-1=7, 조건 만족
left=1, right=2 -> 8-4=4, 조건 불만족
right=3         -> 12-4=8, 다시 조건 만족
```

조건 만족과 불만족의 경계를 따라 두 포인터를 이동하면서 `M` 이상인 가장 작은 차이를 찾는다.

### 차이가 정확히 M이면 조기 종료할 수 있다

정답은 `M` 이상이어야 하므로 차이가 정확히 `M`인 후보를 찾았다면 이보다 좋은 답은 존재하지 않는다. 다음과 같이 바로 반복문을 종료하는 최적화도 가능하다.

```python
if difference == M:
    answer = M
    break
```

필수 처리는 아니지만 최솟값의 하한을 발견했다는 의미를 코드로 표현할 수 있다.

## 시간복잡도

수열의 길이를 `N`이라고 하자.

- 입력 저장: `O(N)`
- 정렬: `O(N log N)`
- 투 포인터 탐색: `O(N)`
- 전체 시간복잡도: **`O(N log N)`**
- 입력 리스트를 포함한 공간복잡도: **`O(N)`**
- 투 포인터 탐색에 사용하는 별도 공간: **`O(1)`**

`left`와 `right`는 각각 최대 `N`번 오른쪽으로 이동하며 뒤로 돌아가지 않는다. 따라서 `while`문이 중첩 반복문처럼 보이지 않더라도 전체 포인터 이동 횟수는 `O(N)`이다.

처음 코드처럼 모든 유효한 차이를 `answer` 리스트에 저장하면 후보 개수만큼 최대 `O(N)`의 추가 공간이 필요하다. 최솟값 변수 하나만 갱신하면 이 추가 공간을 `O(1)`로 줄일 수 있다.

## 알고리즘 흐름(텍스트 다이어그램)

```text
[N, M과 수열 입력]
          |
          v
[수열 오름차순 정렬]
          |
          v
[left = 0, right = 0, answer = INF]
          |
          v
[left와 right가 모두 N 미만인가?] -- No --> [answer 출력]
          |
         Yes
          |
          v
[difference = numbers[right] - numbers[left]]
          |
          v
     [difference < M?]
       /           \
     Yes            No
      |              |
      v              v
 [right 증가]   [answer 최솟값 갱신]
                     |
                     v
                 [left 증가]
       \             /
        \           /
         v         v
       [left > right?]
          /      \
        Yes       No
         |         |
         v         |
   [right = left]   |
         \         /
          v       v
       [다음 차이 확인]
```

## 최종 제출 코드

```python
import sys

input = sys.stdin.readline

N, M = map(int, input().split())
numbers = [int(input()) for _ in range(N)]
numbers.sort()

left = 0
right = 0
answer = float('inf')

while left < N and right < N:
    difference = numbers[right] - numbers[left]

    if difference < M:
        right += 1
    else:
        answer = min(answer, difference)

        if answer == M:
            break

        left += 1

    if left > right:
        right = left

print(answer)
```

로컬 테스트에 사용한 다음 코드는 백준 제출 시 제거해야 한다.

```python
sys.stdin = open('./input/picking_num_input.txt', 'r')
```

## 오늘의 회고(실수한 점 포함)

정렬한 뒤 차이가 작으면 `right`를 이동하고, 차이가 조건을 만족하면 `left`를 이동한다는 투 포인터의 핵심 방향은 맞게 잡았다. 그러나 포인터가 만나는 상황을 탐색 종료로 해석하면서 아직 확인하지 않은 후보를 놓쳤다.

1. **`left == right`는 전체 탐색의 종료 조건이 아니다.**

   두 포인터가 만났더라도 오른쪽에 아직 확인하지 않은 원소가 남아 있을 수 있다. `[1, 10, 17]`, `M = 6` 반례에서 처음 코드는 9를 출력하지만 정답은 7이다.

2. **문제의 ‘같은 수일 수도 있다’는 조건을 놓치면 안 된다.**

   `M = 0`이면 같은 값을 선택한 차이 0이 최적이다. `N = 1`도 가능하므로 두 포인터를 모두 0에서 시작해야 한다.

3. **빈 후보 리스트는 런타임 에러를 만들 수 있다.**

   반복문이 실행되지 않으면 `min(answer)`에서 `ValueError`가 발생한다. 최솟값 변수 하나를 큰 값으로 초기화해 갱신하는 편이 안전하다.

4. **모든 후보를 저장할 필요가 없다.**

   문제에서 요구하는 것은 가장 작은 차이 하나다. 후보 리스트 대신 `answer = min(answer, difference)`로 즉시 갱신하면 코드가 단순해지고 추가 공간도 줄어든다.

5. **투 포인터에서는 포인터의 불변식을 확인해야 한다.**

   이 풀이에서는 항상 `left <= right`를 유지해야 `numbers[right] - numbers[left]`가 정렬된 두 수의 차이가 된다. `left`가 앞서면 `right = left`로 관계를 복구해야 한다.

6. **정답의 하한을 찾으면 종료할 수 있다.**

   조건이 차이 `M` 이상이므로 정확히 `M`을 찾았다면 그 값이 최적이다. 이는 단순한 성능 최적화이면서 정답 범위를 이해했다는 증거이기도 하다.

이번 문제를 통해 투 포인터는 이동 방향만 맞추는 것으로 끝나지 않고, **초기 위치, 반복 조건, 포인터가 만났을 때의 처리, 문제에서 허용하는 선택 범위**까지 함께 설계해야 한다는 점을 복기했다. 특히 작은 반례를 직접 만들어 포인터의 모든 상태를 추적하는 습관이 중요하다.
