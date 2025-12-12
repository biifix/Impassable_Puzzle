# Impassable Gate

**Impassable Gate** is a Sokoban-style puzzle solver project that features both an interactive playable mode and a powerful AI solver. The goal is to push numbered pieces to cover goal squares or goal pieces, navigating through complex mazes.

This project was developed for COMP20003 Assignment 3 (2025), adapted from an Epitech Sokoban project.

## Features

- **Interactive Gameplay**: Play the puzzles manually using a terminal-based interface (ncurses).
- **AI Solver**: Automated solver capable of finding optimal solutions using advanced search algorithms.
- **Multiple Algorithms**:
    - **Algorithm 1**: Naive Breadth-First Search (IW(n+1)). Fast for simple puzzles but memory-intensive.
    - **Algorithm 2**: BFS with Radix Tree for duplicate detection (IW(n)). Memory-optimized but can be slower.
    - **Algorithm 3**: Iterative Width with Multi-tree Novelty Pruning. **The best performer**, offering a balance of speed and memory efficiency, solving even the hardest puzzles.
- **Comprehensive Test Suite**: Includes a variety of puzzles ranging from simple capability tests to "impassable" challenges.

## Installation

### Prerequisites
- GCC Compiler (supporting C11/GNU11, specifically `gcc-15` as configured in Makefile)
- Make
- ncurses library

### Build
To compile the project, simply run:

```bash
make
```

To clean up build artifacts:

```bash
make clean      # Remove object files
make fclean     # Remove object files and executable
make re         # Rebuild from scratch
```

## Usage

### Manual Play
To play a puzzle interactively:

```bash
./gate <puzzle_file>
```
Example:
```bash
./gate test_puzzles/capability1
```
Use arrow keys or WASD (depending on implementation) to move.

### AI Solver
To run the AI solver on a puzzle:

```bash
./gate -s <puzzle_file>
```
Example:
```bash
./gate -s test_puzzles/capability1
```

### Running Tests
To run the full suite of test puzzles:

```bash
make runmanual  # Run all puzzles in manual mode sequentially
make runtests   # Run all puzzles with the AI solver
```

## Performance Summary

Extensive testing across 43 test cases shows that **Algorithm 3** is the superior solver.

| Metric | Algorithm 1 | Algorithm 2 | Algorithm 3 |
| :--- | :--- | :--- | :--- |
| **Success Rate** | ~69% | 100% | **100%** |
| **Avg Time** | 1.1s | 63.5s | **4.7s** |
| **Avg Nodes** | 52k | 653k | **12k** |
| **Memory Overhead** | 0 KB | ~370 MB | **0 KB** |

*Note: Algorithm 1 fails on complex puzzles. Algorithm 2 uses significant memory for the radix tree. Algorithm 3 is highly efficient in both time and space.*

## Project Structure

- `src/`: Source code for game logic and main program.
- `src/ai/`: AI solver implementation (BFS, Radix Tree, Iterative Width).
- `include/`: Header files.
- `lib/`: Helper libraries.
- `test_puzzles/`: Collection of puzzle files for testing.

## Credits

- Original Project: Epitech Sokoban
- Adapted by: Thomas Minuzzo (2024) for Chessformer
- Adapted by: Grady Fitzpatrick (2025) for Impassable Gate
