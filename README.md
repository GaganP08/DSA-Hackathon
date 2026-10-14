# DSA-Hackathon

This repository contains solutions and utilities for several algorithmic problems used in a hackathon-style set of challenges. The main components are:

- common/ - small utilities (CSV read/write helper).
- problem1_maze/ - maze generation and solver + visualization.
- problem2_closest_pair/ - closest-pair-of-points algorithms (Python) and a Java version.
- Other directories (problem3_traveller, problem4_timetable) exist in the repo root.

---

## Structure / Important files

- `common/csv_utils.py`  
  - read_csv(path) -> list of rows (each row is a list of strings)  
  - write_csv(path, rows) -> writes `rows` to CSV

- `problem1_maze/`
  - `maze_solver.py` — maze generator + solver (Python).
  - `visualize_maze.py` — animation/visualization of a solution (Python).
  - `maze.csv`, `maze_path.csv` — example CSV grids (committed sample files).

- `problem2_closest_pair/`
  - `closest_pair.py` — Python implementation comparing brute-force and divide-and-conquer approaches.
  - `visualize_closest.py` — (visualization helper; reads `points.csv` and shows the closest pair).
  - `points.csv` — sample points produced by the script.
  - `ClosestPair.java` — Java implementation of the closest pair problem.
  - `dup_case.txt` — example test case (likely for duplicate points).

- `graph_path.csv` — another CSV present at repo root (small example file).

---

## problem1_maze — maze_solver.py

Purpose:
- Solve a grid-based maze that may include keys and doors.
- Optionally generate a random maze with a guaranteed solvable placement of T key-door pairs.

Key functions and behavior:
- read_grid(path) / write_grid(path, grid): CSV IO helpers for the maze (each cell is one CSV entry).
- DOOR_LETTERS / KEY_LETTERS: the implementation expects keys `a`..`f` (and `g` treated specially) and matching doors `A`..`F` (plus `G`).
- solve_maze(grid): BFS on state (row, col, key_mask). The BFS state includes a bitmask for collected keys. Doors are passable only if the matching key bit is set. Returns (distance, path) where path is a list of (r,c).
- _carve_maze(n, seed): randomized DFS carving to produce a maze of '.' and '#'. Ensures connectivity among open cells.
- generate_maze(n, T, wall_p, seed): carves maze, picks distant S and E, selects positions along the S-E path to place T keys and T doors (in order) to force key/door order, and verifies solvability with solve_maze.
- mark_path(grid, path): returns a copy of the grid with visited path cells replaced by 'x' (leaves S and E intact).
- CLI options (main):
  - --input (default None)
  - --output (default 'maze_path.csv')
  - --generate (generate a maze instead of reading input)
  - --n (grid size, default 10)
  - --t (number of key-door pairs to place, default 3)
  - --seed (random seed)
  - --wall-p (unused in generator body except as parameter; generator uses carving approach)

Usage examples:
- Generate a maze and solve it:
  - python problem1_maze/maze_solver.py --generate --n 11 --t 3 --seed 42
  - Expected printed output (example):
    - Generated 11x11 maze with T=3 -> maze.csv
    - Shortest path length: 34
    - Marked path saved to maze_path.csv
  - Output files:
    - `maze.csv` — generated grid with characters:
      - `#` wall
      - `.` open cell
      - `S` start
      - `E` exit
      - `a`..`f` keys and `A`..`F` doors
    - `maze_path.csv` — grid identical to maze.csv but path cells that were '.' become 'x'

- Solve an existing maze file:
  - python problem1_maze/maze_solver.py --input problem1_maze/maze.csv --output out_path.csv
  - Output: prints "Shortest path length: <d>" and writes `out_path.csv` when solvable.

Notes on behavior:
- The solver handles keys and doors using a 6-bit bitmask (a-f mapped to bits 0..5). Keys are picked up when stepping on their cell.
- BFS state is (r,c,mask) so it explores different key inventories separately.
- If no path found, distance is -1 and no path file is written.

---

## problem1_maze — visualize_maze.py

Purpose:
- Visualize the path found by the solver as an animation using matplotlib.

Key points:
- Reads the same grid format as `maze_solver.py` via read_grid.
- Calls solve_maze(grid) to get the path and then builds frames incrementally showing the path progression and the key inventory at each step.
- Colors:
  - Walls: black
  - Start: green
  - Exit: red
  - Keys and doors: colored according to the script's palette; collected keys/doors are shown differently
  - Path so far: blue-ish
- CLI:
  - --input (default 'maze.csv')
  - --save (GIF filename) — if provided, saves the animation (writer='pillow')
  - --interval (ms between frames, default 300)

Usage:
- python problem1_maze/visualize_maze.py --input maze.csv
- Or save: python problem1_maze/visualize_maze.py --input maze.csv --save anim.gif

Outputs:
- Opens a matplotlib window showing the animation, or writes an animation GIF when `--save` is used.

---

## problem2_closest_pair — closest_pair.py

Purpose:
- Compare brute-force and divide-and-conquer methods for the closest pair of points in 2D.
- Generate a points CSV and print results and timing for both approaches.

Key functions:
- dist2(a,b): squared Euclidean distance.
- brute_force(points): O(N^2) approach; returns (distance, (p1,p2)).
- closest_pair(points): divide-and-conquer approach. Implementation notes:
  - Sorts indices by x-coordinate.
  - Recursively splits the points. For small subproblems (<=3) uses brute force inside recursion.
  - Builds a vertical strip around the midpoint and checks up to 7 neighbors by y-sorted order (classic strip optimization).
  - Returns (distance, pair).
- main CLI:
  - --n (number of points to generate, default 200)
  - --seed (random seed)
  - --float (generate float coordinates uniformly instead of integers)
  - --csv (path to write points; default 'points.csv')

Usage example:
- python problem2_closest_pair/closest_pair.py --n 200 --seed 7
- Produces `points.csv` containing one point per line `x,y`.
- Example output (format):
  - Brute-Force : 1.234567  (x1, y1)  (x2, y2)  time=12.345 ms
  - Divide&Conq : 1.234567  (x1, y1)  (x2, y2)  time=1.234 ms
  - Agree: True

Notes:
- The script measures and prints the time taken by both algorithms and whether both reported the same distance (within a tiny tolerance).
- The Python D&C implementation runs in roughly O(N log^2 N) given the approach; practical performance is much better than brute-force for large N.

---

## problem2_closest_pair — visualize_closest.py

Purpose:
- Plot the set of points (reads `points.csv`) and highlight the closest pair found by an algorithm.  
Usage (typical):
- python problem2_closest_pair/visualize_closest.py --input points.csv --save closest.gif
- Expected to display the scatter of points and show the closest pair in a different color or line.

(If you want, I can open and include exact usage/help and format; currently the repository contains this file but I didn't include its full contents here.)

---

## problem2_closest_pair — ClosestPair.java

Purpose:
- A Java implementation of the closest pair algorithm is included (file `ClosestPair.java`).  
- Likely implements the same divide-and-conquer approach; you can compile and run with `javac ClosestPair.java` then `java ClosestPair` depending on the class's CLI.

---

## Team Members
1. Sri Balaji R(1CR25CS413)
2. Mohith G(1CR24CS105)
3. Mokshith Ramesh(1CR24CS106)
4. Amruth H(1CR25CS403) 
