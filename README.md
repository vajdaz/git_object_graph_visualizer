# Git Object Graph Visualizer

A Python script that creates a comprehensive Graphviz visualization of all Git objects and references in a repository, showing commits, trees, blobs, tags, branches, and their relationships.

## Features

- **Complete Object Visualization**: Displays all Git objects (commits, trees, blobs, tag objects)
- **Reference Tracking**: Shows local branches, remote branches, HEAD, and tag references
- **Branch Tracking**: Displays upstream tracking relationships with dashed arrows
- **Hierarchical Layout**: Objects organized by type at different levels:
  - **Top Level**: HEAD, tag references, tag objects
  - **Second Level**: Local and remote branches
  - **Third Level**: Commits
  - **Fourth Level**: Trees (at varying depths based on structure)
  - **Bottom Level**: Blobs (file contents)
- **Rich Relationship Visualization**: Shows all connections between objects with color-coded edges:
  - Tree relationships (green)
  - Parent commit relationships (red)
  - Tag references (goldenrod)
  - Branch tracking (dashed black)
  - Object references (purple)
- **Automatic SVG Generation**: Runs Graphviz `dot` automatically to produce SVG output
- **Edge Labels**: Filenames and directory names appear as labels on tree→blob/tree edges
- **Missing Reference Handling**: Shows dashed nodes for missing remote branches or non-existent upstream references

## Requirements

- Git 2.0+
- Python 3.6+
- Graphviz (for SVG rendering; optional if only DOT output is needed)

## Installation

1. Clone or download the script
2. Ensure Python 3.6+ and Git are installed
3. Optionally install Graphviz for automatic SVG rendering

## Usage

### Basic Usage (Default: Generate SVG)

```bash
python3 git_object_graph.py
```

This generates `objects.svg` in the current directory with the complete object graph visualization.

### Output DOT Format to Stdout

```bash
python3 git_object_graph.py --no-svg
```

Prints the Graphviz DOT format to stdout, useful for piping to other tools or saving manually.

### Save DOT File Only

```bash
python3 git_object_graph.py --no-svg > graph.dot
```

### Custom SVG Output Filename

```bash
python3 git_object_graph.py my_graph.svg
```

### Save Both DOT and SVG

```bash
python3 git_object_graph.py my_graph.svg my_graph.dot
```

## Node Types and Styles

### References (Top Level)
- **HEAD**: Red diamond with thick border – the current HEAD reference
- **Local Branches**: Pink diamond – local branch references
- **Remote Branches**: Purple diamond – remote tracking branches
- **Local Missing**: Pale pink dashed diamond – local branch referenced by HEAD but doesn't exist yet
- **Remote Missing**: Lavender dashed diamond – upstream remote branch that no longer exists
- **Tag References**: Yellow diamond – annotated tag references
- **Tag Objects**: Hot pink diamond – annotated tag objects

### Git Objects
- **Commits**: Gold ellipse – commit objects
- **Trees**: Light green folder – tree objects (directory snapshots)
- **Blobs**: Light blue note – blob objects (file contents)

## Edge Types and Styling

| Edge Type | Color | Style | Meaning |
|-----------|-------|-------|---------|
| **local** | Orange | solid | Branch points to its commit |
| **remote** | Brown | solid | Remote branch points to its commit |
| **tree** | Green | solid | Commit/tree references child tree or blob; labeled with filename/dirname |
| **parent** | Red | solid | Commit references parent commit |
| **head** | Red | solid (thick) | HEAD points to branch or commit |
| **tag** | Goldenrod | solid | Tag reference points to commit |
| **object** | Purple | solid | Tag object references another object |
| **tracks** | Black | dashed | Local branch tracks remote upstream |
| **tag (missing)** | Orange | dashed | Branch points to non-existent upstream |

## How It Works

1. **Repository Scanning**:
   - Lists all Git objects using `git cat-file --batch-check --batch-all-objects`
   - Discovers all local and remote branches
   - Finds HEAD reference (attached or detached)
   - Collects all tag references

2. **Name Discovery** (First Pass):
   - Parses all tree and tag objects to extract child names
   - Populates mapping of object hashes to filenames/directory names

3. **Object Processing** (Second Pass):
   - Creates nodes for each object with appropriate styling
   - Extracts all references between objects
   - Builds edges with relationship types

4. **Branch & Reference Processing**:
   - Creates branch reference nodes
   - Links branches to their commit targets
   - Identifies upstream tracking relationships
   - Creates placeholder nodes for missing references

5. **Graph Generation**:
   - Organizes nodes into rank levels by type
   - Adds ranking constraints to keep related branches adjacent
   - Applies Graphviz layout algorithm
   - Renders to SVG if requested

## Example Workflows

### Visualize a Repository

```bash
cd /path/to/git/repo
python3 /path/to/git_object_graph.py
# Opens objects.svg in your default viewer or editor
```

### Create a High-Resolution PNG

```bash
python3 git_object_graph.py --no-svg > graph.dot
dot -Tpng -Gdpi=300 graph.dot -o graph_hires.png
```

### Create PDF for Printing

```bash
python3 git_object_graph.py --no-svg > graph.dot
dot -Tpdf graph.dot -o graph.pdf
```

### Use with Custom Graphviz Options

```bash
python3 git_object_graph.py --no-svg > graph.dot
dot -Tsvg -Kneato graph.dot -o graph_neato.svg  # Use different layout engine
```

## Performance Notes

- **Small Repositories** (< 100 objects): < 1 second
- **Medium Repositories** (100–10,000 objects): 1–10 seconds
- **Large Repositories** (> 10,000 objects): 10 seconds to several minutes
  - Processing time depends on object count, tree depth, and branch count
  - Memory usage correlates with total object count

Progress is reported to stderr during processing.

## Troubleshooting

### "dot: command not found"
Install Graphviz or use `--no-svg` to skip SVG generation:
```bash
python3 git_object_graph.py --no-svg
```

### Visualization is too large or crowded
Use custom Graphviz options:
```bash
python3 git_object_graph.py --no-svg > graph.dot
dot -Tsvg -Kfdp graph.dot -o graph_fdp.svg  # Try different layout
```

### Missing branches or references
Ensure all branches are up-to-date:
```bash
git fetch --all
python3 git_object_graph.py
```

## File Structure

- `git_object_graph.py` – Main visualization script
- `README.md` – This file
- `LICENSE` – Project license

## Architecture

The `GitObjectGraphVisualizer` class handles:
- Git command execution and output parsing
- Object discovery and type identification
- Graph node and edge construction
- Graphviz DOT format generation
- Optional automatic SVG rendering via `dot` command

All output is non-destructive; the script only reads from the repository.

## Compatibility

- **Git**: 2.0+ (tested with Git 2.x)
- **Python**: 3.6+ (uses f-strings and subprocess features)
- **Graphviz**: Any version (dot command line tool)
- **Platforms**: Linux, macOS, Windows (with Git and Python installed)

## License

See LICENSE file for details.

