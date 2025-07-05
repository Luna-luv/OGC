## def loading_heuristic(p, node_allocations, rehandling_demands)
```python
        # All demands that should be loaded at port p
        K_load = {idx: r for idx, ((o,d),r) in enumerate(K) if o == p}

        print(f'Demands that are originated from this port: {K_load}')

        if len(rehandling_demands) > 0:
            print(f'Rehandled demands from unloading phase: {rehandling_demands}')
            # Merge the rehandling demands with the loading demands
            for k in rehandling_demands:
                if k in K_load:
                    K_load[k] += 1
                else:
                    K_load[k] = 1

        route_list = []

        last_rehandling_demands = []


        # Total number of demands to load (including rehandling demands)
        total_loading_demands = sum([r for k,r in K_load.items()])
```
- total loading demands = origin loading demands + rehandling demands
        # Get reachable nodes from the gate
        reachable_nodes, reachable_node_distances = util.bfs(G, node_allocations)

        # Get not occupied nodes
        available_nodes = util.get_available_nodes(node_allocations)

        if len(available_nodes) < total_loading_demands:
            print(f"Not enough available nodes ({len(available_nodes)} are available) to load {total_loading_demands} at port {p}.")
            raise Exception("Not enough available nodes to load demand!!! This must not happen!!!")

        if len(reachable_nodes) < total_loading_demands:
            print(f"Not enough reachable nodes ({len(reachable_nodes)} are reachable) to load {total_loading_demands} demands at port {p}. Rehandling...")

            # A very simple rehandling heuristic
            # 1. Get nodes that are available but not reachable
            # 2. Loop until we have enough reachable nodes
            # 2-1. Pick a node from the available but not reachable nodes
            # 2-2. Get the shortest path to the node from the gate
            # 2-3. Roll-off demands occupied on the path by order of distance from the gate. (and push to rehandling_demands stack for the later reloading)
            # 2-4. Check if the number of reachable nodes is enough to load the demand
            available_but_not_reachable = [n for n in available_nodes if n not in reachable_nodes]

            while len(reachable_nodes) < total_loading_demands:
                
                if len(available_but_not_reachable) == 0:
                    print("Not enough available_but_not_reachable nodes to rehandle. This must not happen!!!")
                    raise Exception("Not enough available_but_not_reachable nodes to rehandle. This must not happen!!!")
                
                # Pick a node from the available but not reachable nodes
                n = available_but_not_reachable.pop(0)

                # Get the shortest path to the node from the gate
                distances, previous_nodes = util.dijkstra(G, node_allocations=None)

                path = util.path_backtracking(previous_nodes, 0, n)

                # Roll-off blocking demands on the path by order of distance from the gate. (and push to last_rehandling_demands list for the later reloading)
                for idx, i in enumerate(path[:-1]):
                    if node_allocations[i] != -1:
                        # Node i is occupied by demand node_allocations[i]
                        last_rehandling_demands.append(node_allocations[i])
                        node_allocations[i] = -1

                        # Rehandling route (from the blocking node to the gate)
                        route_list.append((path[:idx+1][::-1], node_allocations[i]))

                        # Increasing the number demands to load
                        total_loading_demands += 1

                # Check if the number of reachable nodes is enough to load the demand
                reachable_nodes, reachable_node_distances = util.bfs(G, node_allocations)

            print(f'After rehandling, we have {len(reachable_nodes)} reachable nodes now! (but have to rehandle {len(last_rehandling_demands)} demands)')


        # Merge the rehandling demands with the loading demands
        for k in last_rehandling_demands:
            if k in K_load:
                K_load[k] += 1
            else:
                K_load[k] = 1

        print(f'total_loading_demands = {total_loading_demands}')

        if total_loading_demands > 0:

            # We take the fartest nodes from the gate to load the demands
            loading_nodes = reachable_nodes[-total_loading_demands:][::-1]

            # Sort the loading demands by destination ports
            sorted_K_load = sorted(K_load.items(), key=lambda x: K[x[0]][0][1], reverse=True)

            # Flatten the loading demands
            flattened_K_load = []
            for k, r in sorted_K_load:
                for _ in range(r):
                    flattened_K_load.append(k)

            assert(len(flattened_K_load) == len(loading_nodes))

            # Get the shortest path to the node from the gate
            distances, previous_nodes = util.dijkstra(G, node_allocations)

            # Allocate the nodes starting from behind so that there is no blocking
            for n, k in zip(loading_nodes, flattened_K_load):
                node_allocations[n] = k

                path = util.path_backtracking(previous_nodes, 0, n)

                route_list.append((path, k))

        print(f'Loading phase completed with {len(route_list)} routes including {len(last_rehandling_demands)} rehandling routes!')

        return route_list, node_allocations
```
