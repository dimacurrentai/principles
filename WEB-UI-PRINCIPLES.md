# Web UI, UX, and Data Principles

> A living document. `ENG-PRINCIPLES.md` §4 states the general web doctrine — functional, neat-looking, minimalistic; this file expands it into UI, UX, and data rules for our web applications, document-style local apps in particular. The house maxim — **local-first, offline-first, CRDT-first** — is enforced here, not merely admired (§5, §7). Each principle is stated as an imperative so it can be checked, enforced, and argued about. Product features appear only as examples; packdiff's review player is the running one. Section references here (§N) name this file's own sections; references into the sibling are spelled `ENG-PRINCIPLES §N`.

---

## 1. The interface is a document

- **The app is an offline-first document, not a smaller imitation of a hosted site.** Present content as a human-readable sequence of named sections (packdiff: Description, Commits, Files changed, Diff), readable top to bottom without interaction.

- **ALWAYS put a plain-language summary above the navigation.** The summary scrolls away; the single navigation row stays pinned.

- **Navigation names exact destinations.** It scrolls and moves focus to the relevant item, not merely to a nearby section — a link that lands "nearby" makes the reader search again.

- **Explicit navigation creates browser history; passive scrolling does not.** Scrolling updates the URL with `replaceState`, so copied links stay useful without polluting Back.

- **Back and Forward restore view changes, never data mutations.** Meaningful view changes — including selection ranges — are restored; mutations are never replayed.

## 2. Stability is a feature

- **Nothing jumps.** A layout shift steals the reader's place — and sometimes their click. Pinned chrome stays one row and controls reserve stable space; menus, messages, and disclosures overlay or scroll away.

- **Preserve DOM identity across updates.** Mutate an existing element in place; recreate it only when its identity truly changes or its implementation cannot accept the required update. Replacement destroys browser-owned state — selection, focus, fullscreen, playback, and scroll position — and makes stable content jump.

- **Context populates reserved space without changing geometry.** Pinned chrome may fill as the user scrolls, but its dimensions never move.

- **Stabilize the point of a direct, local interaction** when opening or closing inline UI. Do not fight ordinary scrolling with global compensation.

- **A disabled action stays in place**, grayed out with an explanation, rather than vanishing (packdiff: Side-by-side when horizontal space is insufficient). A control that disappears reads as a feature that never existed.

- **Success feedback NEVER resizes controls.** Respect reduced motion.

## 3. Prefer direct, obvious interaction

- **The whole meaningful region is the interaction target.** A small icon or gutter may reinforce the affordance but must never be the only doorway — a gutter-sized target turns interaction into a precision exercise (packdiff: the whole valid diff line accepts a comment; the `+` gutter only reinforces).

- **Direct manipulation produces the result directly** — no redundant Apply step (packdiff: dragging across commits selects and renders that range on release).

- **An interactive status performs its most useful action** rather than opening intermediary UI (packdiff: a file's comment count jumps to its first comment).

- **Every programmatic landing gets a subtle, temporary arrival cue.** A jump the user cannot see landing reads as a jump that did nothing.

- **Paths are navigable breadcrumbs.** Ancestors are actionable, stay on one line, and reveal with horizontal touch or trackpad scrolling. NEVER replace navigable information with a destructive ellipsis.

## 4. Progressive disclosure, not clutter

- **Every persistent control must justify its space and attention.** Prefer strong defaults, direct manipulation, and contextual controls over permanent chrome.

- **Secondary metadata appears at its relevant location** through restrained disclosure — persisted and discoverable, not permanently on screen.

- **Do not fill empty states with redundant labels.** Reserve layout space when stability requires it, but an empty state needs no visible zero-count label (packdiff: zero comments shows no `0 comments`).

- **Keep rich models beneath minimal interfaces.** Full functionality does not mean showing everything at once.

- **Avoid duplicate navigation.** When a content section already indexes the document, a permanent sidebar restating it is clutter (packdiff: Files changed is the file index).

## 5. Local-first, offline-first, CRDT-first

- **Local-first, offline-first, CRDT-first.** The app is fully functional with no server and no network; the local model is the one source of truth; and anything that later synchronizes merges *into* that model deterministically (§7), never replaces it. Every bullet in this section and §6–§7 exists to keep this maxim true.

- **ALWAYS identify a document by a hash of its canonical content** — never by filename, timestamp, title, or presentation. Names and rendering change; content identity must not.

- **Identical content restores identical local state** after rename, move, regeneration, or restart. Different content starts with clean state; state is never fuzzily moved between documents.

- **Durable state survives restart:** user-authored data and deliberate presentation choices alike. Local storage is today's mechanism, not the product contract.

- **Exports are today's safe push paths** (packdiff: `Copy as Markdown`, `Copy as JSON`). Future upstreams extend the local model rather than replace it.

## 6. Visible data is an integrity invariant

- **User-authored data may be collapsed but NEVER made invisible.** A collapsed container shows counts of what it holds; expanding reveals the data in place (packdiff: a collapsed file shows its comment and draft counts).

- **Drafts reopen visibly after restart.** A view that excludes user data reports how much is outside it and offers a direct route to the full view.

- **Warnings and errors NEVER reflow the document.** They may collapse to a visible status indicator, but never disappear without acknowledgement; unrelated work stays available whenever safe.

- **Sensitive mutations confirm inline and are reversible.** Deletion in particular requires deliberate confirmation and belongs to a persistent per-document undo history.

## 7. Two tiers of durable state

- **Material changes are auditable, undoable, and future-pushable:** the data the user came to produce (packdiff: comments, viewed status, deletions/restorations, and future shared events).

- **Local state is durable for recovery and comfort but not upstream material:** drafts, collapse state, theme, wrapping, and similar preferences.

- **Defaults start expanded; user state overrides defaults.** Deliberate view state is user data. Externally authored defaults may change the starting point; they never override the user's explicit choice.

- **Both tiers are preserved and inspectable.** Any pinned change count reports material changes only.

- **Model material changes as stable-ID journal operations, not whole-document replacement.** This is the CRDT-first leg of §5's maxim: journal operations merge deterministically when synchronization arrives, without implementing it prematurely.

## 8. Fast and accessible by construction

- **Local filtering, sorting, navigation, and view changes feel instant** and execute client-side (JavaScript/WASM), never round-tripping to a server.

- **Pointer, keyboard, touch, and assistive technology expose the same actions.** Shortcuts accelerate functionality; they never own it exclusively.

- **Navigation moves both viewport and focus.** Focus treatment is visible and layout-neutral.

- **Unmodified single-key shortcuts stand down in every typing context.** Teach important shortcuts where their actions live, summarize them in one discoverable place, and show platform-appropriate Command/Control notation.

- **Meaning NEVER depends on color alone.** Not every reader perceives color; pair it with shape, text, or position.

## 9. Render from data, with explicit state lifetimes

- **Derive every visible state from application data.** Keep selection, inspection targets, drafts, and current records in the model or explicit UI state; never read classes, ARIA attributes, or element existence back as application state. Use one rendering path for structure and focused paint functions for frequent feedback, both projecting the same data.

- **Compute derivatives when needed.** Recompute affordability, highlights, and selection validity from current data instead of storing values that can go stale. Express whether an action is allowed as a predicate on that data, not a separately maintained flag.

- **Give ephemeral state an owner and an end.** Clear selection, hover previews, focus targets, and drafts at every transition that invalidates them, in one place per transition. Persist unfinished drafts when recovery requires it (§7); their lifetime still ends explicitly when committed or discarded.

- **Persist the model, then reconstruct the screen.** Restore saved actions, drafts, and deliberate view choices through the same rendering path used during normal operation.

- **Attach asynchronous results to stable record identity.** Match replies to their record and revision; reject stale results and drop results for deleted records. Never update a list position captured before an asynchronous operation.

## 10. Updates must remain visually consistent

- **Rendering identical data twice produces identical pixels and preserves browser-owned state.** Prefer updates that preserve DOM identity (§2). When rebuilding is unavoidable, capture and restore inner scroll positions, keyboard focus, and open dialogs by stable data identity; keep a dialog open unless its subject no longer exists.

- **Key interactive elements by data identity, never list position.** Restore the user's keyboard focus with `focus({ preventScroll: true })`, preserving focus-visible treatment without introducing mouse-triggered focus rings or scrolling.

- **Paint frequent feedback in place.** Update selection counts, labels, badges, and `aria-pressed` on existing nodes; reserve structural rendering for transitions that require it.

- **Keep reserved boxes when hiding their contents.** Use `visibility: hidden` when a disappearing score, badge, or travelling piece must retain its layout slot.

- **Show a consistent stage throughout a transition.** Do not expose the final state before its pieces arrive or leave an obsolete state visible after the transition has advanced. Model staged views explicitly.

- **Transient feedback expires and identifies its actor.** Time out highlights and mark only the records the relevant actor changed. Persistent warnings and errors follow §6 instead.

- **Give each visual meaning its own component-scoped vocabulary.** Do not reuse a state class for unrelated components with different meanings.

## 11. Bound dynamic content and isolate motion

- **Reserve space for changing values.** Give counters, captions, scores, status lines, and badges stable columns or minimum sizes, tabular figures, and bounded rows. Truncate incidental text within its column; preserve navigable paths as required by §3.

- **Growing content scrolls inside a bounded region.** Lists must not lengthen their row or widen their column. Use size containment where necessary to prevent content from determining the surrounding layout.

- **Use `minmax(0, 1fr)` for flexible grid tracks containing variable content.** Give grid and flex children `min-width: 0` and `min-height: 0` so long content cannot push static regions off screen.

- **Animate outside layout.** Use fixed-position overlay clones with `pointer-events: none` and transform/opacity animation. Cancel measured motion on resize or page hiding before its coordinates become stale.

- **Float dialogs and previews above the layout.** Opening them never displaces the primary surface. Updates elsewhere preserve the user's panel scroll, inspection, and reading position.

## 12. Plan space for each window shape

- **Choose layouts by shape as well as width.** Consider orientation, aspect ratio, and height for wide landscape, short landscape, tablet, and portrait arrangements.

- **Reseat the same components with named grid areas.** Keep one DOM and one state model across layouts instead of maintaining layout-specific copies.

- **Give the primary surface flexible space and bound supporting panels.** Use flexible tracks for the main surface and bounded tracks such as `clamp()` for supporting regions.

- **Size content from its container's tighter constraint.** Use container queries and `aspect-ratio` so content fits both available height and width without distorting its shape.

- **For desktop workspaces, fit essential controls within one screen and scroll regions internally.** Use a `100dvh` shell; switch to natural page scrolling on phones. Document-style applications retain the top-to-bottom reading and scrolling model in §1.

- **Remove decoration before function when space runs out.** Hide secondary captions or help text before controls or essential information; keep the help available through disclosure.

- **Let CSS handle layout changes.** Resize may recompute or cancel measured transient visuals; it never changes application data.

## 13. Make interruptions and verification part of the design

- **Interrupt motion and replay cleanly.** A new interaction cancels the sequence and lands in a consistent state. Apply the same action-validity predicate to every control that can mutate state while work is busy.

- **Reduced motion preserves behavior.** Skip flights and delays under `prefers-reduced-motion`; computation and state changes remain identical.

- **Accessible state is the same state as visual state.** Write ARIA attributes, disabled states, and focusability from the same data in the same paint.

- **Automate browser checks for silent regressions.** Cover several window shapes, horizontal overflow, motion and rapid interruption, selection isolation, draft cancellation, resize cleanup, modal preservation, undo, storage recovery, reduced motion, and console errors. Encode applicable principles in checks so later features cannot silently break them.
