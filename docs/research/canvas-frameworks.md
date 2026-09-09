# Canvas frameworks for fluid desktop and tablet authoring

Research date: 2026-09-09  
Decision ticket: [Research canvas frameworks for fluid desktop and tablet authoring](https://github.com/gweiermann/problem-tree-app/issues/3)

## Verdict

Use **React + TypeScript + React Flow (`@xyflow/react`)** as the default implementation stack, with layout kept behind an application-owned adapter. React Flow is the lowest-risk choice for this product's hardest interactions: it has first-party examples for touch connections, proximity-based linking, collision resolution, and ELK layout, plus unusually explicit keyboard and screen-reader support.

**Vue 3 + TypeScript + Vue Flow (`@vue-flow/core`) is viable, not disqualified.** Its public API and source cover the required primitives: custom HTML nodes, controlled state, node-drag lifecycle events, intersection queries, touch/pinch handling, and baseline keyboard accessibility. It is a reasonable override if a short interaction prototype proves equally good on touch. The risk is not a missing fundamental primitive; it is the smaller maintenance surface and the lack of equivalent first-party examples for the product-specific interaction recipes.

Do not build the canvas from scratch, and do not begin with tldraw, Cytoscape.js, or JointJS. They either solve a broader whiteboard/diagramming problem than this strict-tree editor needs or move important ready-made editor features into a commercial tier. Reconsider only if the chosen Flow library fails the interaction prototype.

## What was evaluated

The decision is about the renderer/editor substrate, not the domain model. The application must remain the authority for strict-tree invariants, causal roles, undoable commands, persistence, and snapshot encoding.

The comparison weights the risky requirements most heavily:

1. One-finger node movement and two-finger pan/zoom on tablets.
2. Custom, editable sticky-note nodes rendered as ordinary DOM.
3. Drag lifecycle access for magnetic parent targets, reparent previews, edge insertion, and unlink zones.
4. Controlled graph state so invalid cycles/multiple parents never become canonical state.
5. Layout integration without surrendering manual positions.
6. Keyboard and screen-reader foundations.
7. Active maintenance, static deployment, and a permissive license.

## Comparison

| Criterion | React Flow 12.11.6 | Vue Flow 1.48.2 | Implication |
|---|---|---|---|
| Custom note UI | React components via `nodeTypes`; HTML inputs and controls are normal node content | Vue components via `node-types`/slots; node data is reactive | Parity for sticky notes |
| Controlled state | Controlled `nodes`, `edges`, `onNodesChange`, `onEdgesChange`; validation hooks and delete interception | `apply-default=false` allows application-owned change handling; `v-model` can sync internal state | Both can enforce a strict tree; keep the domain store separate |
| Drag/reparent primitives | Drag start/move/stop events, internal-node lookup, intersection utilities, reconnect API | Drag start/move/stop hooks, `findNode`, `getIntersectingNodes`, coordinate helpers | Both are sufficient; magnetic behavior is application code |
| Tablet input | Dedicated touch example; pinch/pan options and configurable drag thresholds | Source handles touch events; documented pinch and tap options; configurable drag threshold | Both plausibly work, but physical-device testing remains mandatory |
| Accessibility | Dedicated guide: focusable nodes/edges, Tab navigation, Enter/Space selection, arrow movement, live announcements, configurable/localizable ARIA text | Source includes focusable node/edge wrappers, keyboard movement, ARIA descriptions, and a live region; text is less configurable/documented | React Flow has the stronger supported contract |
| Layout ecosystem | First-party examples for Dagre, D3, ELK, dynamic layout, and collisions | First-party Dagre example; arbitrary positions make ELK integration straightforward but custom | React Flow reduces discovery/prototype cost |
| Product-specific recipes | First-party proximity-connect and rectangle-collision examples closely match the intended UX | Necessary primitives exist, but equivalent official recipes were not found | React Flow has materially lower implementation risk |
| Maintenance signal | MIT; monorepo actively maintained; React package release 12.11.6 published 2026-09-01 | MIT; latest release 1.48.2 published 2026-01-28; latest default-branch commit inspected was 2026-06-23 | Both active enough for an MVP; React Flow has stronger current momentum |

## Evidence

### React Flow

- The core component supports controlled nodes, edges, viewport, drag thresholds, custom node/edge types, reconnect radius, and node drag events. These are the interception points needed to preview a valid parent while dragging and commit one domain command on drop: [React Flow component API](https://reactflow.dev/api-reference/react-flow).
- The official [touch-device example](https://reactflow.dev/examples/interaction/touch-device) demonstrates touch connection behavior and explicitly enlarges handles for tap targets. This does **not** prove the proposed custom gestures feel good on an iPad; it does establish that touch is a supported input path rather than an accidental browser fallback.
- The official [proximity-connect example](https://reactflow.dev/examples/nodes/proximity-connect) computes the nearest node during `onNodeDrag`, renders a temporary candidate edge, and commits it on `onNodeDragStop`. That maps closely to magnetic parent targets, though the app must add strict-tree validation, direction rules, hysteresis, and larger touch zones.
- The official [node-collision example](https://reactflow.dev/examples/layout/node-collisions) resolves rectangular overlaps after a drag. Its algorithm is a useful prototype seed for pushing unplaced thoughts, not a production physics guarantee.
- React Flow documents built-in Tab focus, Enter/Space selection, arrow-key movement, automatic focus panning, ARIA roles, live announcements, and customizable accessibility strings: [accessibility guide](https://reactflow.dev/learn/advanced-use/accessibility).
- React Flow deliberately does not own graph layout. Its [layout overview](https://reactflow.dev/learn/layouting/layouting) provides integrations and characterizes ELK as the most configurable and most complex option. The official [ELK tree example](https://reactflow.dev/examples/layout/elkjs) demonstrates the integration boundary.
- The project is MIT-licensed and maintained in the [xyflow repository](https://github.com/xyflow/xyflow). The examined React release was [`@xyflow/react@12.11.6`](https://github.com/xyflow/xyflow/releases/tag/%40xyflow%2Freact%4012.11.6), published 2026-09-01.

### Vue Flow

- Vue Flow documents built-in dragging, zoom/pan, selection, custom nodes and edges, graph helpers, controls, and a minimap: [Vue Flow introduction](https://vueflow.dev/guide/).
- Custom Vue components can be registered as node types, and node state can be mutated through Vue's reactive model: [node guide](https://vueflow.dev/guide/node.html).
- The app can disable default mutations with `apply-default=false` and apply its own accepted change set, or synchronize nodes/edges with `v-model`: [controlled-flow guide](https://vueflow.dev/guide/controlled-flow.html). This is enough to reject cycles and multiple-parent operations before committing them.
- Vue Flow documents pinch zoom, tap/double-click zoom, pan-on-drag, scrolling behavior, drag thresholds, draggable/connectable flags, and viewport bounds: [configuration reference](https://vueflow.dev/guide/vue-flow/config.html).
- At commit [`17953c3`](https://github.com/bcakmakoglu/vue-flow/commit/17953c329db2d5dc5f5097370cbf5e1173bab520), the source contains touch handlers on handles and the viewport, node drag hooks, `getIntersectingNodes`, keyboard-focusable node/edge wrappers, ARIA descriptions, and an assertive live region. Relevant source entry points are [Handle.vue](https://github.com/bcakmakoglu/vue-flow/blob/17953c329db2d5dc5f5097370cbf5e1173bab520/packages/core/src/components/Handle/Handle.vue), [Viewport.vue](https://github.com/bcakmakoglu/vue-flow/blob/17953c329db2d5dc5f5097370cbf5e1173bab520/packages/core/src/container/Viewport/Viewport.vue), [store actions](https://github.com/bcakmakoglu/vue-flow/blob/17953c329db2d5dc5f5097370cbf5e1173bab520/packages/core/src/store/actions.ts), and [A11yDescriptions.vue](https://github.com/bcakmakoglu/vue-flow/blob/17953c329db2d5dc5f5097370cbf5e1173bab520/packages/core/src/components/A11y/A11yDescriptions.vue).
- Vue Flow has no built-in layout system. Its official [layout example](https://vueflow.dev/examples/layout/simple.html) uses Dagre and writes calculated positions back to the nodes. The same adapter pattern can host ELK.
- The project is MIT-licensed in the [Vue Flow repository](https://github.com/bcakmakoglu/vue-flow). The examined release was [`v1.48.2`](https://github.com/bcakmakoglu/vue-flow/releases/tag/v1.48.2), published 2026-01-28.

### Layout engine

Start the interaction prototype with a simple deterministic tree layout (Dagre or a small custom subtree layout), but define the adapter so ELK can replace it. The strict problem tree is unusual: causes and consequences form two opposing rooted trees around one central node, while unplaced thoughts must remain outside layout and avoid collisions.

ELK's layered algorithm arranges directed nodes in layers, minimizes crossings, supports ports and multiple routing styles, and exposes extensive spacing and model-order controls: [ELK Layered reference](https://eclipse.dev/elk/reference/algorithms/org-eclipse-elk-layered.html) and [layout options](https://eclipse.dev/elk/reference/options.html). ELK only computes geometry; it does not render the graph, which is the separation this app needs: [ELK overview](https://eclipse.dev/elk/).

Recommended layout boundary:

```ts
type LayoutInput = {
  rootId: string
  connectedNodes: Array<{ id: string; width: number; height: number }>
  parentByChild: Record<string, string>
  previousPositionById: Record<string, { x: number; y: number }>
}

type LayoutOutput = Record<string, { x: number; y: number }>
```

Run the cause and consequence halves separately, mirror one half around the central axis, then translate the central problem back to its pre-layout anchor. Collision resolution for unplaced thoughts is a second pass owned by the app. This prevents a renderer or layout library from becoming the persisted domain format.

## Alternatives screened out

- **tldraw** provides an excellent general canvas, custom shapes, and persistent shape bindings: [custom shapes](https://tldraw.dev/examples/shapes/tools/custom-shape) and [bindings](https://tldraw.dev/sdk-features/bindings). For this MVP those abstractions are broader than needed, while strict causal-tree validation, directed-tree layout, parent-target previews, and semantic keyboard navigation would still be custom work. It is React-only, so it does not preserve the Vue preference.
- **JointJS** is capable and framework-agnostic; its open-source core provides SVG graph primitives, and its React integration can render controllable HTML nodes: [official introduction](https://docs.jointjs.com/) and [`HTMLHost`](https://docs.jointjs.com/api-react/Components/HTMLHost/). However, the official comparison states that advanced layout and ready-made UI components are JointJS+ features. Choosing it would add lower-level graph/paper concepts or a commercial dependency without a clear benefit for this focused tree editor.
- **Cytoscape.js** is an excellent graph visualization/analysis library, but its [documented style model](https://js.cytoscape.org/#style) centers on renderer-managed graph-element properties rather than arbitrary editable DOM components. That is the wrong center of gravity for text-first sticky notes and direct manipulation.
- **A bespoke SVG/HTML canvas** would avoid framework coupling but would require rebuilding pan/zoom, selection, coordinate conversion, drag semantics, edge hit areas, focus management, and accessibility. Those are precisely the risky parts the MVP should buy rather than invent.

## Prototype gate before the stack is irreversible

The recommendation is intentionally conditional on interaction evidence. Build one throwaway React Flow slice first, using no production persistence or visual polish. It should include:

1. Create an unplaced thought by double-click/double-tap.
2. Drag it near a valid parent and show a stable magnetic target preview.
3. Drop to attach; drag an attached subtree to another parent to reparent.
4. Drop in a neutral zone to detach.
5. Insert a node by dropping onto an edge target.
6. Pinch-zoom and pan without accidentally moving or editing a note.
7. Undo every committed structural operation as one step.
8. Resolve overlaps after drop without moving unrelated notes unpredictably.

Test first with browser touch emulation, then on physical iPad-class hardware when available. Emulation cannot settle tap-target comfort, palm/gesture conflicts, virtual-keyboard behavior, or perceived magnetic hysteresis.

If the React Flow slice fails for a library-specific reason, reproduce only that failing slice in Vue Flow before considering a lower-level alternative. If both pass, retain React Flow because its examples and accessibility contract reduce delivery risk; choose Vue Flow only if developer fluency produces a clearly better-feeling prototype with no touch regression.

## Consequences for later decisions

- Persist domain nodes and parent relationships, not React Flow/Vue Flow node objects.
- Route all mutations through application commands so undo, autosave, snapshot import, and WebMCP use the same invariant checks.
- Treat magnetic reparenting, unlink zones, edge insertion, subtree previews, and collision response as product interactions implemented above the canvas library.
- Keep every touch target at least as large as the eventual input-contract decision requires; library defaults are not acceptance criteria.
- Static GitHub Pages hosting creates no special constraint for either library: both are client-side packages bundled into ordinary static assets. PWA details belong to the separate offline/sharing research.

## Confidence and remaining unknowns

Confidence is **high** that React Flow and Vue Flow both supply the necessary primitives, and **medium-high** that React Flow is the lower-risk default. Documentation and source review cannot establish fluidity. The deciding unknowns are the custom gesture design, collision behavior, performance with realistic trees, and real iPad browser behavior; those belong in the interaction prototypes and physical-device validation, not in further framework comparison.
