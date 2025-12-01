# Git Tree Visualizer

A Python script that creates a Graphviz visualization of Git tree structures.

## Features

- **Recursive Tree Crawling**: Recursively traverses Git tree objects using `git cat-file -p`
- **Tree/Blob Distinction**: Different visual styles for tree objects (directories) and blob objects (files)
  - Tree objects: Green folder icons
  - Blob objects: Blue note icons
- **Graphviz Output**: Generates DOT format files compatible with Graphviz tools

## Requirements

- Git
- Python 3.6+
- Graphviz (for rendering the output files)

## Usage

```bash
# Display output to stdout
./git_tree_visualizer.py HEAD^{tree}

# Save to a DOT file
./git_tree_visualizer.py HEAD^{tree} tree.dot

# Or use the script without making it executable
python3 git_tree_visualizer.py HEAD tree_output.dot
```

## Rendering the Visualization

Once you have a `.dot` file, you can render it using Graphviz:

```bash
# Generate PNG image
dot -Tpng tree.dot -o tree.png

# Generate PDF
dot -Tpdf tree.dot -o tree.pdf

# Generate SVG
dot -Tsvg tree.dot -o tree.svg

# View in a supported format
dot -Tpng tree.dot | display  # if you have ImageMagick
```

## How It Works

1. **Reference Resolution**: Converts the Git reference (e.g., `HEAD^{tree}`) to its commit hash
2. **Type Verification**: Verifies the reference points to a tree object
3. **Tree Parsing**: Uses `git cat-file -p` to read and parse tree contents:
   - Each line contains: `mode type hash\tname`
   - Example: `100644 blob abc123\tfile.txt`
4. **Recursive Traversal**: For each tree entry:
   - Creates a node in the graph
   - For tree objects, recursively processes them
   - Tracks visited nodes to avoid infinite loops
5. **Graph Generation**: Produces a Graphviz DOT file with styled nodes and edges

## Example

```bash
# In a Git repository
python3 git_tree_visualizer.py HEAD~5^{tree} history_tree.dot
dot -Tpng history_tree.dot -o history_tree.png
```

## Node Styles

- **Tree Nodes (Directories)**: Green folder-shaped nodes
- **Blob Nodes (Files)**: Blue note-shaped nodes
- **Root Node**: The entry point tree object
- **Edges**: Gray lines showing directory/file relationships

## Git References Supported

Any valid Git reference that resolves to a tree object:
- `HEAD^{tree}` - The tree of the current commit
- `main^{tree}` - The tree of the main branch
- `v1.0.0^{tree}` - The tree of a tagged commit
- `abc1234^{tree}` - The tree of a specific commit
- Or any commit hash directly (if it's a commit object)

---

# Git Object Graph Visualizer

A Python script that creates a Graphviz visualization of all Git objects in a repository.

## Features

- **Complete Object Scanning**: Uses `git rev-list --all --objects` to find all objects
- **Multi-type Support**: Visualizes commits, trees, blobs, and tag objects
- **Relationship Mapping**: Shows all references between objects:
  - Commits → Trees (green)
  - Commits → Parent Commits (red)
  - Trees → Blobs/Trees (gray)
  - Tags → Objects (purple)
- **Distinct Node Styles**: Each object type has a unique color and shape
- **Progress Reporting**: Reports progress while processing objects

## Usage

```bash
# Save to a DOT file (recommended for large repositories)
./git_object_graph.py all_objects.dot

# Display to stdout
./git_object_graph.py all_objects.dot > output.txt
```

## Node Types and Styles

- **Commit Objects**: Gold boxes - represent commits in the repository
- **Tree Objects**: Green folders - represent directory snapshots
- **Blob Objects**: Blue notes - represent file contents
- **Tag Objects**: Pink diamonds - represent annotated tags
- **Object Hashes**: First 8 characters of the SHA-1 hash displayed as label

## Edge Types and Colors

- **tree** (Green): Commit references its tree
- **parent** (Red): Commit references parent commit(s)
- **blob/tree** (Gray): Tree references child blobs or trees
- **object** (Purple): Tag references an object

## Example

```bash
# In a Git repository
python3 git_object_graph.py repo_objects.dot
dot -Tpng repo_objects.dot -o repo_objects.png

# Create a large-format visualization
dot -Tpng -Gsize=20,20 repo_objects.dot -o repo_objects_large.png

# Create an SVG for interactive viewing
dot -Tsvg repo_objects.dot -o repo_objects.svg
```

## Performance Notes

- **Small Repositories**: Fast, typically processes in seconds
- **Medium Repositories**: May take 10-30 seconds to scan and process
- **Large Repositories**: Can take minutes; object count is reported during processing
- **Memory**: Graph size depends on number of objects; large repositories may need significant memory

## How It Works

1. **Object Discovery**: Scans all git objects using `git rev-list --all --objects`
2. **Type Identification**: Determines each object's type (commit, tree, blob, tag)
3. **Reference Extraction**: For each object:
   - **Commits**: Extract tree reference and parent commit(s)
   - **Trees**: Extract all child blob/tree references
   - **Tags**: Extract object reference
4. **Graph Construction**: Creates nodes for each object and edges for references
5. **Graph Generation**: Produces a Graphviz DOT file with colored nodes and labeled edges

