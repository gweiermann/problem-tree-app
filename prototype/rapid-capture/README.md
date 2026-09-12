# Rapid capture interaction prototype

Throwaway UI prototype for the decision in **Prototype rapid capture, attachment, and reorganization**.

## Question

Which interaction model feels most fluid for capturing, attaching, rearranging, inserting, detaching, and undoing problem-tree thoughts across pointer and touch input?

## Run

```bash
python3 -m http.server 4173 --directory prototype/rapid-capture
```

Open `http://localhost:4173/?variant=A`.

Variants:

- `A` — Refined magnetic canvas: direct card dragging moves one card and can change structure.
- `B` — Handle-first: card dragging only arranges; a subtle handle changes structure.
- `C` — Inbox-led: capture and select in the side panel while retaining a movable canvas.

Across the variants, cards avoid overlap. Two-finger trackpad scrolling pans the viewport and pinch gestures zoom the canvas without browser zoom. Detaching a node repairs its former branch by reconnecting each direct child to the detached node's former parent.

Use the floating arrows or the URL's `variant` parameter to switch. The State button shows the complete relevant state after every action.

This code is intentionally disposable. It has no persistence, tests, production error handling, or production abstractions.
