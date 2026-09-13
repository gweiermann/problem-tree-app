# Problem Tree Editor — Domain Context

## Product intent

The product is a local-first editor for rapidly externalising and organising a problem tree. Its defining quality is authoring fluidity: using it should feel easier and more enjoyable than reaching for pen and paper.

Desktop and tablet authoring are both first-class targets. Early prototypes may be validated on desktop first; physical tablet validation is deferred until a device is available. A dedicated portrait-phone interaction model is outside the current iteration. Local persistence, offline operation, and snapshot-link sharing support the core authoring experience. Agent-assisted editing is part of the product direction, but is lower priority than the human editing experience.

The first product specification covers a minimal local collection of trees.

## Working language

- **Problem tree**: The connected causal component rooted at the central problem. It is a strict tree with exactly one parent for every non-central attached item, no cross-links, and no cycles. Causes and consequences may form chains of any depth.
- **Central problem**: The single focal problem around which the structure is organised. It remains fixed at the absolute spatial centre of the canvas and may be renamed, but it is never moved, attached, detached, reparented, or deleted as an ordinary item.
- **Cause**: Something understood to contribute causally toward the central problem, directly or through another cause.
- **Consequence**: Something understood to result from the central problem, directly or through another consequence.
- **Branch-side relationship**: The relationship connecting a direct child to the central problem. It identifies the branch as a cause branch or consequence branch; every descendant derives its role from this first relationship.
- **Role**: The meaning of an attached item, derived from the branch-side relationship on its path to the central problem rather than stored as an intrinsic type or inferred from its coordinates. Reattaching a branch across the central divide preserves its structure while reinterpreting the causal direction and role of the entire branch.
- **Reparent**: Attach an item or branch to a different parent. Because the structure is a strict tree, completing a reparent replaces the previous parent relationship without a confirmation dialog.
- **Magnetic attachment target**: A card-sized relationship region that activates when most of a dragged item overlaps it, so attachment depends on the item's position rather than cursor precision. Inactive targets stay hidden during structural dragging; an actionable target appears as a card-sized placeholder. Targets sit outward along the horizontal causal direction; the central problem exposes a cause target on its left and a consequence target on its right.
- **Edge insertion**: Atomically placing an item or branch between two already connected items. The old relationship is removed, the inserted item takes its former place, and the former child becomes an additional child of the inserted item; existing descendants and identities are preserved.
- **Tree document**: One saved library entry containing the central problem, its connected problem tree, and its spatial arrangement.
- **Active document**: The tree document currently open on the canvas.
- **Canvas**: The finite spatial surface on which a tree document is arranged. Its central problem remains anchored while the viewport moves over it; its navigable extent reaches approximately half a viewport beyond the outermost item rather than continuing endlessly.
- **Viewport**: The visible, freely pannable and zoomable window onto the canvas.
- **Tree library**: The local collection of tree documents. It supports creating, opening, renaming, duplicating, deleting, and sharing documents, without folders, search, accounts, or synchronisation.
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
- Dedicated portrait-phone authoring, including vertical consequence/cause orientation and selection-based bottom actions.

## Interaction invariants

- Moving an item never changes its causal meaning by itself; semantic changes require an explicit successful structural operation.
- Attaching a branch to an attached item gives the whole branch the target's role.
- Reattaching a branch across the central divide preserves its structure and atomically reinterprets the entire branch as the target's role, without confirmation and with undo available.
- A structural drag is atomic. Until a valid attachment, reparenting, or edge-insertion succeeds, the prior relationships remain unchanged.
- Dropping outside a valid structural target only changes the attached item's manual position.
- Attaching, reparenting, and edge insertion preserve item identity, text, and descendants.
- Edge insertion removes the inserted branch from its former parent, preserves all existing descendants, and is rejected when it would introduce self-parenting or a cycle.
- Ordinary item deletion is immediate and undoable; it does not open a confirmation dialog. Deletion removes only the selected item: each direct child is promoted to the deleted item's former parent, and children promoted to the central problem inherit the deleted item's branch-side relationship. The central problem cannot be deleted.
- Deleting an entire tree from the library requires confirmation.
- Reparenting replaces the old parent relationship and remains undoable.
- Items move freely while dragged and never displace other items before release. After a drop, attached items never overlap: nearby items move aside without changing causal relationships.
- Attached items are magnetically attracted toward horizontal cause and consequence chains. After a drop, the dropped item and its descendants settle visibly toward straight parent-child lines. Nearby siblings join only when proximity or collision requires rearrangement; distant branches remain fixed.
