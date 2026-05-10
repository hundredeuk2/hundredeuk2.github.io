---
title: LeetCode 3. Longest Substring Without Repeating Characters — 슬라이딩 윈도우의 정석
date: 2026-05-10
tags:
  - SlidingWindow
  - String
---

## 문제 요약

문자열 `s`가 주어졌을 때, **중복 문자 없는 가장 긴 부분 문자열의 길이**를 반환하는 문제.

```
Input:  "abcabcbb"
Output: 3  # "abc"

Input:  "bbbbb"
Output: 1  # "b"

Input:  "pwwkew"
Output: 3  # "wke"
```

---

## 첫 번째 접근 — O(n²) 브루트포스

처음엔 각 인덱스 `i`를 시작점으로, 중복이 나올 때까지 탐색하는 방식으로 풀었다.

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        cnt = 0
        i = 0
        while i < len(s):
            d = []
            for idx in range(i, len(s)):
                if s[idx] in d:
                    break
                d.append(s[idx])
            
            cnt = max(cnt, len(d))
            i += 1
            
        return cnt
```

- **시간복잡도**: O(n²) — 모든 시작점에서 다시 탐색
- **공간복잡도**: O(n)
- **문제점**: 중복이 발견되면 처음부터 다시 탐색. 이미 확인한 정보를 버린다.

---

## 핵심 인사이트 — 슬라이딩 윈도우

> **한 번 지나간 문자의 위치를 기억하면, 돌아갈 필요가 없다.**

브루트포스의 낭비 포인트는 중복이 발견될 때마다 `i`를 1씩만 밀고 전체를 재탐색하는 것.

슬라이딩 윈도우는 다르다:
- `left`, `right` 두 포인터로 현재 윈도우를 유지
- 각 문자의 **마지막으로 본 위치(index)** 를 `dict`에 저장
- 중복 문자 발견 시, `left`를 그 문자 바로 다음으로 **점프**

```
s = "abcabcbb"
       ^        right=3, 'a' 발견 — 이전 'a'는 index 0
                left = max(left, 0+1) = 1  →  윈도우: "bca"
```

단 한 번의 right 순회로 끝난다.

---

## 풀이 2 — O(n) 슬라이딩 윈도우

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        seen = {}   # 문자 → 마지막으로 본 index
        left = 0
        cnt = 0
        
        for right, c in enumerate(s):
            # 이미 본 문자가 현재 윈도우 안에 있다면
            if c in seen and seen[c] >= left:
                left = seen[c] + 1      # 윈도우 왼쪽을 점프
            
            seen[c] = right             # 현재 위치 갱신
            cnt = max(cnt, right - left + 1)
        
        return cnt
```

- **시간복잡도**: O(n) — right 포인터가 한 번만 순회
- **공간복잡도**: O(min(n, m)) — m은 문자 종류 수 (ASCII면 최대 128)

---

## 핵심 조건 — `seen[c] >= left`

`seen`에 문자가 있다고 해서 무조건 `left`를 옮기면 안 된다.
이미 윈도우 밖으로 나간 문자일 수 있기 때문.

```
s = "abba"
            right=3, 'a' 발견
            seen['a'] = 0  →  하지만 left는 이미 2
            0 < 2이므로 → 윈도우 밖 → left 그대로 유지
```

`seen[c] >= left` 조건이 없으면 `left`가 뒤로 돌아가는 버그 발생.

---

## 두 풀이 비교

| | 브루트포스 | 슬라이딩 윈도우 |
|---|---|---|
| 시간복잡도 | O(n²) | **O(n)** |
| 공간복잡도 | O(n) | O(min(n, m)) |
| 인터뷰 전략 | "먼저 떠오른 방법" | **최종 답변** |

---

## 인터뷰 팁

`set` 대신 `dict`를 쓰는 이유: set은 "있냐 없냐"만 알 수 있지만, dict는 **"마지막으로 어디서 봤냐"** 를 저장할 수 있어서 `left` 점프가 가능하다.

슬라이딩 윈도우 문제에서 자주 나오는 패턴:
- `left`, `right` 포인터 유지
- 조건 위반 시 `left`를 옮겨 윈도우 축소
- 매 스텝에서 최댓값/최솟값 갱신

---

> **한 줄 요약**: 중복 문자 위치를 dict로 기억하면, right 한 번 순회로 O(n) 해결.
