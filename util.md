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
- 
