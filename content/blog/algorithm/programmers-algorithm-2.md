---
title: '[프로그래머스] 코테연습 뽀개기 2'
date: 2020-05-18 22:05:46
category: algorithm
draft: false
---

_매주 4~6개의 알고리즘 문제를 풀고 올리는 회고 포스팅입니다._

**기본 골자 알고리즘 : DP / Greedy**

<hr>

> ### 1. N으로 표현 - DP

와 이건 너무 어렵다. DP의 구성요소인 Optimal substructure와 Overlapped subproblem을 생각해야하는데
어떤 걸로 가닥을 잡아야 할지 감이 안잡힌다..

결국 결과가 원하는 건 5를 최대 몇번을 써야 해당 값이 나오느냐? 인데
해당 각 원하는 결과 number를 index로 하여 N의 사용횟수 최소값을 리턴해야하나..?

어떤 데이터를 추가해야 n의 최소값을 포착할 수 있는지 모르겠다.
일단 memoization할 수 있는 부분은 각 연산들의 결과 정도일 것 같다.

손도 못댈 것 같아서 결국 그냥 답을 봤다.

```python

def solution(N, number):
    S = [{N}]
    for i in range(2, 9):
        lst = [int(str(N)*i)]
        for X_i in range(0, int(i / 2)):
            for x in S[X_i]:
                for y in S[i - X_i - 2]:
                    lst.append(x + y)
                    lst.append(x - y)
                    lst.append(y - x)
                    lst.append(x * y)
                    if x != 0: lst.append(y // x)
                    if y != 0: lst.append(x // y)
        if number in set(lst):
            return i
        S.append(lst)
    return -1

```

결국 풀이 방식은, 모든 해당 개수를 사용할 때 의 결과값 -> 이를 기반으로 한 또 다른 연산 경우의 수를 모두 포함, 그 안에 내가 원하는 결과 값이 있는지 없는지 찾는 방식으로 진행되었다.

복잡했던 건 division by zero를 해결해주는 부분이었는데, 저렇게 x, y가 0일 때를 제외해주면 되는 듯… 저런 걸 생각을 못했지 왜…

> ### 2. 타일 장식물 - DP

피보나치 + 그를 이용한 둘레 구하기로 이루어진 문제. 점화식을 쉽게 유도할 수 있어서 간단히 구현할 수 있었던 코드. Bottom up 방식으로 구현하였다.

```python
def dp(mem, count):
    if count == 1:
        return mem[1][1]

    if mem[count][1] != 0:
        return mem[count][1]

    mem[count][0] = mem[count-1][0] + mem[count-2][0]
    mem[count][1] = dp(mem, count-1) + mem[count][0] * 2

    return mem[count][1]


def solution(N):
    mem = [[0, 0] for _ in range(N + 1)]
    mem[1] = [1, 4]
    mem[2][0] = 1
    for i in range(3, N+1):
        dp(mem, i)

    return mem[N][1]
```

> ### 3. 체육복 - Greedy

Greedy 알고리즘은 현재 최적의 결과를 고르고 해당 subproblem을 해결하는 알고리즘! 최적이 뭔지를 생각해야 한다.

일단, Lost -> reserve 겹치는 걸 통해서 reserve를 다시 만들었다.
그 후 최적의 선택을 찾아야하는데, 해당 reserve에 대해서 주변에 lost가 있는지 찾고, 그것이 두개라면 앞에 있는 것으로 할당해서 lost를 없애나갔다. 이렇게 하니까 subproblem이 하나가 됐다.

```python

def allocate(lost, reserve):
    '''allocate reserve to lost, return non allocator'''
    if not lost:
        return lost

    for r in reserve:
        candidate = {r+1, r-1}
        borrower = list(candidate & set(lost))
        if borrower:
            lost.remove(borrower[0])
    return lost

def solution(n, lost, reserve):
    # 선작업
    duplicate = set(lost) & set(reserve)
    lost = list(set(lost) ^ duplicate)
    reserve = list(set(reserve) ^ duplicate)

    non_allocator = allocate(lost, reserve)

    return n - len(non_allocator)
```

> ### 4. 조이스틱 - Greedy

일단 움직임을 파악하면 이렇게 두 가지가 된다.

- 일단 처음 커서에 무조건 위치하므로, 처음 커서의 값을 바꿔준다.  
  -> 이 생각을 좀 잘못해서 나중에 조금 멍청하게 헤맸다. 중복코드도 생겼다.
- 그 후 가장 가까운 커서로 이동하도록 해야한다. (항상 현재 위치에서 가장 가까운 커서로 이동해야한다)  
  -> 이를 구현하는게 조금은 짜증났는데, 현재 위치를 기반으로 양 옆을 하나씩 늘리면서 바꿔야할 알파벳이 있는지를 확인, 둘 중 하나라도 있다면 해당 위치로 이동, 둘다 바꿔야한다면 오른쪽으로 움직일 수 있도록 하여 구현을 했다.
- 해당 값의 커서의 중간값을 찾고, 커서를 위로 굴릴지, 아래로 굴릴지 정해야 한다.

아래 알고리즘이 맞긴 했는데, 오늘 질문하기에 올라오는 코드를 보니 뭔가 문제 자체에 오류가 있었던 것 같다. Joystick에서 커서를 움직이려 할 때의 알고리즘을 잘 구현했는지 모르겠다.

```python
class Joystick:
    def __init__(self, string):
        self.string = string
        self.result = ['A'] * len(string)
        self.cursor = 0
        self.counter = 0

    def count_move(self, alphabet):
        ascil_number = ord(alphabet)
        if ascil_number < 78:
            return ascil_number - ord('A')
        else:
            return ord('Z') - ascil_number + 1

    def move_cursor(self):
        c = 1
        nearest = -1

        while nearest == -1:
            forward = (self.cursor + c) % len(self.string)
            backward = self.cursor - c + len(self.string) if self.cursor - 1 < 0 else self.cursor - c
            # print(self.result, forward, backward, nearest)
            if forward == backward:
                if self.string[backward] != self.result[backward]:
                    self.cursor = backward
                    self.counter += c
                return
            if all(self.string[s] != self.result[s] for s in (forward, backward)):
                nearest = forward
                break
            elif self.string[backward] != self.result[backward]:
                nearest = backward
                break
            elif self.string[forward] != self.result[forward]:
                nearest = forward
                break
            c += 1

        self.cursor = nearest
        self.counter += c

    def change(self):
        s = self.string[self.cursor]
        if s != self.result[self.cursor]:
            self.counter += self.count_move(s)
            self.result[self.cursor] = s

        while ''.join(self.result) != self.string:
            self.move_cursor()
            s = self.string[self.cursor]
            self.counter += self.count_move(s)
            self.result[self.cursor] = s
        return self.counter


def solution(name):
    joystick = Joystick(name)
    answer = joystick.change()
    return answer

```
