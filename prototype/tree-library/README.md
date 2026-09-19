# Tree library and shared-snapshot prototype

Throwaway UI prototype for **Prototype the tree library and shared-snapshot flow**.

## Question

How should the chosen canvas drawer support everyday library management, and how should frozen shared links transition into local trees?

## Run

```bash
python3 -m http.server 4174 --directory prototype/tree-library
```

Open `http://localhost:4174/?variant=A`.

## Variants

- `A` — **Canvas drawer**: the selected direction. A collapsible library drawer shares the viewport with the active tree and preserves spatial context.
- `B` — **Focus drawer**: an overlay drawer dims the canvas. More focus, but more interruption.
- `C` — **Library workspace**: the library temporarily replaces the canvas. Clearest library mode, but the strongest context switch.

At tablet widths the library becomes a bottom sheet. Widths below 650px deliberately show an unsupported message: phone interaction belongs to a separate prototype.

## Scenarios to test

1. Open and close the library while working on a tree.
2. Create a tree from its central problem.
3. Open, independently rename, duplicate, share, and confirm deletion of a saved tree.
4. Drag a handle to reorder the library. The picked-up row follows the pointer, its original slot stays visible, neighboring rows glide into place, and the row settles into its new slot on release.
5. Choose **Simulate new shared link**. Confirm that the tree opens full-canvas in a clearly marked **Shared with you** mode. The arrival toast stays for 3.9 seconds; the mode badge remains until you leave the shared tree.
6. Choose **Edit** or **Save** and confirm that both add the shared tree to the library. Edit continues on the canvas; Save opens the library and highlights the new entry.
7. Choose **Simulate existing shared link** and confirm it directly opens and highlights the identical local tree—without showing the shared mode or creating a duplicate.

The snapshot link, persistence, and identical-tree detection are simulated. No IndexedDB, compression, clipboard write, service worker, or real file export is implemented.
