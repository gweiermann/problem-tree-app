# Selection-first magnetic tree prototype

Throwaway UI prototype for the decision in **Prototype rapid capture, attachment, and reorganization**.

## Question

How should selection-based editing, unambiguous structural dragging, and constraint-assisted landscape layout work together?

## Run

```bash
python3 -m http.server 4173 --directory prototype/rapid-capture
```

Open `http://localhost:4173/?variant=A`.

All variants use the same direct-drag and selected-node control-bar model. They compare layout attraction strength:

- `A` — Magnetic tree: recommended gentle settling.
- `B` — Loose magnets: a very light nudge and more manual placement.
- `C` — Tidy magnets: stronger alignment without fully locking the layout.

Click a node to select it; click empty canvas to deselect it. Dragging on empty desktop canvas draws a selection rectangle. The explicit **Select area** tool enables the same rectangle with touch instead of panning, then returns to normal navigation after selection. Any number of nodes can be selected and removed atomically. Removing nodes repairs each affected chain by reconnecting direct children to the nearest surviving parent. The central problem can be selected and renamed but cannot be removed.

The full-width bottom bar uses one contextual text input. Rename is its default mode. **Add branch**, **＋ Cause**, or **＋ Consequence** switches that same input into creation mode; button confirmation or Enter adds the named branch. Root cause and consequence actions remain available regardless of selection. Creating a branch preserves the current selection and stays in creation mode for rapid repeated entry.

Keyboard shortcuts mirror the bar: `R` returns to rename mode, `A` opens add-branch mode, and `Delete` or `Backspace` removes all selected non-central nodes. Enter confirms the current input mode.

While held, the real zoomed card moves freely; there is no separately sized drag ghost. It never pushes other cards. On release, the dropped card and its descendants animate toward a straighter horizontal chain. Nearby siblings join only when proximity or collision requires them; distant branches remain untouched.

There are no hover plus buttons. Inactive structural targets remain invisible. Once at least half the card overlaps an actionable target, a labeled gray placeholder distinguishes **Insert between** from **Attach as branch**. Branch targets move away from occupied relationship gaps so they do not stack with edge-insertion targets.

Touching the central problem with a dragged card has hard priority over crossing edges: it always creates a new root branch, with the card's center determining cause (left) or consequence (right). Dropping a node back onto its current parent/side is deliberately not actionable.

When moving a subtree between cause and consequence sides, all pre-existing descendants rotate 180 degrees around the moved root before layout settling. The subtree therefore immediately grows in its new causal direction while preserving its internal shape.

When a target is actionable, the dragged card becomes translucent and shrinks slightly so the placeholder stays readable at any zoom. Touch remains usable for convenient testing: drag cards directly, pan empty canvas space with one finger, and pinch with two fingers to zoom. A dedicated portrait-mobile interaction model remains deferred to [Prototype portrait mobile problem-tree interactions](https://github.com/gweiermann/problem-tree-app/issues/16).

Desktop dragging uses an armed threshold: pointer-down remains a click until the pointer moves five pixels. Native text selection is suppressed when the gesture is armed, and move/up/cancel events are scoped to the initiating pointer.

Use the floating arrows or the URL's `variant` parameter to switch. The State button shows the complete relevant state after every action.

This code is intentionally disposable. It has no persistence, tests, production error handling, or production abstractions.
