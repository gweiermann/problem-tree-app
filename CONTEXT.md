# Problem Tree Editor — Domain Context

## Product intent

The product is a local-first editor for rapidly externalising and organising a problem tree. Its defining quality is authoring fluidity: using it should feel easier and more enjoyable than reaching for pen and paper.

Desktop and tablet authoring are both first-class targets. Early prototypes may be validated on desktop first; physical tablet validation is deferred until a device is available. Local persistence, offline operation, and snapshot-link sharing support the core authoring experience. Agent-assisted editing is part of the product direction, but is lower priority than the human editing experience.

The first product specification covers a minimal local collection of trees.

## Working language

- **Problem tree**: A strict causal tree with exactly one parent for every non-central item, no cross-links, and no cycles.
- **Central problem**: The focal problem around which the structure is organised. There is exactly one in the first version.
- **Cause**: Something understood to contribute causally toward the central problem, directly or through another cause.
- **Consequence**: Something understood to result from the central problem, directly or through another consequence.
- **Unplaced thought**: A neutral text item on the canvas that has not yet been attached to the problem tree. It supports capturing an idea before deciding its causal role.
- **Role**: The meaning of an attached item, derived from its path to the central problem rather than stored as an intrinsic type. Reattaching a branch across the central divide changes the role of the entire branch.
- **Reparent**: Attach an item or branch to a different parent. Because the structure is a strict tree, completing a reparent replaces the previous parent relationship without a confirmation dialog.
- **Magnetic attachment target**: A transient drop target shown near a valid parent while an item is dragged. Dropping there attaches or reparents the dragged branch.
- **Edge insertion**: Placing an item between two already connected items, replacing one relationship with two while preserving a strict tree.
- **Active tree**: The problem tree currently shown on the canvas.
- **Tree library**: The local collection of trees. It supports creating, opening, renaming, duplicating, deleting, and sharing trees, without folders, search, accounts, or synchronisation.
- **Snapshot link**: A shareable URL containing a frozen representation of a tree. It is not live collaboration.
- **Imported snapshot**: A snapshot opened from a link. Saving it to the tree library requires confirmation. If its canonical data is byte-identical to an existing tree, the existing tree is opened instead of creating a duplicate.
- **Authoring fluidity**: The degree to which capturing, changing, and reorganising ideas feels immediate and does not interrupt the user's train of thought.
- **Item text**: The sole content carried by a problem, cause, or consequence in the first version.
- **Tidy tree**: An explicit command that arranges the tree legibly without changing its causal meaning.
- **Tree name**: The library label derived from the central problem's text in the first version.

## Explicitly deferred concepts

- Freehand drawing, handwriting recognition, and stylus ink.
- Snapshot lineage, version comparison, version replacement, and preserved version history.
- Folders, search, accounts, cloud synchronisation, and real-time collaboration.
- Independently renaming a tree without changing the central problem.

## Interaction invariants

- Ordinary item and branch deletion is immediate and undoable; it does not open a confirmation dialog.
- Deleting an entire tree from the library requires confirmation.
- Reparenting replaces the old parent relationship and remains undoable.
- Items must not become hidden by overlap. The exact collision and displacement behavior remains a prototype decision.
