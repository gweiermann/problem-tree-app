# WebMCP feasibility for a local-first problem-tree canvas

Research date: 2026-09-09

## Verdict

**A small backend-free proof is feasible and is a good fit for this app, but WebMCP should remain an experimental enhancement rather than an MVP dependency.** A top-level page can register JavaScript tools whose `execute` callbacks read the app's live client-side store or call the same mutation commands used by the human interface. Those commands can update the canvas, append to the undo stack, and persist to `localStorage`; WebMCP itself does not require a server. The W3C draft explicitly describes WebMCP pages as MCP-like servers implemented in client-side script, and its tool callback runs in the owning page. ([WebMCP specification](https://webmachinelearning.github.io/webmcp/#introduction), [WebMCP explainer](https://github.com/webmachinelearning/webmcp#detailed-design))

This is not yet a portable, generally available web capability. WebMCP is an actively changing proposal. Chrome documents an origin trial beginning with Chrome 149 and a local-development flag, while OpenAI currently exposes a subset as **Site tools** in the ChatGPT desktop app's built-in browser. ([Chrome WebMCP overview](https://developer.chrome.com/docs/ai/webmcp), [OpenAI Site tools](https://learn.chatgpt.com/docs/webmcp))

## Why no backend is needed

The imperative API registers a name, description, JSON Schema, and an `execute` JavaScript callback through `document.modelContext.registerTool()`. The browser mediates a call and invokes that callback on the page; the callback may call existing client-side application logic and return a structured result. Chrome's own example calls a UI mutation function directly, and the WebMCP explainer describes execution as client-side logic on the live page. ([Chrome imperative API](https://developer.chrome.com/docs/ai/webmcp/imperative-api), [WebMCP explainer](https://github.com/webmachinelearning/webmcp#lifecycle-of-a-tool-call))

For the problem-tree app, the adapter can therefore sit beside the state store:

```ts
const context = document.modelContext;

await context?.registerTool({
  name: "get_problem_tree",
  description: "Read the open problem tree and its current revision.",
  inputSchema: {
    type: "object",
    properties: {},
    additionalProperties: false,
  },
  annotations: {
    readOnlyHint: true,
    untrustedContentHint: true,
  },
  execute: async () => treeStore.exportForAgent(),
});

await context?.registerTool({
  name: "apply_tree_patch",
  description: "Atomically stage a batch of edits to the open problem tree.",
  inputSchema: {
    type: "object",
    properties: {
      expectedRevision: { type: "integer", minimum: 0 },
      operations: {
        type: "array",
        minItems: 1,
        maxItems: 100,
        items: { $ref: "#/$defs/operation" },
      },
    },
    required: ["expectedRevision", "operations"],
    additionalProperties: false,
    $defs: {
      operation: {
        oneOf: [
          /* add thought, update text, attach/reparent branch, detach, delete branch */
        ],
      },
    },
  },
  annotations: {
    readOnlyHint: false,
    untrustedContentHint: false,
    consequentialHint: false,
  },
  execute: async (input) => treeCommands.stageAtomicPatch(input),
});
```

The exact operation union belongs in the later tool-contract decision. The feasibility-critical point is that `stageAtomicPatch` should be the same domain command used by the UI—not a parallel mutation path. It should validate the strict-tree invariants, update state and visible UI synchronously, persist through the normal autosave path, and create one undo unit. Chrome recommends updating the interface before reporting completion and validating strictly in application code because schema constraints are not guaranteed. ([Chrome WebMCP best practices](https://developer.chrome.com/docs/ai/webmcp/best-practices))

## Recommended proof

Expose only two static tools at first:

1. `get_problem_tree()` returns the open tree's revision, root, nodes, parent relationships, and unplaced thoughts. It should omit viewport geometry unless the agent task needs it.
2. `apply_tree_patch(expectedRevision, operations[])` stages an atomic set of additions, text edits, attachments/reparents, detaches, and branch deletions for review.

Use `expectedRevision` as optimistic concurrency control. Reject a stale revision and return the new revision so the agent rereads instead of overwriting human edits made while it was reasoning. Limit operation count and text length. Return a compact result such as `{ revision, added, updated, moved, detached, deleted, proposalId }`, without echoing all tree text.

The proof passes when, in a supported agent browser:

- `get_problem_tree` returns an edit made manually moments earlier;
- one `apply_tree_patch` call adds and reorganizes several nodes;
- the visible canvas updates before the tool reports success;
- one undo reverses the whole batch;
- a stale `expectedRevision` is rejected without partial mutation;
- the accepted result survives reload through `localStorage`; and
- browsers without `document.modelContext` retain the complete human UI with no errors.

Test registration, schema parsing, direct calls, results, and errors first with Chrome's Model Context Tool Inspector. Then run a real selection/invocation test in the ChatGPT desktop built-in browser, because inspector success does not establish that an OpenAI agent will select the tool reliably. Chrome explicitly positions the inspector for viewing, manually executing, and natural-language testing of registered tools. ([Chrome WebMCP overview](https://developer.chrome.com/docs/ai/webmcp#imitate_agent_chat_with_the_inspector_extension))

## Current constraints

### OpenAI implementation

- Site tools are currently discovered by ChatGPT Work and Codex in the **ChatGPT desktop app's built-in browser**. They belong to the live page and can disappear when it is closed or navigated away from. This rules out unattended or closed-page access; a conventional MCP server would be needed for that. ([OpenAI Site tools](https://learn.chatgpt.com/docs/webmcp))
- OpenAI currently directs users to GPT-5.6 Sol or GPT-5.6 Terra. GPT-5.6 Luna has WebMCP disabled. Site tools are unavailable in Enterprise and Edu workspaces, and availability also depends on rollout and app version. ([OpenAI Site tools](https://learn.chatgpt.com/docs/webmcp))
- OpenAI's browser currently supports only a subset: declarative form tools and tools registered inside either same-origin or cross-origin iframes are not discovered. Register imperatively with JavaScript in the top-level page. ([OpenAI Site tools limitations](https://learn.chatgpt.com/docs/webmcp#limitations))
- The built-in browser safety-reviews every call and applies its normal access/confirmation rules. Tool metadata is a hint, not a grant of authority. ([OpenAI Site tools security](https://learn.chatgpt.com/docs/webmcp#security-and-user-controls))

### Browser and platform

- Chrome currently exposes the feature through an origin trial from Chrome 149 and through `chrome://flags/#enable-webmcp-testing` for local development. Chrome says the API remains under active discussion and may change. Do not assume ordinary Safari, Firefox, mobile, or embedded webviews implement it merely because the site itself runs there. ([Chrome WebMCP overview](https://developer.chrome.com/docs/ai/webmcp))
- The draft API is a secure-context API. Chrome additionally requires origin isolation and disables WebMCP where `document.domain` is enabled; its `tools` Permissions Policy defaults to `self`. GitHub Pages supplies HTTPS, but the proof should still feature-detect the API and verify headers/behavior on the deployed origin. ([WebMCP specification](https://webmachinelearning.github.io/webmcp/#modelcontext-interface), [Chrome security and permissions](https://developer.chrome.com/docs/ai/webmcp#security_and_permissions))
- The specification does not prescribe how a browser exposes registered tools to its agent. Different agents may use MCP, proprietary function calling, or another mechanism, so browser-specific end-to-end tests are unavoidable. ([WebMCP specification: agent observation](https://webmachinelearning.github.io/webmcp/#browser-agent-observation))
- WebMCP enables offline client-side state access only in the narrow technical sense that the page callback needs no backend. It does **not** make the cloud-backed agent available offline. The app's human editing and PWA behavior must remain independent of agent availability.

## Security and privacy consequences for this app

“Local-first” no longer means “data stays on the device” during an agent interaction. Returning a tree to an agent discloses that content to the agent/browser system even though the app has no backend. The tool surface should be opt-in or visibly enabled, and the agent contract should avoid accessing unopened trees in the local library.

Treat tree text as untrusted because users can paste externally authored content containing prompt injection. Mark the read tool with `untrustedContentHint: true`; never interpolate tree text into tool descriptions; and keep tool results structured. The W3C draft and Chrome guidance both identify tool metadata/results and user-generated content as prompt-injection surfaces. ([WebMCP security considerations](https://webmachinelearning.github.io/webmcp/#security), [Chrome secure-tools guidance](https://developer.chrome.com/docs/ai/webmcp/secure-tools))

Use static, precise descriptions and a small surface. Chrome warns that more and overlapping tools consume agent context and make tool selection harder. It also recommends concise metadata/results, currently suggesting no more than 500 characters per tool description and 1.5K characters per individual result. ([Chrome WebMCP best practices](https://developer.chrome.com/docs/ai/webmcp/best-practices), [Chrome secure-tools guidance](https://developer.chrome.com/docs/ai/webmcp/secure-tools#set_character_budgets))

For the first proof, do not expose tree-library deletion, share-link copying, navigation, or import. Stage mutations for human review, keep them undoable, and reuse all domain validation. Under that design, `apply_tree_patch` is a write tool but not a significant real-world or non-reversible action, so `consequentialHint: false` is reasonable; the browser may still apply its own confirmation policy. If later tools publish data or permanently delete it, reassess and likely set `consequentialHint: true`. The specification defines annotations as behavioral hints, and OpenAI explicitly says declarations do not prove what a tool actually does. ([WebMCP tool annotations](https://webmachinelearning.github.io/webmcp/#modelcontexttool-dictionary), [OpenAI Site tools security](https://learn.chatgpt.com/docs/webmcp#security-and-user-controls))

## Decision consequence

Proceed with a two-tool proof after the canvas store and mutation command layer exist. Keep the adapter small, feature-detected, top-level, and removable. The proof should validate the tool contract and real OpenAI-agent behavior, but production architecture and core UX must not depend on WebMCP until availability and interoperability mature.
