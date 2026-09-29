# libMultiRobotPlanning

A fork of Wolfgang Hönig's [libMultiRobotPlanning](https://github.com/whoenig/libMultiRobotPlanning), a C++14 library of templated search algorithms for multi-robot/multi-agent task and path planning. All of the original algorithms, and the credit for them, belong to the upstream project; this fork reorganizes the repository so that it can be used as a solver library by an external benchmarking harness (see [Changes in this fork](#changes-in-this-fork)).

## Algorithms

* Single-agent
  * A*
  * A* epsilon (focal search)
  * SIPP (Safe Interval Path Planning)
* Multi-agent
  * Conflict-Based Search (CBS), on grids and on generalized roadmaps
  * Enhanced Conflict-Based Search (ECBS)
  * CBS and ECBS with optimal task assignment (CBS-TA, ECBS-TA)
  * Prioritized planning using SIPP
* Assignment
  * Minimum sum-of-cost assignment (flow-based; integer costs; any number of agents/tasks)
  * Next-best assignment (series of optimal solutions)

## Changes in this fork

Compared with upstream [`4c75fa2`](https://github.com/whoenig/libMultiRobotPlanning/commit/4c75fa2):

### Repository layout

* `example/*.cpp` → `src/`: the command-line programs are the solvers this fork is used for, not just examples.
* `example/*.py` → `tools/`: `visualize_roadmap.py` now sits next to `annotate_roadmap.py` and `collision.py`.
* Removed `visualize.py` (grid animation): it now lives in the benchmarking project, which uses it for the solutions of every solver library. The tests' `createVideo` option runs it from there (`../../../tools/visualize.py`, relative to `build/`).
* Removed `standard_benchmark_converter.py` (Moving AI → YAML): it now lives in the benchmarking project, which converts the Moving AI benchmarks for this library.
* Removed `benchmark/`: the benchmark scenarios now live in the benchmarking project that uses this library as a submodule.
* Removed the GitHub Actions workflow, `InstallPackagesUbuntu`, `.clang-format`, `.clang-tidy`, `doc/libMultiRobotPlanning.md` and the top-level and `tools/` `__init__.py` files.
* Added [`requirements.txt`](requirements.txt) for the Python tools.

### Build configuration

* The build type defaults to `Release` when none is given. Unoptimized, `cbs_ta` and `ecbs_ta` take minutes per run on a 32x32 map.
* The `docs` target is only created if both Doxygen and `doc/Doxyfile.in` are present, and `Doxyfile.in` was regenerated.
* The `run-test` target was removed (and dropped from `everything`); run the tests as shown under [Tests](#tests).

### Behaviour changes

* `ShortestPathHeuristic` (used by `cbs_ta`/`ecbs_ta`) no longer writes `searchGraph.dot` into the working directory on every run. That file is only useful for debugging, and parallel runs overwrote each other's copy. Pass a file name to the new `dotFile` constructor argument to get it.
* `tools/annotate_roadmap.py` works both as a script and when imported (including in its multiprocessing workers), and converts vertex coordinates to floats, so integer roadmaps no longer fail in `tools/collision.py`.
* `tools/collision.py`: the bounding-box precheck no longer modifies its arrays in place, which failed for integer inputs.
* The Python tools use PyYAML's C loader when it is available, which is much faster on large schedules.

## Layout

| Path | Contents |
| --- | --- |
| `include/libMultiRobotPlanning/` | The algorithms, as header-only class templates |
| `src/` | One command-line program per algorithm, each built into `build/` |
| `tools/` | Python roadmap visualizer and roadmap annotation |
| `test/` | Python unit tests and their problem files |
| `doc/Doxyfile.in` | Doxygen template for the `docs` target |

### `include/` vs. `src/`

`include/libMultiRobotPlanning/*.hpp` holds the search algorithms as header-only class templates, e.g. `ECBS<State, Action, Cost, Conflict, Constraints, Environment>` in `ecbs.hpp`. A template knows nothing about grids or coordinates, only about the `State`/`Action`/`Environment` types it is instantiated with, so the whole class lives in the header. Each header's doc comment lists the methods (`getNeighbors`, `isSolution`, `admissibleHeuristic`, `focalHeuristic`, ...) that the `Environment` type must provide.

`src/*.cpp` instantiates those templates into command-line solvers. Each file (e.g. `ecbs.cpp`) defines the concrete `State`, `Action` and `Conflict`/`Constraints` types for a 2D grid (or, for `cbs_roadmap.cpp`, a roadmap graph), an `Environment` implementing the interface the header expects, and a `main()` that reads a problem from YAML, runs the algorithm and writes the solution back out.

So the header is the reusable algorithm and the `.cpp` file is one concrete problem plugged into it: a new kind of problem (e.g. continuous space) means a new `.cpp` file, not a change to the header.

## Build

Requires CMake, Boost (`program_options`, graph) and yaml-cpp.

```sh
cmake -S . -B build
cmake --build build -j
```

On the Oscar cluster, run `module load boost` and `module load yaml-cpp` first. Pass `-DCMAKE_BUILD_TYPE=Debug` for an unoptimized debug build.

API docs (needs Doxygen): `cmake --build build --target docs`, written to `build/doc/html/index.html`.

## Running a solver

Every program reads a problem with `-i` and writes the solution with `-o`:

```sh
cd build
./ecbs -i ../test/mapf_simple1.yaml -o output.yaml -w 1.3
```

Run any program with `--help` for its options.

### Problem files

Grid problems are YAML files with a `map` and a list of `agents`:

```yaml
agents:
-   name: agent0
    start: [1, 7]
    goal: [6, 3]
map:
    dimensions: [8, 8]
    obstacles:
    - [4, 5]
```

The task-assignment solvers (`cbs_ta`, `ecbs_ta`) instead read a list of `potentialGoals` per agent.

Roadmap problems (for `cbs_roadmap`) have agents that start and end at named vertices, and a `roadmap` section. CBS works on general graphs, with a particular focus on optional wait actions (so it can be used with motion primitives), but the annotation and visualization tools assume a 2D Euclidean embedding with straight-line edges:

```sh
cd build
python3 ../tools/annotate_roadmap.py ../test/mapf_simple1_roadmap_to_annotate.yaml annotated.yaml
./cbs_roadmap -i annotated.yaml -o output.yaml
python3 ../tools/visualize_roadmap.py annotated.yaml output.yaml
```

## Python tools

```sh
pip install -r requirements.txt
```

`cvxpy` is only needed for `annotate_roadmap.py`. Saving a video (`--video`) also needs `ffmpeg` on your `PATH`.

| Script | Usage |
| --- | --- |
| `tools/visualize_roadmap.py` | `python tools/visualize_roadmap.py <roadmap.yaml> <schedule.yaml> [--video FILE] [--speed N] [--radius R]`: animate a roadmap solution (`<schedule.yaml>` is `cbs_roadmap`'s `-o` output). Without `--video` the animation opens in a window. Grid solutions are animated by the benchmarking project's `tools/visualize.py`. |
| `tools/annotate_roadmap.py` | `python tools/annotate_roadmap.py <roadmap.yaml> <annotated.yaml> [radius]`: add edge-collision information for robots of the given radius (default 0.3) so that `cbs_roadmap` can avoid conflicts. Uses 8 processes. |

## Tests

The tests run the binaries from `build/`, so run them from there:

```sh
cd build
python3 -m unittest discover -s ../test
python3 ../test/test_next_best_assignment.py TestNextBestAssignment.test_1by2   # a single test
```

## License

MIT, as upstream; see [LICENSE](LICENSE).
