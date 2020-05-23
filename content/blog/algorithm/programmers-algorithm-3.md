---
title: '[프로그래머스] 코테연습 뽀개기 3'
date: 2020-05-23 22:05:99
category: algorithm
draft: false
---

_매주 4~6개의 알고리즘 문제를 풀고 올리는 회고 포스팅입니다._

**기본 골자 알고리즘 : DP / Greedy**

<hr>

> ### 1. 정수 삼각형 - DP

처음에는 흠.. 어떻게 하지? 하고 단순하게
memoization을 늘 생각하고 substructure로 쪼개니까 이와 같은 생각이 떠올랐다.

`어떤 특정 위치에서의 최대 값 Sn = Sn-1에서 나에게 올 수 있는 것 중 max값 + an`

이런 식으로 이용하여 첫번째 라인부터 끝라인까지의 최대값들을 전부 구한 후, 마지막 라인의 최댓값을 리턴하면 되는구나!

모서리 부분만 바라보는 곳을 달리하여 풀어야한다는 걸 깨달았다.

그래서 코드를 짰다.

```python

def dp(triangle, mem, height):
    line = triangle[height]
    for i in range(len(line)):
        if i == 0:
            mem[height][i] = mem[height-1][i] + line[i]
        elif i == len(line)-1:
            mem[height][i] = mem[height-1][i-1] + line[i]
        else:
            mem[height][i] = max(mem[height-1][i-1], mem[height-1][i]) + line[i]

def solution(triangle):
    mem = [[-1 for i in range(len(t))] for t in triangle]
    mem[0][0] = triangle[0][0]
    for i in range(1, len(triangle)):
        dp(triangle, mem, i)
    return max(mem[len(triangle)-1])

```

생각보다 간단히 풀려서 다행이라고 생각이 들었다.

엄청나게 괴랄스럽고 간결한 코드를 봐서 첨부하려고 한다.

```python

solution = lambda t, l = []: max(l) if not t else solution(t[1:], [max(x,y)+z for x,y,z in zip([0]+l, l+[0], t[0])])

'''
설명충 설명 들어갑니다.
1 한 층씩 제거하며, 그 층에서 계산한 최대 이동거리 배열을 계산하여, 한 층을 제거한 traingle을 첫번째 input, 이동거리 배열을 두 번째 input으로 넣어줍니다.
1. 따라서 traingle이 없으면 제거할 층이 없으므로 최종 조건입니다.
2. [0] + l, l + [0] 을 이용하여 모서리 조건을 해결해줍니다.
'''
```

> ### 2. 등굣길 - DP

현재 위치에서의 최단 경로 = 왼쪽 위치에서의 최단경로 + 위쪽 위치에서의 최단경로

이를 이용해서 최단 경로를 세는 알고리즘을 구현할 수 있다.
초기 값은 처음 시작하는 부분을 1로 해주어 문제를 해결했다.
갈 수 없는 puddle은 inf 처리를 해주어 선택하지 않도록 해주었다.

```python
from pprint import pprint
from math import inf

mod_val = 1000000007

dx = [-1, 0]
dy = [0, -1]
def dp(mem, y, x):
    if mem[y][x] != 0:
        return mem[y][x]

    prev = [mem[y+dy[0]][x+dx[0]], mem[y+dy[1]][x+dx[1]]]
    for p in prev:
        if p != inf:
            mem[y][x] += p

    return mem[y][x]

def solution(m, n, puddles):
    mem = [[inf] * int(m+1)]
    for i in range(0, n):
        mem.append([inf] + [0 for _ in range(m)] + [inf])
    mem.append([inf] * int(m+1))

    mem[1][1] = 1

    for x, y in puddles:
        mem[y][x] = inf

    for i in range(1, n+1):
        for j in range(1, m+1):
            dp(mem, i, j)

    answer = mem[n][m]
    return answer % mod_val
```

> ### 3. 큰 수 만들기 - Greedy

생각보다 간단할 거라고 생각했는데, 너무 시간을 많이 잡아먹었다. 2시간이 지나도 풀지 못했다. 일단 생각했던 로직은 수의 경향성을 보는 것이었다. 수의 경향성이 작아짐 혹은 같음 -> 다시 커짐의 경향으로 가는 순간의 수를 빼주면 된다고 생각했고, 그렇게 구현해주니 정확성 테스트에서는 모두 통과를 했다. 하지만 효율성에서는 전부 시간초과가 났다.

자리수가 100만자리 이하였고, 내가 했던 방법은 k개만큼 해당 구현한 함수를 돌려주어 수를 빼는 로직이었다. 당연히 시간 초과가 날 수 밖에 없는 구조였다.

그래서 k를 인자로 받아서, k가 0이 될 때까지 위와 같은 실행을 해주는데, 특정 수를 뺀 자리에서 지속해서 경향성을 테스트하여 수를 빼주는 형태로 구현했다. 다시 처음부터 경향성을 검사하는 것을 막기 위해서였다. 이렇게 구현하니 효율성에서는 나름 해결이 되었지만, 엣지 케이스가 나오면서 실패가 뜨기 시작했다.

이유를 살펴보니, `41467328 5`라는 케이스에서 에러가 났다. 이 경우를 계속 돌리니 4를 빼야하는 부분에서 6을 빼는 것이었다. 해당 위치에서부터 다시 경향성을 구하는 방식이 잘못 된 것이었다. 그래서 앞에 값이 증가하는 경향을 보이기 시작하는 부분에서 값을 빼기를 시작해야겠다는 생각을 했는데, 이렇게 구현한 방법도 문제를 해결하지 못했다.

내가 최종적으로 짠 코드다. 정리할 시간이 없어서 코드가 정신이 없다.

```python
def greedy(number, k):
    increase = None
    idx = 0
    #     while idx < len(number)-1:
    #         if number[idx] > number[idx+1]:
    #             phase = 1
    #             idx += 1
    #             break
    #         elif number[idx] < number[idx+1]:
    #             phase = 2
    #             break
    #         idx += 1

    #     if phase == 2:
    #         number.pop(idx)
    #         return

    while k != 0 and idx < len(number) -1:
        if number[idx] > number[idx + 1]:
            if increase is None:
                increase = idx
        elif number[idx] < number[idx + 1]:
            number = number[:idx] + number[idx + 1:]
            # print(increase, number, k)
            if increase is not None:
                idx = increase
                increase = None
            k -= 1
            continue
        idx += 1

    return number[:-k] if k else number


def solution(number, k):
    # number_list = []
    # for n in number:
    #     number_list.append(int(n))
    number = greedy(number, k)
    return number
```

후에 풀이 방법을 살펴보니, 이건 스택을 이용해서 number의 길이만큼의 time complexity로 구할 수 있는 문제였다. 해결책을 보니 너무나도 신박해서 놀라울 따름이었다. 이런 생각은 정말 어떻게 하지..?

~~현재는 시간이 없어서 구현을 하지 않은채로 포스팅을 올릴 것이고, 온갖 시험들이 끝난 이후에 마저 올릴 예정이다.~~

```python

```

> ### 4. 구명보트 - Greedy

처음에는 잉 뭐지? 되게 간단한데? 하고 구현을 했는데, 보니까 한 구명보트에 `2명`만 탈 수 있다는 것을 보지 못했다. 그래서 다시 생각을 하고 구현을 했다.

들어오는 인자인 `people` 리스트를 정렬하고, 앞과 뒤에 포인터를 두어 맨 뒤의 값과 앞의 값을 더했을 때, limit 값보다 작으면 2명을 태워 각각의 index를 줄이거나 늘려나가고, 그렇지 않으면 뒤의 포인터만 줄여 나가는 식으로 구현했다.

잘 구현했다고 생각했는데 자꾸 실패가 떴는데, 같이 스터디를 하는 친구가 엣지 케이스를 잡아주었다. 무조건 뒤의 큰 값만 태우는 경우가 마지막 남은 사람에 대한 인원 추가를 안할 거라 생각했는데, 홀수일 경우 앞 포인터 증가와 뒷포인터 감소가 동시에 일어나면서 중간에 낀 사람에 대해서는 구명보트가 추가가 되지 않는다는 걸 알았다.

그래서 빠르게 고쳤다.

```python
def solution(people, limit):
    people = sorted(people)

    if len(people) == 1:
        return 1
    if len(people) == 2:
        return 2 if sum(people) > limit else 1

    answer, end = 0, 0
    i, j = 0, len(people)-1

    while i < j:
        smallest, biggest = people[i], people[j]
        if smallest + biggest <= limit:
            i += 1
            j -= 1
            end = 0
        else:
            j -= 1
            end = 1
        answer += 1

    if (i == j and end == 0) or end == 1:
        answer += 1

    return answer
```
