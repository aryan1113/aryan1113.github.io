2. Core abstraction (this is 80% of OR)

Every OR problem reduces to:

Decision variables
e.g.,

𝑥
𝑖
,
𝑗
=
1
x
i,j
	​

=1 if rider i takes order j
Objective function
minimize cost / maximize efficiency
e.g., total delivery time
Constraints
each order assigned once
rider capacity ≤ 3
delivery within 10 mins

If you understand this mapping, you understand OR at a functional level.

3. The 3 layers of OR (map to difficulty)
Layer 1: Linear Programming (LP)
continuous variables
solvable efficiently

Use case:

resource allocation

Key idea:

optimal solution lies at boundary

Layer 2: Integer Programming (IP / MIP)
discrete decisions (0/1, integers)
NP-hard → becomes real-world hard

This is where Zepto-type problems live.

Example:

routing
assignment
Layer 3: Heuristics (real-world systems)
“good enough, fast”
because exact solution too slow

Examples:

greedy
local search
genetic algorithms
4. The 5 canonical problems (know these, you’re covered)

Don’t go broad—just understand these deeply:

Assignment problem
→ who does what
Knapsack
→ what fits under constraints
Shortest path / graph
→ routing
Vehicle Routing Problem (VRP)
→ multi-agent routing (Zepto core problem)
Network flow
→ movement through systems

6. Tooling (just enough to recognize, not master)

You don’t need Gurobi depth, just awareness:

Google OR-Tools → practical entry point
PuLP / Pyomo → “write equations in Python”
Gurobi / CPLEX → industrial solvers

Key idea:

You define the problem → solver finds solution

Step 2 (4–5 hours)
Implement:
simple assignment problem
small VRP using OR-Tools

Goal:
→ see how problems are encoded