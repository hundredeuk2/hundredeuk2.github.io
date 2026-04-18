---
title: LeetCode 46. Permutations
date: 2026-04-19
publish: true
tags:
  - leetcode
  - backtracking
  - dfs
---
## 문제 요약

중복 없는 정수 배열 `nums`가 주어졌을 때, 가능한 모든 순열을 반환하는 문제다.

---

## 첫 번째 시도 — DFS + visited

방문 배열로 사용 여부를 체크하면서 DFS로 순열을 만드는 방식으로 짰다.

python

```python
class Solution(object):
    def dfs(self, visited, per_list):
        for i, num in enumerate(self.nums):
            if visited[i]:
                continue
            per_list.append(num)
            if len(per_list) == len(self.nums):
                self.answers.append(per_list)
                return
            else:
                return self.dfs(visited, per_list)

    def permute(self, nums):
        self.nums = nums
        self.answers = []
        visited = [False] * len(nums)

        for i, num in enumerate(nums):
            init_list = [num]
            visited[i] = True
            self.dfs(visited, init_list)
            visited[i] = False
        return self.answers
```

버그가 세 개였다.

**버그 1. `visited[i]`를 업데이트 안 함**

python

```python
per_list.append(num)
# visited[i] = True 가 없음!
```

dfs 안에서 방문 처리를 안 하니까 같은 숫자가 중복으로 들어간다.

**버그 2. 참조 저장 문제**

python

```python
self.answers.append(per_list)   # 참조를 저장
# 나중에 per_list가 바뀌면 저장된 값도 같이 바뀜!
self.answers.append(per_list[:])  # 복사본을 저장해야 함
```

**버그 3. return 위치가 잘못됨**

python

```python
else:
    return self.dfs(visited, per_list)  # 첫 번째 경우만 탐색하고 끝!
```

`return self.dfs()`를 하면 첫 번째 경우만 탐색하고 나머지를 포기한다. 백트래킹은 모든 경우를 탐색해야 한다.

---

## 두 번째 시도 — 백트래킹 패턴 적용

python

```python
class Solution:
    def dfs(self, visited, per_list):
        if len(per_list) == len(self.nums):
            self.answers.append(per_list[:])  # 복사본 저장
            return

        for i, num in enumerate(self.nums):
            if visited[i]:
                continue
            visited[i] = True         # 방문 처리
            per_list.append(num)
            self.dfs(visited, per_list)
            per_list.pop()            # 백트래킹
            visited[i] = False        # 방문 해제

    def permute(self, nums):
        self.nums = nums
        self.answers = []
        visited = [False] * len(nums)
        self.dfs(visited, [])
        return self.answers
```

통과했다.

---

## 최적화 — swap 방식

`visited` 배열을 없애고 swap으로만 처리하면 배열 접근 비용이 줄어든다.

python

```python
class Solution:
    def permute(self, nums):
        self.answers = []
        self.dfs(nums, 0)
        return self.answers

    def dfs(self, nums, start):
        if start == len(nums):
            self.answers.append(nums[:])
            return

        for i in range(start, len(nums)):
            nums[start], nums[i] = nums[i], nums[start]  # swap
            self.dfs(nums, start + 1)
            nums[start], nums[i] = nums[i], nums[start]  # 원복
```

---

## 시간복잡도

순열 자체가 n!개 존재하고, 각 순열을 저장할 때 길이 n을 복사해야 해서 어떤 방식을 써도 **O(n * n!) 이하로는 못 내려간다.** 결과물 자체가 n!개라서 그걸 만드는 시간보다 빠를 수 없기 때문이다.

|방식|시간복잡도|차이|
|---|---|---|
|visited 방식|O(n * n!)|visited 배열 접근 비용 존재|
|swap 방식|O(n * n!)|상수 최적화|

---

## 정리

**백트래킹 핵심 패턴**

```
선택 → visited[i] = True, append
탐색 → dfs()
취소 → pop(), visited[i] = False
```

> 백트래킹은 선택 → 탐색 → 취소 세 단계가 항상 세트다. 취소를 빠뜨리면 이전 상태로 못 돌아간다.

> `answers.append(per_list)`가 아니라 `answers.append(per_list[:])`로 복사본을 저장해야 한다. 참조를 저장하면 나중에 리스트가 바뀔 때 같이 바뀐다.