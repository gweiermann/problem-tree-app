# Problem Tree Editor — Domain Context

## Product intent

The product is a local-first editor for rapidly externalising and organising a problem tree. Its defining quality is authoring fluidity: using it should feel easier and more enjoyable than reaching for pen and paper.

Desktop and tablet authoring are both first-class targets. Early prototypes may be validated on desktop first; physical tablet validation is deferred until a device is available. Local persistence, offline operation, and snapshot-link sharing support the core authoring experience. Agent-assisted editing is part of the product direction, but is lower priority than the human editing experience.

The first product specification covers a minimal local collection of trees.

## Working language

- **Problem tree**: A strict causal tree with exactly one parent for every non-central item, no cross-links, and no cycles. Causes and consequences may form chains of any depth.
- **Central problem**: The single focal problem around which the structure is organised. It remains fixed at the absolute spatial centre of the canvas and may be renamed, but it is never moved, attached, detached, reparented, or deleted as an ordinary item.
- **Cause**: Something understood to contribute causally toward the central problem, directly or through another cause.
- **Consequence**: Something understood to result from the central problem, directly or through another consequence.
- **Unplaced thought**: A neutral text item on the canvas that has not yet been attached to the problem tree. It supports capturing an idea before deciding its causal role.
- **Unplaced branch**: A disconnected causal structure created when an attached item with descendants is detached. Its internal parent relationships and item identities remain intact, but the branch has no cause or consequence role until it is reattached to the problem tree.
- **Branch-side relationship**: The relationship connecting a direct child to the central problem. It identifies the branch as a cause branch or consequence branch; every descendant derives its role from this first relationship.
- **Role**: The meaning of an attached item, derived from the branch-side relationship on its path to the central problem rather than stored as an intrinsic type or inferred from its coordinates. Reattaching a branch across the central divide preserves its structure while reinterpreting the causal direction and role of the entire branch.
- **Reparent**: Attach an item or branch to a different parent. Because the structure is a strict tree, completing a reparent replaces the previous parent relationship without a confirmation dialog.
- **Magnetic attachment target**: A transient drop target shown near a valid parent while an item is dragged. Dropping there attaches or reparents the dragged branch.
- **Detachment zone**: A thick, viewport-fixed border that becomes visible while an attached item or branch is dragged. Dropping into it deliberately detaches the branch; ordinary dragging elsewhere only changes layout.
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

- Moving an item never changes its causal meaning by itself; semantic changes require an explicit successful structural operation.
- A structural drag is atomic. Until a valid attachment, reparenting, edge-insertion, or detachment drop succeeds, the prior relationships remain unchanged.
- Dropping outside a valid structural target only changes the item's manual position. Dropping an attached branch into the visible detachment zone removes its connection to the problem tree, preserves its internal structure and current position, and makes it an unplaced branch.
- Attaching, detaching, reparenting, and edge insertion preserve item identity, text, descendants, and manual position unless the user separately changes them.
- Ordinary item and branch deletion is immediate and undoable; it does not open a confirmation dialog.
- Deleting an entire tree from the library requires confirmation.
- Reparenting replaces the old parent relationship and remains undoable.
- Items must not become hidden by overlap. The exact collision and displacement behavior remains a prototype decision.
