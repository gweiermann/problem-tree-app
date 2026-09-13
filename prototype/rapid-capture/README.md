# Magnetic tree and inbox prototype

Throwaway UI prototype for the decision in **Prototype rapid capture, attachment, and reorganization**.

## Question

How should inbox-only capture, overlap-aware attachment, and constraint-assisted landscape layout work together?

## Run

```bash
python3 -m http.server 4173 --directory prototype/rapid-capture
```

Open `http://localhost:4173/?variant=A`.

All variants use the same direct-drag model and permanent thought inbox. They compare layout attraction strength:

- `A` — Magnetic tree: recommended gentle settling.
- `B` — Loose magnets: a very light nudge and more manual placement.
- `C` — Tidy magnets: stronger alignment without fully locking the layout.

While held, a card moves freely and never pushes other cards. On release, the dropped card and its descendants animate toward a straighter horizontal chain. Nearby siblings join the animation only when proximity or collision requires them; distant branches remain untouched. A drop target activates when at least half the dragged card overlaps its card-sized magnetic region.

Unattached thoughts exist only in the inbox. Drag an attached node into the inbox to detach it, or drag an inbox thought onto a magnetic target. Detaching repairs the old chain by reconnecting direct children to the detached node's former parent.

During a drag, inactive targets remain invisible. Once at least half the card overlaps an actionable target, a gray card-sized **Drop here** placeholder appears.

On desktop only, moving the mouse near a relationship reveals a quiet plus button close to its parent card. Clicking it creates and attaches a new thought at that exact relationship. Coarse pointers do not show these plus buttons.

Every card, including the central problem, has a hover/focus rename button and also supports double-click rename.

Touch remains usable for convenient testing: drag cards and inbox thoughts directly, pan empty canvas space with one finger, and pinch with two fingers to zoom. A dedicated portrait-mobile interaction model is deferred to [Prototype portrait mobile problem-tree interactions](https://github.com/gweiermann/problem-tree-app/issues/16).

Desktop dragging uses an armed threshold: pointer-down remains a click until the pointer moves five pixels. Native text selection is suppressed when the gesture is armed, and move/up/cancel events are scoped to the initiating pointer.

Use the floating arrows or the URL's `variant` parameter to switch. The State button shows the complete relevant state after every action.

This code is intentionally disposable. It has no persistence, tests, production error handling, or production abstractions.
