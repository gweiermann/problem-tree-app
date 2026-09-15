# Tree library and shared-snapshot prototype

Throwaway UI prototype for **Prototype the tree library and shared-snapshot flow**.

## Question

How much should opening the local tree library interrupt the active canvas, and how should frozen shared snapshots transition between preview, temporary exploration, deduplication, and saving?

## Run

```bash
python3 -m http.server 4174 --directory prototype/tree-library
```

Open `http://localhost:4174/?variant=A`.

## Variants

- `A` — **Canvas drawer**: a collapsible library drawer shares the viewport with the active tree. Recommended starting point because it preserves spatial context.
- `B` — **Focus drawer**: an overlay drawer dims the canvas. More focus, but more interruption.
- `C` — **Library workspace**: the library temporarily replaces the canvas. Clearest library mode, but the strongest context switch.

At tablet widths the library becomes a bottom sheet. Widths below 650px deliberately show an unsupported message: phone interaction belongs to a separate prototype.

## Scenarios to test

1. Open and close the library while working on a tree.
2. Create a tree from its central problem.
3. Open, duplicate, share, and delete a saved tree.
4. Preview a new shared snapshot without storing it.
5. Explore a snapshot temporarily, then save it from the canvas banner.
6. Switch the snapshot test to **Already saved** and confirm it routes to the existing local tree instead of duplicating it.
7. Compare A/B/C for interruption and orientation, not visual polish.

The snapshot link and persistence are simulated. No IndexedDB, compression, clipboard write, service worker, or real file export is implemented.
