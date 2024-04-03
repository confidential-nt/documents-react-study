사족: 어제 방금 막 여행에서 돌아와서 새 아티클을 진지하게 읽어볼 시간이 없었습니다..그래서 제게 많은 영감을 준 문제를 다시 한 번 복습하는 걸로 오늘 이 시간을 보내보려 합니다..

### 링크

https://school.programmers.co.kr/learn/courses/30/lessons/258711

그림은 링크 참고

### 문제

도넛 모양 그래프, 막대 모양 그래프, 8자 모양 그래프들이 있습니다. 이 그래프들은 1개 이상의 정점과, 정점들을 연결하는 단방향 간선으로 이루어져 있습니다.

크기가 n인 도넛 모양 그래프는 n개의 정점과 n개의 간선이 있습니다. 도넛 모양 그래프의 아무 한 정점에서 출발해 이용한 적 없는 간선을 계속 따라가면 나머지 n-1개의 정점들을 한 번씩 방문한 뒤 원래 출발했던 정점으로 돌아오게 됩니다. 도넛 모양 그래프의 형태는 다음과 같습니다.

<그림>

크기가 n인 막대 모양 그래프는 n개의 정점과 n-1개의 간선이 있습니다. 막대 모양 그래프는 임의의 한 정점에서 출발해 간선을 계속 따라가면 나머지 n-1개의 정점을 한 번씩 방문하게 되는 정점이 단 하나 존재합니다. 막대 모양 그래프의 형태는 다음과 같습니다.

<그림>

크기가 n인 8자 모양 그래프는 2n+1개의 정점과 2n+2개의 간선이 있습니다. 8자 모양 그래프는 크기가 동일한 2개의 도넛 모양 그래프에서 정점을 하나씩 골라 결합시킨 형태의 그래프입니다. 8자 모양 그래프의 형태는 다음과 같습니다.

<그림>

도넛 모양 그래프, 막대 모양 그래프, 8자 모양 그래프가 여러 개 있습니다. 이 그래프들과 무관한 정점을 하나 생성한 뒤, 각 도넛 모양 그래프, 막대 모양 그래프, 8자 모양 그래프의 임의의 정점 하나로 향하는 간선들을 연결했습니다.
그 후 각 정점에 서로 다른 번호를 매겼습니다.
이때 당신은 그래프의 간선 정보가 주어지면 생성한 정점의 번호와 정점을 생성하기 전 도넛 모양 그래프의 수, 막대 모양 그래프의 수, 8자 모양 그래프의 수를 구해야 합니다.

그래프의 간선 정보를 담은 2차원 정수 배열 edges가 매개변수로 주어집니다. 이때, 생성한 정점의 번호, 도넛 모양 그래프의 수, 막대 모양 그래프의 수, 8자 모양 그래프의 수를 순서대로 1차원 정수 배열에 담아 return 하도록 solution 함수를 완성해 주세요.

### 제한사항

1 ≤ edges의 길이 ≤ 1,000,000

edges의 원소는 [a,b] 형태이며, a번 정점에서 b번 정점으로 향하는 간선이 있다는 것을 나타냅니다.

1 ≤ a, b ≤ 1,000,000

문제의 조건에 맞는 그래프가 주어집니다.

도넛 모양 그래프, 막대 모양 그래프, 8자 모양 그래프의 수의 합은 2이상입니다.

### 접근 방법

#### 문제 재정의

어떤 그래프가 있는데, 이 그래프는 막대 그래프, 도넛 그래프, 8자 모양 그래프 + 이들 그래프 각각의 임의의 정점 하나로 간선을 뻗고 있는 '생성된 정점'으로 이루어져 있다.

그래프에 대한 정보가 입력으로 주어질 때, 생성한 정점의 번호, 도넛 모양 그래프의 수, 막대 모양 그래프의 수, 8자 모양 그래프의 수를 순서대로 1차원 배열에 담아 리턴하라.

#### 전략

1. 사실 이 문제를 처음부터 혼자의 힘으로 풀 생각은 없었다. 왜냐하면 실제로 나는 이 코테를 작년에 봤었고, 그림 + 그래프의 콜라보로 지레 겁을 먹고 풀지 않았던 문제였기 때문이다. 따라서 어느 정도 문제의 큰 틀만 이해하고 카카오 기술 블로그에 올라와져 있는 문제 풀이를 보기로 결심했다. 나의 힘으로 파악한 것은 도넛 그래프는 순환하는 모양을, 막대 그래프는 한쪽으로만 가는 모양을, 8자 그래프는 도넛 그래프를 합친 형태를 띄고 있다는 것. 그리고 '생성된 정점'은 다른 모든 그래프의 적어도 하나의 정점과 연결되어있다는 것. 대략 여기까지 였다. 그리고 카카오의 블로그로 가 문제 풀이를 보았고 내가 뭘 놓치고 있었는지를 깨달았다.

[해당 블로그](https://tech.kakao.com/2023/12/27/2024-coding-test-winter-internship/)

그림, 혹은 복잡해보이는 그래프라고 지레 겁먹지 말고, 각 그래프를 구분할 수 있는 핵심적 특징을 구분하면 문제는 풀린다는 것이다.

> 생성된 정점: 나가는 간선이 2개 이상(그래프의 개수) 존재하고, 들어오는 간선이 존재하지 않습니다.
> 막대 모양 그래프: 생성된 정점과 연결된 간선을 제외했을 때, 들어오는 간선이 없는 정점이 하나 존재하고, 나가는 간선이 없는 정점이 하나 존재합니다(두 정점은 같을 수 있습니다). 나머지 정점은 나가는 간선과 들어오는 간선이 하나씩 존재합니다.
> 도넛 모양 그래프: 생성된 정점과 연결된 간선을 제외했을 때, 모든 정점이 나가는 간선과 들어오는 간선이 하나씩 존재합니다.
> 8자 모양 그래프: 생성된 정점과 연결된 간선을 제외했을 때, 1개의 정점을 제외하면 나가는 간선과 들어오는 간선이 하나씩 존재합니다. 1개의 정점은 나가는 간선 2개와 들어오는 간선 2개가 존재합니다.

위 특징을 간파하면 문제는 풀린다는 것을 깨달았다.

#### 계획 검증

1에 대한 검증 -> 그러면 탐색을 해서 어디서 어떻게 어떤 형태로 뻗어있는지를 알아내야 할텐데, 나의 경우 bfs를 사용해서 문제를 풀려고 시도를 해보았다. 잘만 구현한다면 시간 초과는 나지 않을 것으로 생각했다. ~~(실패했지만..)~~

#### 구현

```python
from collections import defaultdict, deque

def solution(edges):
    graph = defaultdict(dict)
    for edge in edges:
        start = edge[0]
        end = edge[1]
        init_graph(start, end, graph)
        graph[start]["next"].append(end)
        graph[start]["out"] += 1
        graph[end]["in"] += 1

    generated_vertex = find_generated_vertex(graph)
    number_of_graph = len(graph[generated_vertex]["next"])
    eliminate_edge_from_generated_vertex(graph, generated_vertex)
    stick_graph_count = count_stick_graph(graph, generated_vertex)
    eight_graph_count = count_eight_graph(graph, generated_vertex)
    donut_graph_count = number_of_graph - (stick_graph_count + eight_graph_count)
    return [generated_vertex, donut_graph_count, stick_graph_count, eight_graph_count]

def init_graph(start, end, graph):
    if not "next" in graph[start]:
        graph[start]["next"] = []
    if not "in" in graph[start]:
        graph[start]["in"] = 0
    if not "out" in graph[start]:
        graph[start]["out"] = 0
    if not "next" in graph[end]:
        graph[end]["next"] = []
    if not "in" in graph[end]:
        graph[end]["in"] = 0
    if not "out" in graph[end]:
        graph[end]["out"] = 0


def find_generated_vertex(graph):
    queue = deque()
    visited = [False] * (len(graph.keys()) + 1)
    start = list(graph.keys())[0]
    queue.append(start)
    visited[start] = True

    while queue:
        node = queue.popleft()
        if graph[node]["in"] == 0 and graph[node]["out"] >= 2:
            return node
        for next_node in graph[node]["next"]:
            if not visited[next_node]:
                queue.append(next_node)
                visited[next_node] = True

def eliminate_edge_from_generated_vertex(graph, generated_vertex):
    edge_from_generated_vertex = graph[generated_vertex]["next"]
    for edge in edge_from_generated_vertex:
        graph[generated_vertex]["out"] -= 1
        graph[edge]["in"] -= 1

def count_stick_graph(graph, generated_vertex):
    queue = deque()
    visited = [False] * (len(graph.keys()) + 1)
    start = generated_vertex
    queue.append(start)
    visited[start] = True
    count = 0
    while queue:
        node = queue.popleft()
        if node != generated_vertex and (graph[node]["in"] == 0 or graph[node]["out"] == 0):
            count += 1
        for next_node in graph[node]["next"]:
            if not visited[next_node]:
                queue.append(next_node)
                visited[next_node] = True
    return count


def count_eight_graph(graph, generated_vertex):
    queue = deque()
    visited = [False] * (len(graph.keys()) + 1)
    start = generated_vertex
    queue.append(start)
    visited[start] = True
    count = 0
    while queue:
        node = queue.popleft()
        if node != generated_vertex and graph[node]["in"] == 2 and graph[node]["out"] == 2:
            count += 1
        for next_node in graph[node]["next"]:
            if not visited[next_node]:
                queue.append(next_node)
                visited[next_node] = True
    return count
```

#### 트러블 슈팅

입출력 예제에 대해서는 맞게 동작했지만 막상 코드를 제출하니 오답처리되는 경우도 있었고, 런타임 에러가 나는 경우도 있었다. 런타임에러는 아마 find_generated_vertex가 None을 리턴하는 경우가 있기 때문에 발생하는 것이라 추측하고 있다. 그리고 오답 처리 관련해서는, 일부 이어지지 않는 그래프 때문인 것 같았다.

그래서 탐색을 더 정확하게 하려고 수정을 시도했으나, 시간초과가 나는 경우도 있고 해서 다른 결정을 해야했다.

나는 더 이상 시간을 지체할 수 없었고 다른 사람이 작성한 힌트를 보았다.

[힌트](https://school.programmers.co.kr/questions/71669)

허를 찔렸다. 결국 중요한 것은, 생성된 정점은 들어오는 간선은 없고 나오는 간선은 2개 이상(문제에서 그래프의 수가 2개이상이라고 했으므로)이라는 것이고, 막대 그래프의 경우 나가는 간선이 0인 경우만 신경쓰면 되는 것이며, 8자 그래프는 나가는 간선이 2인 경우만 신경쓰면 되는 것이다. 왜냐하면 이것은 이들 각각의 구분되는 특징이기 때문에 충분히 그래프를 구분해낼 수 있기 때문이다. 들어오는 간선에 관해서는, 생성된 정점에 의해 들어오는 간선은 어쩔 수 없이 +1이 될 수 밖에 없는데 이 경우는 무시해도 된다. 왜냐하면 나가는 간선만 보면 되기 때문이다.

그리고 나가는 간선만 어떤지 보면 되기 때문에, 굳이 bfs같은 탐색을 사용할 필요 없이 루프만 간단하게 돌면 끝이다.

### 코드

```python
def solution(edges):
    n = 0
    # [정점, 도넛, 막대, 8자]
    answer = [0, 0, 0, 0]
    total = 0

	 # n을 이렇게 구해내야 겠다는 생각은 하지 못했던 것 같다..
    for _, [a, b] in enumerate(edges):
        max_node = max(a, b)
        n = max(max_node, n)

  # meta [노드에 들어오는 간선의 개수, 노드로부터 나가는 간선의 개수]
    meta = [[0, 0] for _ in range(n + 1)]
    graph = [[] for _ in range(n + 1)]

    for _, [a, b] in enumerate(edges):
        graph[a].append(b)
        meta[b][0] += 1
        meta[a][1] += 1

    for i, [come, out] in enumerate(meta):
        if i == 0: # 0 스킵 -> 스킵하지 않으면 정확하지 않은 결과가 도출된다.
            continue

        if out == 0:
            answer[2] += 1
        elif out == 1:
            continue
        elif out == 2:
            if come > 0 :
                answer[3] += 1
            else:
                answer[0] = i
                total = len(graph[i])
        elif out > 2:
            answer[0] = i
            total = len(graph[i])


    answer[1] = total - answer[2] - answer[3]
    return answer

```

### 마치며

답을 구하기 위해 가장 핵심이 되는 포인트를, 복잡해보이는 거 다 치우고 어떻게 찾을 것이냐에 관해 깊게 생각해볼수 있게 하는 문제였다. 그리고 항상 생각하는 것이지만, 특정 방법론에 매몰되지 말고, 가장 원하는 답을 쉽고 빠르게 도출해낼 수 있는 알고리즘을 생각해내는 것이 중요한 것 같다. 자료구조도 웬만하면 간단하고 명확한 형태로 생각해내는 것이 좋은 것 같다.
