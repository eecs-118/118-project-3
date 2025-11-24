# EECS 118 -- P3: Searching for More Things

These exercises extend your earlier work on graph search to handle **multi-goal**, **heuristic**, and **greedy** search in the Catman environment.

You will still edit **only two Python files**:

- `p1_Search.py`
- `p1_SearchAgents.py`

All other project files are provided for you and should not be modified.

## Instructions

1. See `README.md` for Lab 1 for setup and implementation instructions.
2. Download `test_cases.zip` from Canvas and unzip it into your `p1_search` folder to get the test cases for E31--E35. Your folder structure should look like this:

	 ```
	 your_workspace_folder/
	 ├── p1_search/
	 │   ├── p1_Search.py
	 │   ├── p1_SearchAgents.py
	 │   ├── test_cases/
	 │   │   ├── E14/
	 │   │   ├── E15/
	 │   │   : 
	 │   │   ├── E35/
	 │   │   └── CONFIG
	 :
	 └── (other files and folders from previous labs)
	 ```

---

## E31 -- Implement A* Search Algorithm

In this exercise, you will implement the A* search algorithm in `p1_Search.py`. A* is an informed search algorithm that uses a heuristic function to guide the search.

### What you must implement

In `p1_Search.py`, complete:

```python
def aStarSearch(problem, heuristic):
    """Search the node that has the lowest combined cost and heuristic."""
    ...
```

The function should return a list of actions (e.g., ['North', 'South', 'East', 'West']) that leads to the goal state.

### How to run and debug E31

1. Move into the project folder:

   ```bash
   cd p1_search
   ```

2. Run Catman on a tiny corners layout with A*:

   ```bash
   python catman.py -l tinyCorners -p SearchAgent -a fn=astar,prob=CornersProblem --frameTime 0.5
   ```

You should see Catman plan a path using A* search.

---

## E32 -- Multi-Corner Navigation (CornersProblem)

Catman now needs to visit **all four special corners** of the maze. Each corner contains catnip stash that Catman must collect.
Your goal is to compute a **shortest path** that visits all four corners (in any order).

This is your first **multi-goal search problem**, so the search state must remember:

- Catman's current position, and  
- Which subset of corners have been visited so far.

### What you must implement

In `p1_SearchAgents.py`, complete the class:

```python
class CornersProblem(p1_Search.SearchProblem):
    def getStartState(self):
        """Return the initial search state."""
        ...

    def isGoalState(self, state):
        """Return True iff this is a goal state (all corners visited)."""
        ...

    def getSuccessors(self, state):
        """Return a list of (successor, action, cost) triples."""
        ...
```

Your implementation should:

- Represent the search state as something like:
  - `(position, visited_corners_info)`
- Mark a corner as visited **when Catman steps onto it**.
- Treat the state as a goal when **all four corners have been visited**.
- In `getSuccessors`, for each legal movement direction:
  - Compute the next position.
  - Update the visited-corners part of the state if the new position is a corner.
  - Return `(next_state, action, 1)`.

The search algorithm (DFS, BFS, UCS, A*) will run **on top of this problem** using your previous work in `p1_Search.py`.

### How to run and debug E32

1. Move into the project folder:

   ```bash
   cd p1_search
   ```

2. Run Catman on a tiny corners layout with BFS:

   ```bash
   python catman.py -l tinyCorners -p SearchAgent -a fn=breadthFirstSearch,prob=CornersProblem --frameTime 0.5
   ```

3. Try UCS on a medium layout:

   ```bash
   python catman.py -l mediumCorners -p SearchAgent -a fn=ucs,prob=CornersProblem --frameTime 0.5
   ```

4. Once you have E33 working (A* with a heuristic), you can also run:

   ```bash
   python catman.py -l mediumCorners -p AStarCornersAgent -z 0.5
   ```

You should see Catman plan a path that visits all four corners. You can print debugging information (e.g., the state) inside `getStartState`, `getSuccessors`, and `isGoalState` to verify your state representation.

### Autograder for E32

To run the autograder only for E32:

```bash
python autograder.py -q E32
```

To run all questions, including E32:

```bash
python autograder.py
```

The E32 test files live in:

```text
test_cases/E32/
```

They check:

- Correct state representation,
- Correct successor generation,
- Proper tracking of visited corners,
- Solution length and number of expanded nodes.

---

## E33 -- Admissible Heuristic for the Corners Problem

Your `CornersProblem` works, but searching the full state space can still be expensive. In E33, you will design a **heuristic function** that estimates how much cost remains to visit all unvisited corners from a given state.

The heuristic will be used with **A*** search to speed up the solution while still guaranteeing optimality.

### What you must implement

In `p1_Search.py`, complete:

```python
def cornersHeuristic(state, problem):
    """Heuristic for the CornersProblem."""
    ...
```

Here:

- `state` is the current search state from `CornersProblem`
  - typically `(position, visited_corners_info)`.
- `problem` is the `CornersProblem` instance.

### Requirements

Your heuristic must:

- Be **admissible**: never overestimate the true remaining cost.
- Be **consistent**: satisfy the triangle inequality for all states and actions.
- Return `0` when the state is already a goal (all corners visited).
- Use information about **remaining corners** to give a meaningful lower bound.

Helpful ideas:

- Consider the distance from the current position to the **closest** remaining corner, plus an estimate for the rest.
- Or, compute something like the **maximum** of distances to remaining corners.
- You can use Manhattan distance as a lower bound, or maze distance (if used carefully).

### How to run and debug E33

Use your heuristic with the provided A* Corners agent:

```bash
cd p1_search

# Tiny corners with A*
python catman.py -l tinyCorners -p AStarCornersAgent -z 0.5

# Medium corners with A*
python catman.py -l mediumCorners -p AStarCornersAgent -z 0.5
```

Compare:

- The total path cost.
- The number of **expanded nodes** printed by the problem.

A better heuristic (still admissible) should **reduce the number of expanded nodes** compared to using the null heuristic with A* (or UCS).

### Autograder for E33

To run only E33 tests:

```bash
python autograder.py -q E33
```

To run all tests:

```bash
python autograder.py
```

The tests in:

```text
test_cases/E33/
```

check that:

- Your heuristic is admissible and (effectively) consistent,
- A* with your heuristic finds optimal paths,
- Performance is within expected expansion limits.

---

## E34 -- Food Search Heuristic (Many-Goal Search)

Now Catman must eat **every piece of food** (every dot) in the maze. Unlike E32, here there may be **dozens of goals**, and naive search can be extremely slow.

You will write a heuristic that helps A* solve this **FoodSearchProblem** efficiently.

### What you must implement

In `p1_SearchAgents.py`, you already have:

```python
class AStarFoodSearchAgent(SearchAgent):
    """A SearchAgent for FoodSearchProblem using A* and your foodHeuristic"""
    def __init__(self):
        self.searchFunction = lambda prob: p1_Search.aStarSearch(prob, foodHeuristic)
        self.searchType = FoodSearchProblem
```

You must implement the heuristic:

```python
def foodHeuristic(state, problem):
    """Heuristic for the FoodSearchProblem."""
    ...
```

Here:

- `state` is `(pacmanPosition, foodGrid)`,  
  where `foodGrid` can be converted to a list of positions with `foodGrid.asList()`.
- `problem` is a `FoodSearchProblem` instance. You can access:
  - `problem.walls`
  - `problem.start`
  - `problem.heuristicInfo` (a dictionary that persists across calls)

### Requirements

Your heuristic must:

- Be **admissible**.
- Should also be **consistent**.
- Use information about **all remaining food pellets**.
- Perform significantly better than the trivial heuristic `0`.

Ideas:

- Think about the **distance to the farthest food**.
- Consider using `mazeDistance` between points to get exact path distances.
- Consider lower bounds like:
  - The distance to the farthest dot, or
  - The length of a minimal spanning path over remaining dots (approximate).

Be careful: computing very expensive distances inside the heuristic can be correct but slow. Caching via `problem.heuristicInfo` may help.

### How to run and debug E34

Use the provided A* Food agent:

```bash
cd p1_search

# Small test layout
python catman.py -l testSearch -p AStarFoodSearchAgent

# Slightly trickier layouts
python catman.py -l tinySearch -p AStarFoodSearchAgent
python catman.py -l trickySearch -p AStarFoodSearchAgent

# Larger layouts
python catman.py -l mediumSearch -p AStarFoodSearchAgent
```

Watch:

- The path Catman takes,
- The **total cost**,
- The number of **search nodes expanded**.

Once your heuristic is working well, Catman should clear all food with far fewer expansions than UCS or A* with the null heuristic.

### Autograder for E34

To run only E34 tests:

```bash
python autograder.py -q E34
```

To run all tests:

```bash
python autograder.py
```

The tests in:

```text
test_cases/E34/
```

check:

- Optimality (still finds minimal solutions),
- Heuristic admissibility,
- That the number of expanded nodes is below certain thresholds.

---

## E35 -- Closest-Dot Search (Greedy Repeated BFS)

Finally, you will implement a **greedy strategy** for eating all food:

1. From the current state, find the **closest piece of food**.
2. Use BFS to find a shortest path to that food.
3. Move along that path.
4. Repeat until there is no food left.

You will implement the helper that finds the path to the nearest dot.

### What you must implement

In `p1_SearchAgents.py`, complete:

```python
class ClosestDotSearchAgent(SearchAgent):
    """Search for all food using a sequence of searches"""
    def registerInitialState(self, state):
        ...
        while (currentState.getFood().count() > 0):
            nextPathSegment = self.findPathToClosestDot(currentState)
            ...

    def findPathToClosestDot(self, gameState):
        """Return a list of actions that reaches the closest dot."""
        ...
```

The helper problem is already defined:

```python
class AnyFoodSearchProblem(PositionSearchProblem):
    ...
    def isGoalState(self, state):
        """Fill this in with a goal test that returns True when state has food."""
        ...
```

You must:

- Implement `AnyFoodSearchProblem.isGoalState` so that the goal is **any position that contains food**.
- Implement `findPathToClosestDot(gameState)` to run a search (e.g., BFS) on `AnyFoodSearchProblem` and return the list of actions to reach the **nearest** food.

### Requirements

- `findPathToClosestDot` must use your general search algorithms (e.g., BFS from `p1_Search.py`) on `AnyFoodSearchProblem`.
- The path it returns must be valid: all actions legal from the starting state.
- The greedy strategy of "one closest dot at a time" should successfully clear all food on the layouts we test.
- The solution does not need to be **globally optimal**, but it must be correct and reasonably efficient.

### How to run and debug E35

Use the `ClosestDotSearchAgent`:

```bash
cd p1_search

# Small debug layouts
python catman.py -l testSearch -p ClosestDotSearchAgent
python catman.py -l tinySearch -p ClosestDotSearchAgent

# Trickier layout
python catman.py -l trickySearch -p ClosestDotSearchAgent

# Larger search layouts
python catman.py -l mediumSearch -p ClosestDotSearchAgent
```

Watch Catman as it repeatedly finds and goes to the nearest piece of food.  
If you see illegal moves or crashes, carefully check:

- Your goal test in `AnyFoodSearchProblem.isGoalState`.
- The path returned by `findPathToClosestDot`.

### Autograder for E35

To run only E35 tests:

```bash
python autograder.py -q E35
```

To run all tests:

```bash
python autograder.py
```

The tests in:

```text
test_cases/E35/
```

check:

- Correct goal test,
- Correct BFS-like behavior for the nearest food,
- That the greedy sequence of closest-dot paths eventually clears all food,
- That performance is within reasonable bounds.

---

## Notes

### Summary of Editable Files for Lab 3

You only modify:

| Question | File You Modify      | Main Functions / Classes                  |
|----------|----------------------|-------------------------------------------|
| E31       | `p1_Search.py`       | `aStarSearch`                        |
| E32       | `p1_SearchAgents.py` | `CornersProblem.getStartState`, `isGoalState`, `getSuccessors` |
| E33       | `p1_Search.py`       | `cornersHeuristic`                        |
| E34       | `p1_SearchAgents.py` | `foodHeuristic`                           |
| E35       | `p1_SearchAgents.py` | `AnyFoodSearchProblem.isGoalState`, `findPathToClosestDot` |

No other files should be edited.

### General Testing Advice

- Start with **tiny layouts** (`tinyCorners`, `tinySearch`, etc.).
- Add `print(...)` statements to inspect:
  - States, successors, and visited sets.
  - Heuristic values for different states.
- Use `python autograder.py -q Exx` to focus on one question (replace `Exx` with E31-E35) at a time.
- When things look good, run:

  ```bash
  python autograder.py
  ```

to see your full provisional score before submitting to Gradescope.
Instructions for submission to Gradescope will be provided on Canvas.

---

That's it for now...

(by Ibrahim Albool)

---

![Willem Dafoe looking up at the assignment](/assets/willem-dafoe-looking-up-600.jpg)

Meme by me, as usual.