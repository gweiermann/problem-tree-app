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

- `A` — Magnetic tree: recommended medium-strong settling.
- `B` — Loose magnets: weaker settling and more manual placement.
- `C` — Tidy magnets: fully automatic horizontal alignment.

While held, a card moves freely and never pushes other cards. On release, the tree straightens toward horizontal cause/consequence chains and resolves overlap. A drop target activates when at least half the dragged card overlaps its card-sized magnetic region.

Unattached thoughts exist only in the inbox. Drag an attached node into the inbox to detach it, or drag an inbox thought onto a magnetic target. Detaching repairs the old chain by reconnecting direct children to the detached node's former parent.

Hover near a magnetic target to reveal its plus button. Clicking the plus creates and attaches a new thought at that exact relationship.

Use the floating arrows or the URL's `variant` parameter to switch. The State button shows the complete relevant state after every action.

This code is intentionally disposable. It has no persistence, tests, production error handling, or production abstractions.
