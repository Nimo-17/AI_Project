# Solving Sudoku with A*, Hill Climbing and Backtracking

A comparison of three classic AI techniques on the same set of Sudoku puzzles, built for an Artificial Intelligence course. The notebook runs every algorithm on identical puzzles, measures how well and how fast each one solves them, and picks a winner from the results.

## What's inside

| File | Description |
|---|---|
| `sudoku_ai_analysis.ipynb` | The full analysis: solvers, experiment, charts and conclusion |
| `images/` | Chart images used in this README |
| `README.md` | This file |

## Algorithms compared

| Algorithm | Family | Idea |
|---|---|---|
| Backtracking | Constraint satisfaction | Fill cells one by one, undo on conflict |
| Backtracking + MRV | Constraint satisfaction | Same, but always fill the cell with the fewest legal values first |
| A* Search | Informed search | Expand the board with the lowest `f = g + h` |
| Hill Climbing | Local search | Start from a full random guess, keep swapping cells to reduce conflicts |

**How each one is set up**

- **Backtracking:** depth-first assignment with constraint checking. The MRV version picks the most constrained empty cell, so dead ends show up early.
- **A\*:** each state is a partially filled board. `g` is the number of cells filled, and `h` is the sum of legal values across all empty cells. Boards with an empty cell that has no legal value are discarded. The heuristic is not strictly admissible, which is acceptable here because every solution has the same length.
- **Hill Climbing:** every 3x3 box is filled with its missing digits at random, so only rows and columns can have duplicates. The cost is the number of duplicate digits across rows and columns. Each move swaps two non-given cells inside one box, and the search stops at a local minimum. It restarts up to 10 times.

## Dataset

[Kaggle: 9 Million Sudoku Puzzles and Solutions](https://www.kaggle.com/datasets/rohanrao/sudoku?select=sudoku.csv) (`sudoku.csv`).

The dataset is not included in this repo because of its size. Download it from Kaggle and place `sudoku.csv` in the same folder as the notebook.

To keep loading fast, the notebook reads a random 0.3% of the rows (seeded, so it is reproducible), then draws 60 puzzles from that sample. Puzzles are split into **Easy**, **Medium** and **Hard** thirds by their number of blank cells.

## How to run

1. Install the requirements:
   ```bash
   pip install numpy pandas matplotlib jupyter
   ```
2. Download `sudoku.csv` from the Kaggle link above and put it next to the notebook.
3. Start Jupyter and run all cells:
   ```bash
   jupyter notebook sudoku_ai_analysis.ipynb
   ```

Loading the CSV takes a few seconds. The experiment itself finishes quickly.

## Evaluation method

- All algorithms get exactly the same 60 puzzles.
- A run counts as solved only if the final board is a fully valid Sudoku that keeps every given digit.
- For each run the notebook records time taken and the number of states explored.
- Algorithms are ranked by success rate first, then by lower average time.

## Results

Results from one run of the notebook (random seed 42):

| Algorithm | Success rate | Mean time (s) | Median time (s) | Max time (s) | Mean states explored |
|---|---|---|---|---|---|
| Backtracking | 100% | 0.0051 | 0.0008 | 0.1835 | 720.6 |
| Backtracking + MRV | 100% | 0.0022 | 0.0020 | 0.0056 | 44.2 |
| A* | 100% | 0.0022 | 0.0018 | 0.0125 | 45.7 |
| Hill Climbing | 20% | 0.0300 | 0.0307 | 0.0625 | 3591.7 |

### 1. Overall comparison

![Overall comparison](images/overall-comparison.png)

Three of the four algorithms solved every puzzle. Hill climbing solved only 20%. On the right, the time axis is logarithmic, so each gridline is ten times the one below. Backtracking + MRV and A* are the fastest, plain backtracking is a little slower, and hill climbing is slowest even though it fails most of the time.

### 2. Search effort and time spread

![Search effort and time spread](images/search-effort-and-time-spread.png)

The left chart shows how many states each algorithm had to generate or evaluate. MRV and A* need only about 45 on average, plain backtracking about 720, and hill climbing about 3,600. The right chart shows how much the time varies between puzzles, where a short box means the algorithm is predictable. MRV and A* are tightly grouped. Plain backtracking has a much wider spread and one extreme case near 0.18 s. Hill climbing is consistently slow because it keeps restarting.

### 3. Effect of difficulty

![Effect of difficulty](images/effect-of-difficulty.png)

Puzzles with more blank cells are harder. On the left, hill climbing falls from roughly half the Easy puzzles to almost none on Medium and none on Hard, while the other three stay at 100%. On the right, plain backtracking is actually the quickest on Easy puzzles, but its time grows sharply on Hard ones. MRV and A* start slightly slower because they do extra work at every step, then stay almost flat as difficulty rises. (Success rates for hill climbing are read from the bars, so they are approximate.)

### 4. Time vs number of blanks

![Time vs number of blanks](images/time-vs-blanks.png)

Each dot is one puzzle run. MRV and A* (teal and orange) rise slowly and stay close together as blanks increase. Plain backtracking (blue) is scattered, mostly fast but with occasional spikes on puzzles with many blanks. Hill climbing (red) sits above everything else.

## Conclusion

- **A\* and Backtracking + MRV are effectively tied.** Both solved every puzzle with a mean time of about 0.0022 s and roughly 45 states explored. A* ranks first only because of the tie-break on tiny time differences, so the gap is not meaningful.
- **Plain backtracking** is also complete, but it explores far more states and has a much worse worst case (0.18 s vs 0.006 s for MRV). The MRV heuristic is what makes it efficient.
- **Hill climbing** is incomplete. It gets stuck in local minima where no single swap helps, so its success rate collapses as puzzles get harder.

Sudoku is a constraint satisfaction problem, so algorithms that use the constraints directly (backtracking with MRV, or A* with a constraint-based heuristic) are the natural fit.

## Limitations

- The sample is only 60 puzzles, so small differences (like A* vs MRV) can change between runs or sample sizes.
- Backtracking is capped at 50,000 assignments and A* at 3,000 expanded states, so a run that hits the cap counts as unsolved.
- Hill climbing is randomized, so its numbers vary with the seed.
- Timings depend on the machine they are run on.

## Tech stack

Python, NumPy, pandas, Matplotlib, Jupyter Notebook
