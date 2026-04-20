---
title: LeetCode 56. Merge Intervals
publish: true
date: 2026-04-20
tags:
  - array
  - sorting
  - interval
---
## 문제 요약

구간(interval) 배열이 주어졌을 때, 겹치는 구간들을 합쳐서 반환하는 문제다.

```
Input:  [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]
```

`[1,3]`과 `[2,6]`은 겹치므로 `[1,6]`으로 합친다.

---

## 첫 번째 시도 — 잘못된 겹침 조건

시작 시간 기준으로 정렬하고, 앞 구간의 끝점이 다음 구간 안에 포함되는지 `range()`로 체크했다.

```python
class Solution:
    def merge(self, intervals: List[List[int]]) -> List[List[int]]:
        new_intervals = sorted(intervals, key=lambda x: x[0])
        answers = []
        start_point = 0

        while True:
            if start_point >= len(intervals):
                break
            for idx in range(start_point+1, len(new_intervals)):
                if new_intervals[start_point][-1] in range(new_intervals[idx][0], new_intervals[idx][1]):
                    continue
                else:
                    answers.append([new_intervals[start_point][0], new_intervals[idx][1]])
                    start_point = idx + 1
                    break
        return answers
```

버그가 두 개였다.

**버그 1. 겹침 조건이 틀렸다**

`range()`로 앞 구간의 끝점이 다음 구간 안에 있는지 체크했는데, 이 방식은 `[1,3]`과 `[3,5]`처럼 끝점과 시작점이 같은 경우를 겹침으로 못 잡는다. 올바른 겹침 조건은 단순하다.

```python
앞 구간의 끝 >= 다음 구간의 시작
```

**버그 2. 끝점 갱신을 안 했다**

`[1,3]`과 `[2,6]`이 겹칠 때 합친 구간의 끝은 `max(3, 6) = 6`이어야 한다. 그런데 코드에서 끝점 갱신 없이 그냥 `continue`로 넘어가기 때문에 잘못된 값이 나온다.

---

## 최종 풀이

```python
class Solution:
    def merge(self, intervals: List[List[int]]) -> List[List[int]]:
        new_intervals = sorted(intervals, key=lambda x: x[0])
        answers = []

        for interval in new_intervals:
            # 비어있거나, 마지막 구간과 안 겹치면 그냥 추가
            if not answers or answers[-1][1] < interval[0]:
                answers.append(interval)
            else:
                # 겹치면 끝점을 더 큰 값으로 갱신
                answers[-1][1] = max(answers[-1][1], interval[1])

        return answers
```

핵심 아이디어는 **정렬 후 앞에서부터 순서대로 보면서 마지막으로 추가한 구간과만 비교**하는 것이다. 정렬이 되어 있으면 새 구간은 항상 지금까지 본 구간들 중 가장 오른쪽과만 비교하면 충분하다.

```
[1,3]  → answers: [[1,3]]
[2,6]  → 3 >= 2 겹침 → [[1,6]]
[8,10] → 6 < 8 안겹침 → [[1,6],[8,10]]
[15,18]→ 안겹침 → [[1,6],[8,10],[15,18]] ✓
```

---

## 엣지케이스

| 입력 | 출력 | 케이스 |
|------|------|--------|
| `[[1,4],[4,5]]` | `[[1,5]]` | 끝점 = 시작점 |
| `[[1,4],[2,3]]` | `[[1,4]]` | 완전히 포함되는 경우 |

완전히 포함되는 경우 `max()`를 쓰지 않으면 끝점이 작아지는 버그가 생긴다.

---

## 정리

**이번에 얻은 교훈**

> 구간 문제는 먼저 시작 시간으로 정렬하면 마지막 구간과만 비교하면 된다. 결과 배열의 마지막 원소를 직접 수정하는 패턴을 기억하자.

> 겹침 조건은 `앞 끝 >= 뒤 시작`, 합칠 때 끝점은 `max(앞 끝, 뒤 끝)`.