## check_feasibility(prob_info, solution)
### condition for feasibility
- if the solution contains valid routes for all ports (경로의 유효성)
- routes adhere to constraints such as node indices, edge validity, and being simple (제약을 잘 지켰는가)
- loading, unloading, and rehandling operations are performed correctly (선적, 하역, 재선적)
- demand requirements are satisfied at each port (각 항구에서의 수요 충족)

### prob_info
