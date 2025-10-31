# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an **Impassable Gate** puzzle solver - a sokoban-style game where the player pushes numbered pieces (0-9) to cover goal squares (G) or goal pieces (I-Q). The project includes:
- An interactive ncurses-based game mode
- An AI solver using Iterative Width (IW) search algorithms

The codebase was adapted from an Epitech Sokoban project and modified for COMP20003 Assignment 3 2025.

## Build and Test Commands

**Build the project:**
```bash
make
```

**Clean build artifacts:**
```bash
make clean      # Remove object files
make fclean     # Remove object files and executable
make re         # Clean rebuild
```

**Run interactive game (manual play):**
```bash
./gate <puzzle_file>
# Example: ./gate test_puzzles/capability1
```

**Run AI solver (automated):**
```bash
./gate -s <puzzle_file>
# Example: ./gate -s test_puzzles/capability1
```

**Run all test puzzles manually:**
```bash
make runmanual
```

**Run all test puzzles with AI solver:**
```bash
make runtests
```

**Test Puzzles:**
- `capability1-13`: Solvable puzzles of increasing difficulty (13 puzzles)
- `impassable1-3`: Unsolvable puzzles (3 puzzles)

## Architecture

### Game State Structure (`gate_t` in include/gate.h)

The core game state is represented by the `gate_t` struct:
- `map`: Current game state (2D char array)
- `map_save`: Original/unchanging state reference
- `buffer`: Raw file buffer for puzzle loading
- `lines`, `num_chars_map`: Dimensions
- `player_x`, `player_y`: Player position
- `num_pieces`: Number of movable pieces (0-9)
- `piece_x[]`, `piece_y[]`: Piece positions (lowest y, tie-break by lowest x)
- `soln`: String representing moves made (format: "0u1d2l" = piece 0 up, piece 1 down, piece 2 left)

### Directory Structure

**Core game logic (`src/`):**
- `main.c`: Entry point, handles `-h` (help) and `-s` (solve) flags
- `play.c`: Interactive ncurses game loop
- `map_reading.c`, `map_check.c`: Puzzle file I/O and validation
- `find_player.c`: Locates player and pieces on the map
- `movement.c`, `key_check.c`: Movement validation and execution
- `win_check.c`: Win condition verification
- `helper.c`: Help text

**AI solver (`src/ai/`):**
- `ai.c`: Main solver entry point with `solve()` function
  - Implements three algorithms:
    1. Algorithm 1: BFS without duplicate detection (IW(n+1))
    2. Algorithm 2: BFS with radix tree duplicate detection (IW(n))
    3. Algorithm 3: Iterative Width with multi-tree novelty pruning
- `queue.c/.h`: Queue implementation for BFS (linked list based)
  - `applyAction()`: Core function that generates successor states
- `radix.c/.h`: Radix tree for state deduplication (bit-packed)
- `hashtable.c/.h`: Hash table implementation (from goldsborough/hashtable)
- `utils.c/.h`: Timing utilities (`now()` function)

**Libraries (`lib/`, `include/`):**
- `libmy.h`: Basic I/O (`my_putchar`, `my_putstr`)
- `gate.h`: All game-related function prototypes

### Key Architectural Patterns

**State Representation:**
- States are bit-packed using `packMap()` to minimize memory
- Piece positions encoded as: piece_id (pBits) + y_pos (hBits) + x_pos (wBits)
- `getPackedSize()` calculates bytes needed based on board dimensions

**Movement System:**
- Pieces are numbered 0-9 (character codes '0'-'9')
- Directions: 'u' (up), 'd' (down), 'l' (left), 'r' (right)
- `attempt_move()` validates moves before applying
- `move_location()` updates piece positions

**Win Condition:**
- Game won when no 'G' (goal squares) or 'I'-'Q' (goal pieces) remain uncovered in the map

### Compilation Notes

**Makefile quirk:** Lines 27-29 reference `.o` files directly instead of `.c`:
```make
src/ai/radix.o \
src/ai/ai.o \
src/ai/utils.o
```
These should be built from their corresponding `.c` files before linking.

**Dependencies:**
- ncurses library (`-lncurses` flag required)
- Standard C libraries
- **Compiler**: Uses `gcc-15` specifically (see Makefile line 9)

### AI Solver Implementation

**Switching Algorithms:**
The `find_solution()` function in `src/ai/ai.c:665-668` controls which algorithm runs. Comment/uncomment to switch:
```c
void find_solution(gate_t* init_data) {
    // algo1(init_data);  // BFS without duplicate detection
    algo2(init_data);     // BFS with radix tree duplicate detection
    // algo3(init_data);  // IW with multi-tree novelty pruning
}
```

**Three Implemented Algorithms:**
1. **algo1** (lines 186-316): Breadth-First Search without duplicate detection (IW(n+1))
2. **algo2** (lines 321-475): BFS with single radix tree for duplicate detection (IW(n))
3. **algo3** (lines 480-659): Iterative Width search with multiple radix trees for novelty pruning

**Key AI Components:**
- `duplicate_state()` (ai.c:41-94): Deep copies gate_t including all dynamic allocations
- `free_state()` (ai.c:100-139): Frees state-specific memory (map, solution string)
- `free_initial_state()` (ai.c:141-181): Frees unchanging initial data (buffer, map_save)
- `applyAction()` (queue.c): Generates successor states by applying piece movements
- `winning_state()` (ai.c:722-731): Checks if all goals are covered
- `packMap()` (ai.c:686-717): Bit-packs state for radix tree storage
- `getPackedSize()` (ai.c:674-681): Calculates bytes needed for bit-packed state

### Puzzle File Format

Example (`test_puzzles/capability1`):
```
########
###GG###
###HH###
#  00  #
##    ##
#      #
#      #
#      #
#      #
########
```

- `#`: Walls
- `G`: Goal squares
- `H-Q`: Goal pieces (placed pieces complete the level)
- `0-9`: Movable pieces
- Space: Empty walkable space

## Using Gemini CLI for Large Codebase Analysis

When Claude's context window is insufficient for analyzing large codebases, use the Gemini CLI (`gemini -p`) to leverage Google Gemini's massive context capacity.

### When to Use Gemini CLI vs Claude

**Use Gemini CLI for:**
- Entire codebase analysis (>100KB of files)
- Project-wide pattern detection
- Architecture overview across multiple directories
- Verification of feature implementation across many files
- Questions requiring full repository context

**Continue using Claude for:**
- Single file analysis or small file sets
- Writing/editing code
- Detailed explanations and tutorials
- Interactive debugging
- General programming questions

### Basic Syntax

Use `@` to include files/directories (paths relative to where you run the command):
```bash
# Single file
gemini -p "@src/ai/ai.c Explain the three algorithm implementations"

# Multiple files
gemini -p "@include/gate.h @src/ai/ai.c Analyze the AI data structures"

# Directory
gemini -p "@src/ai/ Summarize the solver architecture"

# Multiple directories
gemini -p "@src/ @include/ Explain the overall project structure"

# Entire project
gemini -p "@./ Overview of this Impassable Gate solver"
# Or equivalently:
gemini --all_files -p "Analyze project structure"
```

### Implementation Verification Queries

**Feature Detection:**
```bash
# Check algorithm completeness
gemini -p "@src/ai/ Are all three algorithms (algo1, algo2, algo3) fully implemented?"

# Verify memory management
gemini -p "@src/ai/ai.c Is proper cleanup implemented? Check malloc/free pairs"

# Find specific patterns
gemini -p "@src/ List all functions that modify the game state"
```

**Performance & Optimization:**
```bash
# Check bit-packing
gemini -p "@src/ai/ Explain how state bit-packing works across all algorithms"

# Radix tree usage
gemini -p "@src/ai/ How is the radix tree used differently in algo2 vs algo3?"

# Memory efficiency
gemini -p "@src/ai/ Compare memory usage between the three algorithms"
```

### Best Practices

1. **Be specific in queries** - Vague questions on large codebases yield vague answers
2. **Use directory scoping** - Don't include unnecessary directories
3. **Combine with Claude** - Use Gemini for discovery, Claude for implementation
4. **Path awareness** - Always run from project root for consistent paths

### Example Workflow

1. **Discovery Phase** (Gemini):
```bash
   gemini -p "@src/ai/ Explain the differences between algo1, algo2, and algo3"
```

2. **Implementation Phase** (Claude):
   - Take Gemini's findings back to Claude
   - Focus on specific files identified
   - Write/modify code with Claude's assistance

### Important Notes

- No `--yolo` flag needed for read-only analysis
- Gemini includes file contents directly in context
- Results are best when queries are specific and focused
- Consider breaking very large projects into logical segments
