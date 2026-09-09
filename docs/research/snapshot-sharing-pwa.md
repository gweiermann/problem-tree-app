# Snapshot sharing and offline GitHub Pages constraints

Research for [Research snapshot sharing and offline GitHub Pages constraints](https://github.com/gweiermann/problem-tree-app/issues/8), checked 2026-09-09. Sources are standards, browser-engine documentation/source, and GitHub's own documentation.

## Verdict

The MVP can be a static, offline-capable GitHub Pages app with snapshots entirely in the URL fragment. Use IndexedDB for trees, a versioned compressed fragment format for sharing, and a path-scoped service worker for the application shell. Do not claim a universal safe URL length or durable browser storage: neither exists. Measure the generated ASCII URL, warn progressively, keep copying available, and provide JSON file export/import as the dependable fallback.

The design should use these boundaries:

- A share URL is a frozen snapshot, not storage or synchronization.
- Browser storage is the working library, but remains user-clearable and potentially evictable.
- Cached application files make a previously loaded installation work offline; a first visit on an offline device cannot work.
- GitHub Pages is deployment only. No snapshot content is sent to it when the content lives after `#`.

## Recommended snapshot wire format

Use one explicit, versioned fragment form:

```text
https://gweiermann.github.io/problem-tree-app/#pt=v1.<base64url(zlib(canonical-json-utf8))>
```

The normative encode/decode pipeline should be:

1. Project the tree into a documented snapshot schema, excluding local-only fields such as library timestamps and undo state.
2. Canonicalize the JSON deterministically: fixed field order, stable node/edge order, no insignificant whitespace, and UTF-8 encoding.
3. Compute SHA-256 over those uncompressed canonical bytes for equality/deduplication.
4. Compress the canonical bytes with the zlib-wrapped `deflate` format.
5. Encode the compressed bytes as unpadded base64url and prefix them with `pt=v1.`.

The fragment must carry the schema/codec version before the opaque payload so future decoders can reject or migrate intentionally. Base64url's URL-safe alphabet is standardized in [RFC 4648 section 5](https://www.rfc-editor.org/rfc/rfc4648.html#section-5). `CompressionStream`/`DecompressionStream` standardize interoperable zlib-wrapped `deflate` in the [WHATWG Compression Standard](https://compression.spec.whatwg.org/#supported-formats). SHA-256 is available through `SubtleCrypto.digest()`; [Mozilla's API documentation](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/digest) records broad browser availability and the secure-context requirement.

Compression output is transport, not identity: conforming compressors may produce different valid byte streams for identical input. Compare the digest of the uncompressed canonical bytes, not the compressed fragment string. An exact digest match opens the existing local tree. A matching tree UUID with a different digest is **not** an update in this MVP; snapshot lineage/version replacement is deferred, so import it as a separate copy.

Native compression is viable for a modern-browser baseline. MDN's browser compatibility data currently records `CompressionStream` support from Chrome 80, Firefox 113, and Safari/iOS 16.4; `deflate-raw` starts later in Chromium, which is another reason to specify ordinary `deflate`. See the primary compatibility dataset at the pinned [`CompressionStream.json`](https://github.com/mdn/browser-compat-data/blob/2251e59becac361cdc2a04b9e3b6776c0357b225/api/CompressionStream.json). If the eventual supported-iPad decision includes Safari older than 16.4, ship one bundled zlib codec and use it for both encoding and decoding rather than creating a second wire format.

Decoding untrusted links must be bounded. Reject unknown versions, invalid base64url, decompression/checksum failures, payloads over a specified compressed size, output over a specified uncompressed size, invalid schema, cycles, multiple parents, missing central problem, duplicate IDs, and counts/text lengths above the product limits. Parse into a temporary value and only write to the library after validation and the user's save confirmation.

## Fragment behavior and practical URL limits

Fragments are the right location for this serverless snapshot. The [WHATWG URL Standard](https://url.spec.whatwg.org/#concept-url-fragment) defines the fragment as client-side URL data. HTTP's target URI expressly excludes it because fragments are reserved for client-side processing ([RFC 9110 section 7.1](https://www.rfc-editor.org/rfc/rfc9110.html#section-7.1)). Consequently GitHub Pages receives the path `/problem-tree-app/`, not the tree payload. This also avoids server-side deep-link routing: every snapshot loads the same static document.

That property is not confidentiality. The full URL remains visible to the browser, clipboard, history/bookmarks, extensions, screenshots, and any application to which the user pastes it. The product decision is to avoid an interrupting privacy warning, so UI copy must simply avoid calling a snapshot link private or encrypted.

There is no standards-defined portable maximum URL length. The URL grammar allows an arbitrary sequence of URL units in a fragment and gives no numeric ceiling ([WHATWG URL Standard](https://url.spec.whatwg.org/#url-writing)). Implementations differ: Chromium documents a 2 MiB accepted-URL limit and a 32 KiB omnibox display limit in its [URL display guidelines](https://chromium.googlesource.com/chromium/src/+/master/docs/security/url_display_guidelines/url_display_guidelines.md#URL-Length), while current WebKit source separately defines a 2 MiB maximum fragment in [`SecurityOrigin.cpp`](https://github.com/WebKit/WebKit/blob/main/Source/WebCore/page/SecurityOrigin.cpp). Neither figure guarantees successful passage through messaging, mail, QR, link-preview, or clipboard software.

Therefore treat URL size thresholds as product guardrails, not compatibility facts:

- Measure the final URL after compression and base64url. Because the payload alphabet is ASCII, byte and code-unit counts are effectively the same for that portion.
- Up to 8 KiB: normal copy-link flow.
- Above 8 KiB: keep copy available, show a non-blocking size caution, and present JSON file download as a sibling action.
- Above 32 KiB: label link portability as poor and make JSON the recommended action, but retain “Copy anyway.” A browser-engine ceiling is not a reason to pretend the link cannot work.
- Before release, run actual round-trip fixtures around 2, 8, 16, 32, and 64 KiB through every supported desktop/tablet browser and the sharing channels the team actually uses. Adjust the two provisional UX thresholds from those results without changing the wire format.

## Local library storage

Use IndexedDB as the source of truth for the tree library. IndexedDB stores structured-clone values transactionally and asynchronously; its specification is the [Indexed Database API](https://w3c.github.io/IndexedDB/). In contrast, Web Storage is synchronous and string-only; Mozilla documents the UI-blocking risk and recommends asynchronous alternatives for larger data in its [Web Storage API documentation](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API). Authoring fluidity rules out serializing and synchronously writing the entire library to `localStorage` on each edit.

`localStorage` may hold only tiny, disposable preferences if useful. It must not contain the authoritative trees. Published Web Storage quotas are small (Mozilla documents 5 MiB each for local and session storage), whereas IndexedDB quota is browser- and device-policy dependent. Every write can still fail with `QuotaExceededError`; handle it and surface a persistent “changes not saved” state rather than silently continuing. Browser storage starts in best-effort mode and can be evicted; `navigator.storage.estimate()`, `persisted()`, and `persist()` are the standard inspection/request mechanisms ([WHATWG Storage Standard](https://storage.spec.whatwg.org/)).

For tablet behavior specifically, WebKit states that Safari 17/iPadOS 17 fully supports the Storage API, assigns browser origins a quota of up to 60% of disk, still provides no guarantee that the quota is usable, and decides persistence requests heuristically, including whether the app runs as a Home Screen web app ([WebKit storage policy](https://webkit.org/blog/14403/updates-to-storage-policy/)). Persistent mode reduces automatic eviction; it cannot prevent the user from clearing site data. Private browsing and restrictive modes may disable or clear storage. A local-only product must retain explicit JSON export/import even when persistence is granted.

Write constraints for the later state specification:

- Commit an edit to memory immediately; enqueue the IndexedDB transaction without blocking pointer or keyboard feedback.
- Persist semantic commits, not pointer-move frames: text accept/blur, create/delete/reparent, and drag end.
- Serialize writes per tree or use revision checks so an older transaction cannot overwrite a newer one.
- Treat a successful IndexedDB transaction completion as saved; catch aborts and quota errors.
- Use a schema-versioned database and migration path. Do not rely on service-worker cache storage for user data.
- Request persistent storage as a progressive enhancement after meaningful user engagement; the app must remain correct when the method is unavailable or the request is denied.
- Expose file export because browser-managed storage cannot be promised as a backup.

## The GitHub Pages origin trap

GitHub documents a project site's default location as `https://<owner>.github.io/<repository>/` ([What is GitHub Pages?](https://docs.github.com/en/pages/getting-started-with-github-pages/what-is-github-pages#types-of-github-pages-sites)). Browser storage is partitioned by **origin**, not pathname. Therefore this app and every other public project site at `https://gweiermann.github.io/...` share the same localStorage namespace, IndexedDB origin, Cache Storage quota, and persistence decision.

Required mitigations:

- Name the database and caches with an app-specific prefix, for example `problem-tree-app:v1` and `problem-tree-app:shell:<build-id>`.
- Delete only caches carrying that prefix during upgrades. Chromium's service-worker lifecycle guidance explicitly warns about multiple apps sharing an origin and recommends app-specific cache prefixes ([service-worker lifecycle](https://web.dev/articles/service-worker-lifecycle#storage)).
- Remember that namespacing prevents accidental collisions, not same-origin access. Another script hosted under `gweiermann.github.io` can access the same origin's storage. A dedicated custom origin is required if path-level isolation later matters.
- Treat a future move from the default Pages URL to a custom domain as an origin migration: existing browser data will not follow automatically. Offer export/import before moving.

## Offline/PWA delivery on GitHub Pages

GitHub Pages is sufficient because it serves static HTML, CSS, and JavaScript ([GitHub Pages overview](https://docs.github.com/en/pages/getting-started-with-github-pages/what-is-github-pages)). All Pages sites support HTTPS, and `github.io` sites are served over HTTPS automatically ([GitHub HTTPS documentation](https://docs.github.com/en/pages/getting-started-with-github-pages/securing-your-github-pages-site-with-https)). That satisfies the secure-context requirement for service workers ([Service Workers specification](https://www.w3.org/TR/service-workers/#secure-context)).

Deploy as a project site, not as if hosted at `/`:

- Build every asset, manifest, icon, `start_url`, and registration URL under `/problem-tree-app/` (or derive them from the deployment base).
- Put the worker at `/problem-tree-app/sw.js` and scope it to `/problem-tree-app/`. A worker's default maximum scope is its containing directory unless the server broadens it with `Service-Worker-Allowed` ([Service Workers specification](https://www.w3.org/TR/service-workers/#service-worker-allowed)). GitHub Pages does not need a custom response header for the desired project-path scope.
- Set the web app manifest `id`, `start_url`, and `scope` explicitly and consistently under `/problem-tree-app/`, with a trailing slash on scope. The manifest standard explains the fallback and prefix-based scope rules ([Web Application Manifest](https://www.w3.org/TR/appmanifest/#scope-member)).
- Prefer fragment state over path-based SPA routes. Reloading `/problem-tree-app/#pt=...` requests the real `/problem-tree-app/` file, whereas arbitrary history-path routes require 404 workarounds on static hosting.

Precache the minimal application shell and content-hashed build assets during worker installation. For in-scope navigation, return the cached shell when offline; for versioned assets use cache-first because the names change with content. Keep user records in IndexedDB, not in the precache. A shared snapshot can be opened offline only if that origin/path was successfully loaded and controlled before the device went offline.

Do not activate a new worker in the middle of an editing session without coordination. Updated workers normally wait until the old worker no longer controls clients; the lifecycle is designed to keep one version active across a page ([Chromium service-worker lifecycle guidance](https://web.dev/articles/service-worker-lifecycle#waiting)). Detect a waiting worker, finish pending IndexedDB commits, then offer a reload. `skipWaiting()` plus forced reload risks changing application/schema versions while another tab still uses the old code.

Use a custom GitHub Actions build/deploy workflow for the chosen framework. GitHub recommends Actions for non-Jekyll custom builds ([Creating a GitHub Pages site](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site#static-site-generators)). Pages' 1 GiB published-site limit, 10-minute deployment timeout, and 100 GiB/month soft bandwidth limit are far above this app's expected shell but should remain CI constraints ([GitHub Pages limits](https://docs.github.com/en/pages/getting-started-with-github-pages/github-pages-limits)).

## Verification matrix for the implementation specification

The implementation-ready acceptance plan should include:

| Area | Required checks |
| --- | --- |
| Snapshot codec | Golden canonical-JSON fixtures; Unicode/emoji; deterministic digest; encode in each supported engine and decode in every other; corrupt/truncated/oversized payload rejection; unknown-version handling. |
| URL transport | Final lengths around 2/8/16/32/64 KiB; address-bar and clipboard round trips on desktop and iPad; intended messaging channels; fragment absent from a captured network request. |
| Import | Exact canonical digest opens existing tree; same UUID/different digest creates a copy; malformed input never mutates IndexedDB; saving requires confirmation. |
| Persistence | Reload/reopen; concurrent tabs; write abort and quota failure; storage denied/unavailable; user-cleared data; database migration; export/import recovery. |
| Offline | First online visit then airplane-mode reload; installed and ordinary browser modes on iPad; shared-fragment open while shell is cached; clean-device offline failure is explained. |
| Pages paths | Production build at `/problem-tree-app/`; direct reload; manifest launch; worker scope; no root-relative asset leaks; only app-prefixed caches are deleted. |
| Updates | Old controlled tab plus deployed update; waiting-worker UI; pending edit saved before reload; old database migrated once; two tabs on different app versions do not corrupt data. |

## Decisions this research does not make

The later decision tickets still need to set the supported browser/iPadOS floor, exact tree schema and canonical ordering, maximum node count/text length/uncompressed import size, provisional URL-warning copy, and autosave timing. Those are product choices. The constraints above keep each choice compatible with static GitHub Pages, offline use, and a future snapshot decoder.
