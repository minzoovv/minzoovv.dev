---
title: '[프로그래머스] 코테연습 뽀개기 1'
date: 2020-05-18 20:05:37
category: algorithm
draft: false
---

_매주 4~6개의 알고리즘 문제를 풀고 올리는 회고 포스팅입니다._

**기본 골자 알고리즘 : BFS / DFS**

<hr>

> ### 1. 타겟 넘버

+, -를 번갈아가면서 모두 생각해보면 되는 완전 탐색이어서 이 부분에 해결할 수 있는 실마리가 있다고 생각했다.

아래와 같이 구현했다.

```python

results = []
def dfs(numbers, visited, result):
    if visited == len(numbers):
        results.append(result)
        return
    minus_result = result - numbers[visited]
    plus_result = result + numbers[visited]
    dfs(numbers, visited+1, minus_result)
    dfs(numbers, visited+1, plus_result)

def solution(numbers, target):
    dfs(numbers, 0, 0)
    return results.count(target)
```

모든 result를 전부 생각해서 target과 같은 것의 개수를 세는 것이 비효율적인건지 모르겠다… 뭔가 그런 결과값을 모두 저장한 다음에 거기서 target과 일치하는 것을 세는 구조는 효율적으로 바꿀 수 있을 것 같다. 공간을 너무 많이 차지하는 안좋은 코드인 듯.

> ### 2. 네트워크

있는 모든 노드들의 연결성을 찾는 문제였다.
처음에는 아래와 같이 구현했다.

```python
def dfs(computers, network, number):
    if number in network:
        return network

    network.append(number)
    links = computers[number]
    next = []
    for i in range(len(links)):
        if links[i] == 1 and i not in network:
            next.append(i)

    if not next:
        return network

    for n in next:
        network.extend(dfs(computers, network, n))
    return list(set(network))

def solution(n, computers):
    visited = [0 for _ in range(n)]
    answer = 0

    while any(v == 0 for v in visited):
        number = visited.index(0)
        network = dfs(computers, [], number)
        for n in network:
            visited[n] = 1
        answer += 1
    return answer
```

초반 테스트 케이스들은 전부 맞았지만, 헤비한 연산이 필요해서인지, 뒤에서는 런타임에러나 시간 초과 발생. 아마도 연결성을 표시하는 부분에서 비효율이 많고 잘못되어서 에러가 난 것 같다는 생각이 든다.

저 extends 부분을 고치니 문제가 해결되었다. 처음에는 해당 node와 연결되어있지만, 방문하지 않은 노드들에 대해서 dfs를 진행시킬 때 방문했다는 것들을 담는 network를 extends 처리해서 중복 제거까지 해줘야겠다는 생각을 했는데, 쓸모없는 연산이라는 생각이 들었다.
1이 2,3이라는 노드에 연결되어있을 때, 어차피 dfs이므로 2, 3 순서대로 재귀호출을 해주고 network를 그 새로운 dfs의 network 값으로 갱신해주면 된다는 사실을 깨닫고, 이렇게 고쳐주었다.

```python
# 훨씬 깔끔하다
    for n in next:
        network = dfs(computers, network, n)
    return network
```

그렇게 고쳤더니 전부 성공! 이걸 현대카트 코테에서도 했으면 좋았을 텐데.

> ### 3. 단어 변환

생각보다 금방 풀었다. 각 단어를 1번의 변경으로 변경할 수 있는 경우로 그래프를 만들고, 후에 그 그래프를 이용해 dfs를 실시해주고, 진행이 될 때마다 count를 진행해주었다.

```python

 def bfs(link, start, dest):
    queue = []
    visited = []
    dist = {}

    dist[start] = 0
    queue.append(start)
    while queue:
        target = queue.pop(0)
        if target in visited:
            continue
        visited.append(target)

        for element in link[target]:
            if element not in dist:
                dist[element] = dist[target] + 1
            queue.append(element)

    return dist[dest]


def is_diff_only_one(word1, word2):
    diff = 0
    for i in range(len(word1)):
        if word1[i] != word2[i]:
            diff += 1

    return True if diff == 1 else False


def solution(begin, target, words):
    answer = 0
    if target not in words:
        return answer

    # make link
    link = {}
    for word in words + [begin]:
        link[word] = []
        for i in range(len(words)):
            if is_diff_only_one(word, words[i]):
                link[word].append(words[i])

    return bfs(link, begin, target)
```

bfs의 전형적인 알고리즘이었던 것 같다.

> ### 4. 여행경로

제일 짜증났던 문제… 부들부들…
처음에는 단순히 이름 빠른 순으로만 가면 되는 줄 알고 구현했는데 그렇게 구현하면 안되는 거였다.

```python
def dfs(tickets, idx, visited, route, route_list):
    ticket = tickets[idx]  # 해당 ticket의 정보를 가져옴
    visited[idx] = 1  # 해당 ticket의 사용 표시

    # 내가 다음 곳으로 갈 필요가 있나? 목적지를 다했나?
    next = []
    if all(v == 1 for v in visited):
        # 전부 ticket을 소진함
        route_list.append(route + [ticket[1]])

        return

        # 티켓을 소비하지 않았음, 그러면 갈 곳이 있는지 체크해야함
    for i in range(len(tickets)):
        # 다음에 갈 수 있는 곳을 전부 가져옴
        if ticket[1] == tickets[i][0] and visited[i] == 0:
            next.append(i)

    if next:
        # 갈 곳이 있음.
        for n in next:
            new_route = route[:] + [tickets[n][0]]
            dfs(tickets, n, visited, new_route, route_list)

    else:
        # 갈 곳이 없음
        visited[idx] = 0
        return


def solution(tickets):
    route_list = []

    for i in range(len(tickets)):
        visited = [0 for _ in range(len(tickets))]
        if tickets[i][0] == "ICN":
            dfs(tickets, i, visited, ["ICN"], route_list)

    # print(route_list)

    return route_list

# route list를 받아야하고,
# 해당 ticket를 전부 사용하는 시점에 해당 여태 왔던 route를 route list에 넣고 끝내야 한다
# 그렇지 않거나, 더이상 소비할 수 없는 상황이면 리턴하고 끝내야함
# 결국 남는건 route_list
# 해당 ticket를 사용했다는 것을 체크해줘야함, 해당 scope에 머물러 있어야 함.
# entry point가 모든 ICN에서 시작하는 ticket, 그리고 해당 ticket 다음에 갈 곳이 없으면 그냥 끝내야함

```

결국 test case 1번을 해결하지 못하고 그냥 정답을 봐버렸다.  
Test case 1번은 중복된 ticket에 대한 처리를 하지 못해서 에러가 나는 것이었다.

```python

from collections import defaultdict

def dfs(graph, N, key, footprint):
    print(footprint)
    # 모든 ticket을 다 사용했을 때 이렇게 그대로 return을 해준다.
    if len(footprint) == N + 1:
        return footprint

    for idx, country in enumerate(graph[key]):
        graph[key].pop(idx)

	      # 아예 새로운 footprint 선언. 나의 new_route와 동일함
        tmp = footprint[:]
        tmp.append(country)

        ret = dfs(graph, N, country, tmp)

        # 다시 해당 방문을 graph에 넣어준다. restore하는 방식
        graph[key].insert(idx, country)

        # return 값이 존재할 때만 진행해준다
        if ret:
            return ret

   # 해당 그래프가 갈 곳이 없으면 저절로 return None이 되어 무시가 된다.
def solution(tickets):
    answer = []
    # 신박한 defaultdict! 전부 list로 초기값이 초기화 된다. 완전 좋다.
    graph = defaultdict(list)

    N = len(tickets)
    for ticket in tickets:
        graph[ticket[0]].append(ticket[1])
        # 이렇게 그래프를 만들고 sorting해주었다. 내가 처음 시도했던 방법.
		  graph[ticket[0]].sort()

    answer = dfs(graph, N, "ICN", ["ICN"])

    return answer
```

이 알고리즘은 일단 ticket의 이름을 가장 우선해서 돌도록 하되, restore하는 방식을 생각을 못한 나와 달리 한 dfs가 끝난 후에 다시 보상 커맨드를 넣어서 ticket의 상태가 유지될 수 있도록 하였다.

어려웠던 부분은 route, ticket이 여러 function call에서 공유되는데, 그것이 모든 function call의 scope에 맞게 상태가 유지될 수 있도록 하는게 가장 어려웠다. 그런 부분을 잘 보여주는 코드인 것 같다.

내가 구현했던 부분에 있어서의 오류는 뭔지 확인을 다시 해봐야할 것 같다.
