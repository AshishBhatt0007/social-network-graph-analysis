# Social Network Connection and Recommendation Analysis

A Python project that models a social network as a graph (users = vertices, friendships = edges) and implements:

- **BFS** and **DFS** traversal to explore connections
- **Mutual connections** between two users
- **Shortest connection path** (degrees of separation) using BFS
- **Friend recommendations** (friends-of-friends ranked by mutual friends)
- **Connected components** (separate friend groups)
- **Performance analysis** of BFS/DFS/shortest path on networks of different sizes (random and "hub-style" networks)

Only the standard library is needed to run the algorithms; `matplotlib` is used just for the benchmark chart.

## Project structure

```
social-network-analysis/
├── main.py                    # command-line interface
├── social_network/
│   ├── graph.py               # Graph class (adjacency list, CSV load/save)
│   ├── algorithms.py          # BFS, DFS, mutual, shortest path, recommendations
│   ├── generator.py           # random + hub (preferential attachment) graphs
│   └── benchmark.py           # timing + CSV + chart
├── data/sample_network.csv    # small example network
├── tests/test_algorithms.py   # unit tests
├── results/                   # benchmark CSVs and charts
├── requirements.txt
└── LICENSE
```

## Setup

```bash
git clone https://github.com/<your-username>/social-network-analysis.git
cd social-network-analysis
python -m venv .venv
# Windows: .venv\Scripts\activate     Mac/Linux: source .venv/bin/activate
pip install -r requirements.txt
```

## Usage

```bash
python main.py demo                      # runs everything on the sample data
python main.py bfs Alice
python main.py dfs Alice
python main.py mutual Alice Eve
python main.py path Alice Judy
python main.py recommend Alice
python main.py components
python main.py --data my_edges.csv bfs Alice     # use your own data
```

Your own data is a CSV with two columns (`user_a,user_b`), one friendship per line.

### Sample output (`python main.py demo`)

```
BFS from Alice : Alice -> Bob -> Carol -> David -> Eve -> Frank -> Grace -> Heidi -> Ivan -> Judy
DFS from Alice : Alice -> David -> Frank -> Grace -> Heidi -> Eve -> Carol -> Bob -> Ivan -> Judy
Mutual (Alice, Eve): ['Bob', 'Carol']
Shortest Alice -> Judy: Alice -> Bob -> Eve -> Heidi -> Ivan -> Judy (5 hops)
Recommendations for Alice: [('Eve', 2), ('Frank', 1)]
Connected components: 2 (sizes [10, 2])
```

### Run the tests

```bash
python -m unittest discover -s tests -v
```

## Performance analysis

```bash
python main.py benchmark                                   # random networks
python main.py benchmark --kind hub                        # hub-style networks
python main.py benchmark --sizes 1000 10000 100000 --repeats 5
```

This prints a table and saves `results/benchmark_<kind>.csv` and a chart `results/benchmark_<kind>.png`.

Example results (random network, about 10 friends per user, median of 5 runs; your numbers will differ by machine):

| Users | Edges | BFS (ms) | DFS (ms) | Shortest path (ms) |
|------:|------:|---------:|---------:|-------------------:|
| 1,000 | 4,962 | 0.49 | 0.65 | 0.20 |
| 5,000 | 24,973 | 4.37 | 5.23 | 1.14 |
| 10,000 | 49,969 | 10.43 | 12.82 | 0.37 |
| 50,000 | 249,972 | 99.17 | 157.00 | 14.40 |
| 100,000 | 499,974 | 370.67 | 410.48 | 10.63 |

### Observations

- **Time complexity:** BFS and DFS are both **O(V + E)**: each user and friendship is processed once. Measured time grows roughly in line with the network size.
- **Space complexity:** O(V) for the visited set plus the queue (BFS) or stack (DFS).
- **BFS vs DFS:** similar speed. They differ in purpose: BFS finds the fewest-hops path and works level by level (friends, then friends-of-friends); DFS goes deep first and suits connectivity checks and components.
- **Shortest path** stops as soon as the target is found, so it is often much faster than a full traversal. Its worst case (target far away or unreachable) is also O(V + E).
- **Random vs hub networks:** real social networks have a few very popular users. Hub-style graphs have shorter paths between users, which is why the shortest-path search finishes quickly.
- Timing is affected by the machine, Python's memory allocation and garbage collection, so the median of several runs is reported.

## How it works (short)

| Feature | Technique |
|---|---|
| Graph storage | Adjacency list (`dict` of user -> friends) |
| BFS | `collections.deque` as a queue |
| DFS | Explicit stack (no recursion limit problems on big graphs) |
| Mutual friends | Set intersection of two friend lists |
| Shortest path | BFS with a `parent` map, then walk back from target to source |
| Recommendations | Count friends-of-friends who are not already friends, rank by count |

## License

MIT
