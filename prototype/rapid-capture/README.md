# Selection-first magnetic tree prototype

Throwaway UI prototype for the decision in **Prototype rapid capture, attachment, and reorganization**.

## Question

How should selection-based editing, unambiguous structural dragging, and constraint-assisted landscape layout work together?

## Run

```bash
python3 -m http.server 4173 --directory prototype/rapid-capture
```

Open `http://localhost:4173/?variant=A`.

All variants use the same direct-drag, selection-first editing model. They compare layout attraction strength:

- `A` — Magnetic tree: recommended gentle settling.
- `B` — Loose magnets: a very light nudge and more manual placement.
- `C` — Tidy magnets: stronger alignment without fully locking the layout.

Click a node to select it; click empty canvas to deselect it. On tablet and desktop, the selected node exposes local **Add child**, **Add parent**, and **Rename** actions. The central problem instead offers explicit **Add cause** and **Add consequence** actions. Double-clicking any node, pressing `R`, or pressing Enter starts an inline rename. Confirming the name returns focus and selection to the node that opened the action, restoring its quick actions instead of triggering another rename.

The full-width bottom action bar is smartphone-only. It remains a blank white surface when nothing is selected. A selected node initially shows only **Rename**, **Add**, and **Remove**. Each command moves the bar into a focused state: a rename form, an add relationship choice followed by a naming form, or a removal confirmation. Creating a child or parent preserves the current selection, so repeated branch creation stays anchored to the original node.

Keyboard shortcuts mirror the actions: `A` adds a child, `P` inserts a parent, `C`/`E` add a cause/consequence when the center is selected, and `Delete` or `Backspace` removes all selected non-central nodes. During inline editing, Enter confirms and Escape cancels. Outside editing, `Home` selects the central problem, Left/Right move toward or away from the center according to the branch direction, and Up/Down move between siblings. `[` and `]` switch magnetic-strength variants.

Dragging on empty desktop canvas draws a selection rectangle. The explicit **Select area** tool enables the same rectangle with touch instead of panning, then returns to normal navigation after selection. Any number of nodes can be selected and removed atomically. Removing nodes repairs each affected chain by reconnecting direct children to the nearest surviving parent. The central problem can be selected and renamed but cannot be removed.

While held, the real zoomed card moves freely; there is no separately sized drag ghost. It never pushes other cards. On release, the dropped card and its descendants animate toward a straighter horizontal chain. Nearby siblings join only when proximity or collision requires them; distant branches remain untouched.

There are no hover plus buttons. Inactive structural targets remain invisible. Once at least half the card overlaps an actionable target, one persistent labeled gray placeholder distinguishes **Insert between** from **Attach as branch**. It animates only when the structural candidate changes, so ordinary pointer movement over the same target remains visually steady. Branch targets move away from occupied relationship gaps so they do not stack with edge-insertion targets.

Touching the central problem with a dragged card has hard priority over crossing edges: it always creates a new root branch, with the card's center determining cause (left) or consequence (right). Dropping a node back onto its current parent/side is deliberately not actionable.

When moving a subtree between cause and consequence sides, all pre-existing descendants rotate 180 degrees around the moved root before layout settling. The subtree therefore immediately grows in its new causal direction while preserving its internal shape.

When a target is actionable, the dragged card becomes translucent and shrinks slightly so the placeholder stays readable at any zoom. Touch remains usable for convenient testing: drag cards directly, pan empty canvas space with one finger, and pinch with two fingers to zoom as far out as 18%. Mobile text-entry states survive the viewport resize caused by opening the on-screen keyboard. A dedicated portrait-mobile interaction model remains deferred to [Prototype portrait mobile problem-tree interactions](https://github.com/gweiermann/problem-tree-app/issues/16).

Desktop dragging uses an armed threshold: pointer-down remains a click until the pointer moves five pixels. Native text selection is suppressed when the gesture is armed, and move/up/cancel events are scoped to the initiating pointer.

Use the floating arrows or the URL's `variant` parameter to switch. The State button shows the complete relevant state after every action.

This code is intentionally disposable. It has no persistence, tests, production error handling, or production abstractions.
