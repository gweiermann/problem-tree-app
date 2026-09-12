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

While held, a card moves freely and never pushes other cards. On release, the dropped card and its descendants animate toward a straighter horizontal chain. Nearby siblings join the animation only when proximity or collision requires them; distant branches remain untouched. A drop target activates when at least half the dragged card overlaps its card-sized magnetic region.

Unattached thoughts exist only in the inbox. Drag an attached node into the inbox to detach it, or drag an inbox thought onto a magnetic target. Detaching repairs the old chain by reconnecting direct children to the detached node's former parent.

Hover near a magnetic target to reveal its plus button. Clicking the plus creates and attaches a new thought at that exact relationship.

Touch input is first-class in this revision: drag cards and inbox thoughts directly, pan empty canvas space with one finger, and pinch with two fingers to zoom. On phone-sized screens the inbox becomes a horizontal bottom tray and controls use touch-sized hit areas.

Use the floating arrows or the URL's `variant` parameter to switch. The State button shows the complete relevant state after every action.

This code is intentionally disposable. It has no persistence, tests, production error handling, or production abstractions.
