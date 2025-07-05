## comment
### condition for feasibility
- if the solution contains valid routes for all ports (경로의 유효성)
- routes adhere to constraints such as node indices, edge validity, and being simple (제약을 잘 지켰는가)
- loading, unloading, and rehandling operations are performed correctly (선적, 하역, 재선적)
- demand requirements are satisfied at each port (각 항구에서의 수요 충족)

### prob_info (dictionary type)
#### ❓dictionary type
- (key : value) 연관배열
  - key는 unique, value은 어떤 자료형이든지 가능
#### key
- N(int) : number of nodes(including the gate node 0)
- E(list of tuples) : list of valid undirected edges in the graph
- K(list of tuples) : list of demands, where each demand is represented as ((origin, destination), quantity)
- P(int) : number of ports
- F(int) : fixed cost for each route
- LB(float) : lower bound for the objective value

### solution (dictionary type)
- keys are port indices(0 to P-1), values are lists of routes
- each route is represented as a tuple(route, demand_index)
  - route : a list of node indices
  - demand_index : the index of the demand being handled
 
### dictionary contains..
- feasible(bool type) : feasible = True, infeasible = False
- obj(float, optional) : the total objective value of the solution(if it is feasible)
- infeasibility(list, optional) : reasons for infeasibility
- solution(dict) : the input solution

## check_feasibility(prob_info, solution)
```python
node_allocations = np.ones(N, dtype=int)*-1
```
- np.ones : NumPy funcition that creates a new array filled with '1'.
- '-1' : not must, -1 can be replaced to any number, but in this case '-1' to indicate the empty node.

```python
supposedly_loaded_demands_after_ports = {}
for p in range(P):
  supposedly_loaded_demands_after_ports[p] = {}
  for k, ((o, d), r) in enumerate(K):
    if o <= p < d:
      supposedly_loaded_demands_after_ports[p][k] = r
obj = 0
infeasibility = {}
```
- supposedly_loaded_demands_after_ports = {} : preparing a place to store “which demands are on board when leaving each port”
- loop
  - current port is p, total ports are P, o is the origin port, d is the destination port
  - if current port p is between o and d, we consider it remaining on board.

## def bfs
```python
def bfs(G, node_allocations, root=0):
    root = 0

    current_layer = [root]
    visited = set(current_layer)


    reachable_nodes = []
    reachable_node_distances = []

    dist = 0
    while current_layer:
        next_layer = []
        for node in current_layer:
            for child in G[node]:
                if child not in visited and node_allocations[child] == -1:
                    visited.add(child)
                    next_layer.append(child)
        current_layer = next_layer
        dist += 1
        reachable_nodes.extend(current_layer)
        reachable_node_distances.extend([dist] * len(current_layer))
    
    return reachable_nodes, reachable_node_distances
```
- purpose : to discover all unoccupied nodes in the graph that can be reached from the gate node, to measure how far away they are.
- collects two things for each layer of expansion:
  - which new nodes became reachable and are still free
  - how many edges away from the root each of them sits(distance)

## def get_available_nodes(node_allocations)
```python
def get_available_nodes(node_allocations):
    return [n for n,alloc in enumerate(node_allocations) if alloc == -1][1:] # # Excluding the gate node
```
- **ignores connectivity and distance**, just simply scans the allocation array and picks out every unassigned node
- return : flat list of all free nodes excluding gate node.
- usage : can simply find which node is free

## def dijkstra(G, node_allocations=None, start=0)
```python
def dijkstra(G, node_allocations=None, start=0):  
    distances = {node: float('inf') for node in G}
    distances[start] = 0

    previous_nodes = {node: None for node in G}

    priority_queue = [(0, start)]  # (distance, node)

    while priority_queue:
        current_distance, current_node = heapq.heappop(priority_queue)

        if current_distance > distances[current_node]:
            continue

        for neighbor in G[current_node]:
            if node_allocations is None or node_allocations[neighbor] == -1:
                distance = current_distance + 1

                if distance < distances[neighbor]:
                    distances[neighbor] = distance
                    previous_nodes[neighbor] = current_node
                    heapq.heappush(priority_queue, (distance, neighbor))

    return distances, previous_nodes
```
- function; **how does Dijkstra(or BFS) figure out which paths are actually usuable, given some nodes are blocked?**
>```
>if node_allocations is None or node_allocations[neighbor] == -1
>```
>Before stepping to neighbor nodes, this code functions. It simply never enqueues the node that's occupied(just becomes a wall we can't pass through).

## difference between BFS and Dijkstra
| 항목         | BFS                           | Dijkstra                  |
| ---------- | ----------------------------- | ------------------------- |
| **핵심 구조**  | FIFO 큐                        | 최소 힙 (priority queue)     |
| **시간 복잡도** | $O(N+E)$                      | $O((N+E)\log N)$          |
| **코드 분량**  | 간단 (큐·방문 체크만)                 | 힙·거리 테이블·이전 노드 테이블 필요     |
| **방문 순서**  | “거리 1 → 거리 2 → …” 레이어별 탐색     | 거리 오름차순 순차 탐색             |
| **추가 메모리** | 방문 집합(`visited`)              | 거리(`distances`), 이전 노드, 힙 |
| **확장성**    | 가중치 없는 그래프 전용                 | 가중치, 휴리스틱(A\*) 확장 용이      |
| **주요 용도**  | 단순 도달성, k-단계 노드 검색, 이분 그래프 검사 | 일반 최단 경로, 가중치/휴리스틱 적용     |

## def path_backtracking(previous_nodes, start, target)
```python
def path_backtracking(previous_nodes, start, target):
    path = []
    current_node = target
    while current_node is not None:
        path.append(current_node)
        current_node = previous_nodes[current_node]
    path.reverse()  
    return path
```
- purpose : return **path**
- for clarity, we split dijkstra(or bfs) and path_batracking each
- 
- 
