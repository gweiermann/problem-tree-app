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

- `A` — Magnetic canvas: direct branch dragging performs spatial and structural work.
- `B` — Handle-first: card dragging arranges; a dedicated handle changes structure.
- `C` — Capture dock: capture into an inbox, then select and place.

Use the floating arrows or the URL's `variant` parameter to switch. The State button shows the complete relevant state after every action.

This code is intentionally disposable. It has no persistence, tests, production error handling, or production abstractions.

