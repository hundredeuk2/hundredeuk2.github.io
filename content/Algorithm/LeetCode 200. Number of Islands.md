---
title: LeetCode 200. Number of Islands
publish: true
date: 2026-04-20
tags:
  - dfs
  - bfs
---
## 문제 요약

2D 격자에서 `'1'`(육지)과 `'0'`(물)으로 이루어진 지도가 주어졌을 때, 섬의 개수를 구하는 문제다. 상하좌우로 연결된 `'1'`들이 하나의 섬이다.

---

## 첫 번째 시도 — DFS (재귀)

방문 배열을 따로 만들고 DFS로 연결된 육지를 전부 방문 처리하는 방식으로 짰다.

```python
class Solution:
    def dfs(self, y, x):
        dirs = [(0,1), (1,0), (0,-1), (-1,0)]
        for d in dirs:
            xp, yp = x + d[0], y + d[1]
            if self.maps[yp][xp]:  
                continue
            if 0 <= xp < len(self.grid[0]) and 0 <= yp < len(self.grid):
                self.maps[yp][xp] = True
                dfs(yp, xp)
            else:
                return

    def numIslands(self, grid: List[List[str]]) -> int:
        self.grid = grid
        self.maps = [[False] * len(grid[0])] * len(grid)  
        cnt = 0
        for i in range(len(self.grid)):
            for j in range(len(self.grid[0])):
                self.maps[i][j] = True   
                if not self.maps[i][j]:
                    if grid[i][j] == 1:
                        dfs(i, j)
                        cnt += 1
```

버그가 세 개였다.

**버그 1. 범위 체크 순서** 범위 확인 전에 `self.maps[yp][xp]`에 접근하면 `IndexError` 발생한다. 범위 체크가 먼저 와야 한다.

**버그 2. visited 배열 초기화 문제**

```python
self.maps = [[False] * len(grid[0])] * len(grid)
```

이렇게 하면 모든 행이 같은 리스트 객체를 참조한다. `self.maps[0][0] = True`를 하면 `self.maps[1][0]`, `self.maps[2][0]`... 전부 True로 바뀐다. list comprehension을 써야 한다.

```python
self.maps = [[False] * len(grid[0]) for _ in range(len(grid))]
```

**버그 3. visited 체크 순서**

```python
self.maps[i][j] = True   # 먼저 True로 만들고
if not self.maps[i][j]:  # 항상 False → dfs 절대 실행 안 됨
```

순서가 반대였다.

---

## 두 번째 시도 — BFS (visited 배열 분리)

버그를 수정하고 BFS로 바꿨다.

```python
from collections import deque

class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        rows, cols = len(grid), len(grid[0])
        visited = [[False] * cols for _ in range(rows)]
        cnt = 0

        def bfs(y, x):
            q = deque([(y, x)])
            visited[y][x] = True
            while q:
                cy, cx = q.popleft()
                for dy, dx in [(0,1),(1,0),(0,-1),(-1,0)]:
                    ny, nx = cy+dy, cx+dx
                    if 0 <= ny < rows and 0 <= nx < cols:
                        if not visited[ny][nx] and grid[ny][nx] == '1':
                            visited[ny][nx] = True
                            q.append((ny, nx))

        for i in range(rows):
            for j in range(cols):
                if not visited[i][j] and grid[i][j] == '1':
                    bfs(i, j)
                    cnt += 1
        return cnt
```

결과: **260ms, Beats 29.44%**

---

## 세 번째 시도 — BFS (grid 자체를 visited로)

`visited` 배열을 따로 안 만들고 방문한 `'1'`을 바로 `'0'`으로 바꾸는 방식으로 최적화했다.

```python
class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        rows, cols = len(grid), len(grid[0])
        cnt = 0

        def bfs(y, x):
            q = deque([(y, x)])
            grid[y][x] = '0'
            while q:
                cy, cx = q.popleft()
                for dy, dx in [(0,1),(1,0),(0,-1),(-1,0)]:
                    ny, nx = cy+dy, cx+dx
                    if 0 <= ny < rows and 0 <= nx < cols and grid[ny][nx] == '1':
                        grid[ny][nx] = '0'  # 큐에 넣을 때 바로 마킹
                        q.append((ny, nx))

        for i in range(rows):
            for j in range(cols):
                if grid[i][j] == '1':
                    bfs(i, j)
                    cnt += 1
        return cnt
```

결과: **238ms** — 조금 빨라졌지만 기대만큼은 아니었다.

---

## 최종 풀이 — DFS (list 스택)

`deque` 오버헤드를 없애고 `list` 스택으로 DFS를 구현했다. `list.pop()`은 O(1)이고 C레벨에서 더 빠르게 동작한다.

```python
class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        rows, cols = len(grid), len(grid[0])
        cnt = 0

        for i in range(rows):
            for j in range(cols):
                if grid[i][j] == '1':
                    stack = [(i, j)]
                    grid[i][j] = '0'
                    while stack:
                        y, x = stack.pop()
                        for dy, dx in ((0,1),(1,0),(0,-1),(-1,0)):
                            ny, nx = y+dy, x+dx
                            if 0 <= ny < rows and 0 <= nx < cols and grid[ny][nx] == '1':
                                grid[ny][nx] = '0'
                                stack.append((ny, nx))
                    cnt += 1
        return cnt
```

---

## 정리

| 방식                  | 속도    |         |
| ------------------- | ----- | ------- |
| BFS + visited 배열 분리 | 260ms |         |
| BFS + grid 자체 활용    | 238ms |         |
| DFS + list 스택       | 235ms | <- Best |

**이번에 얻은 교훈 두 가지**

> `[[False] * n] * m` 은 절대 쓰면 안 된다. 항상 list comprehension으로 초기화하자.

> 큐에 넣을 때 바로 마킹해야 중복 방지된다. 꺼낼 때 마킹하면 같은 셀이 큐에 중복으로 들어갈 수 있다.