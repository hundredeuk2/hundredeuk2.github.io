---
title: LeetCode 2. Add Two Numbers
date: 2026-04-27
publish: true
tags:
  - leetcode
  - linkedlist
  - medium
---
## 문제 요약
 
역순으로 저장된 두 LinkedList가 주어졌을 때, 두 수를 더한 결과를 역순 LinkedList로 반환하는 문제다.
 
```
Input:  l1 = [2,4,3], l2 = [5,6,4]
Output: [7,0,8]
설명: 342 + 465 = 807
```
 
---
 
## 첫 번째 시도
 
carry를 들고 순회하면서 합산하는 방식으로 짰다.
 
```python
class Solution:
    def addTwoNumbers(self, l1, l2):
        dummy = 0
        while True:
            if not l1.next and not l2.next:
                break
            tmp1 = l1.val if l1.val else 0
            tmp2 = l2.val if l2.val else 0
            sum_tmp = tmp1 + tmp2 + dummy
 
            if sum_tmp >= 10:
                dummy = 1
                sum_tmp -= 10
```
 
버그가 세 개였다.
 
**버그 1. 결과를 어떻게 리턴할지 막힘**
 
LinkedList 결과를 배열에 담으려 했는데, 반환 타입이 `ListNode`다. 순회하면서 새 노드를 직접 만들어 연결해야 한다.
 
**버그 2. `dummy` 변수명 혼동**
 
`dummy`를 carry(올림수) 용도로 썼는데, LinkedList 문제에서 `dummy`는 결과 리스트의 가짜 시작 노드를 뜻하는 게 관례다. 역할별로 이름을 분리해야 한다.
 
**버그 3. 종료 조건이 잘못됨**
 
```python
if not l1.next and not l2.next:
    break
```
 
두 리스트 길이가 다르면 짧은 쪽이 먼저 `None`이 돼서 `.next` 접근 시 터진다. 또 carry가 남아있어도 강제 종료된다.
 
**버그 4. `l1.val if l1.val else 0` 오류**
 
val이 0이면 falsy라서 0을 0으로 처리 못 한다. `l1.val if l1 else 0`이 맞다.
 
---
 
## 두 번째 시도 — dummy_head 패턴 적용
 
```python
class Solution:
    def addTwoNumbers(self, l1, l2):
        dummy_head = ListNode(0)   # 결과 리스트의 가짜 시작 노드
        current = dummy_head
        carry = 0
 
        while l1 or l2 or carry:
            val1 = l1.val if l1 else 0
            val2 = l2.val if l2 else 0
 
            total = val1 + val2 + carry
            carry = total // 10
 
            current.next = ListNode(total % 10)
            current = current.next
 
            if l1: l1 = l1.next
            if l2: l2 = l2.next
 
        return dummy_head.next
```
 
통과했다.
 
---
 
## 시간복잡도
 
| 항목 | 복잡도 |
|---|---|
| 시간 | O(max(N, M)) — 두 리스트 중 긴 쪽 기준 |
| 공간 | O(max(N, M)) — 결과 리스트 길이 |
 
---
 
## 정리
 
**dummy_head 패턴**
 
```
dummy_head = ListNode(0)   # 가짜 시작 노드
current = dummy_head
# 순회하면서 current.next에 노드 붙이기
return dummy_head.next     # 가짜 노드 다음부터 리턴
```
 
LinkedList 결과를 만들 때 거의 항상 쓰는 패턴이다. 없으면 첫 번째 노드를 따로 처리해야 해서 코드가 지저분해진다.
 
**while 조건**
 
```python
while l1 or l2 or carry:
```
 
`carry`를 조건에 넣는 게 포인트다. 999 + 1 = 1000처럼 마지막에 carry가 남는 케이스를 처리해준다.
 
> `l1.val if l1.val else 0`이 아니라 `l1.val if l1 else 0`이다. val이 0이면 falsy라서 틀린다.
 
> LinkedList 문제에서 `dummy`는 carry가 아니라 가짜 시작 노드 이름으로 쓰는 게 관례다.