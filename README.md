# HexaJS Examples

This repository contains the example browser extension projects for [HexaJS](https://github.com/hexajs-dev/hexajs).

These examples are maintained as real framework consumers, not just snippets. They are used to validate that HexaJS package changes still build working projects with different UI setups and feature sets.

## Included Examples

| Example | Stack | Main functionality | Best for learning |
| --- | --- | --- | --- |
| `hexa-grayscale` | React, managed popup | Injects a floating page control that toggles full-page grayscale. | The smallest useful HexaJS content + view lifecycle. |
| `hexa-grayscale-vue` | Vue 3, managed popup | The same grayscale behavior implemented with Vue-driven managed UI. | How HexaJS architecture stays stable while the UI renderer changes. |
| `clip-volt` | React, managed popup, managed devtools | Captures copied text, stores it, filters it by privacy rules, and syncs it across extension surfaces. | Controllers, handlers, reducers, effects, and typed messaging working together. |
| `smart-clipper` | React, managed popup, managed devtools | Lets the user drag-select part of a page, run OCR, copy text, and inspect diagnostics. | A heavier real-world orchestration flow with services, ports, runtime sync, and workers. |

All examples include HexaJS build targets for `chrome`, `firefox`, `safari`, `opera`, `edge`, and `brave`.

## Recommended Reading Path

If you want to understand HexaJS progressively instead of jumping into the biggest project first, read the examples in this order:

1. `hexa-grayscale`
2. `hexa-grayscale-vue`
3. `clip-volt`
4. `smart-clipper`

That path moves from lifecycle and views, to framework-agnostic architecture, to stateful extension flows, and finally to a full orchestration-heavy product example.

## Example Deep Dive

### hexa-grayscale

This is the smallest example that still feels like a real extension. It injects a floating eye control into the page and toggles a grayscale filter over the current document. The popup is intentionally simple and acts as supporting UI, so the main lesson is the content-side experience.

- Main functionality: mount a content-side view, toggle UI state in the page, and cleanly remove everything when the content context is torn down.
- HexaJS core usage: [hexa-grayscale/src/background/main.ts](hexa-grayscale/src/background/main.ts) shows the minimal `@Background()` lifecycle. [hexa-grayscale/src/content/content.ts](hexa-grayscale/src/content/content.ts) shows `@Content`, `ContentRunAt`, and `@InjectView()` in the smallest possible shape. [hexa-grayscale/src/content/ui/grayscale-toggle/grayscale-toggle-view.ts](hexa-grayscale/src/content/ui/grayscale-toggle/grayscale-toggle-view.ts) shows the `@View` and `HexaView` pattern for mounting a UI component into the page while still owning page-level DOM effects.
- What developers should notice: the business behavior is not hidden in random browser callbacks. The content entrypoint is a class, the view is a class, and lifecycle is explicit through `onInit()` and `onDestroy()`.
- Start here: [hexa-grayscale/src/content/content.ts](hexa-grayscale/src/content/content.ts), [hexa-grayscale/src/content/ui/grayscale-toggle/grayscale-toggle-view.ts](hexa-grayscale/src/content/ui/grayscale-toggle/grayscale-toggle-view.ts), [hexa-grayscale/ui/popup/src/App.tsx](hexa-grayscale/ui/popup/src/App.tsx).

### hexa-grayscale-vue

This example keeps the same extension behavior as the React grayscale project, but swaps the managed UI layer to Vue. That makes it a strong proof that HexaJS is not tied to one frontend library for popup or view rendering.

- Main functionality: the same floating grayscale toggle, but rendered through Vue-managed components.
- HexaJS core usage: [hexa-grayscale-vue/src/background/main.ts](hexa-grayscale-vue/src/background/main.ts) and [hexa-grayscale-vue/src/content/content.ts](hexa-grayscale-vue/src/content/content.ts) keep the same background/content lifecycle model as the React example. [hexa-grayscale-vue/src/content/ui/grayscale-toggle/grayscale-toggle-view.ts](hexa-grayscale-vue/src/content/ui/grayscale-toggle/grayscale-toggle-view.ts) points a HexaJS view class at a Vue component, and [hexa-grayscale-vue/ui/popup/src/App.vue](hexa-grayscale-vue/ui/popup/src/App.vue) shows the same managed popup concept with Vue.
- What developers should notice: the HexaJS architecture barely changes when the renderer changes. You still organize behavior around background classes, content classes, views, and lifecycle hooks rather than around framework-specific glue.
- Start here: [hexa-grayscale-vue/src/content/content.ts](hexa-grayscale-vue/src/content/content.ts), [hexa-grayscale-vue/src/content/ui/grayscale-toggle/grayscale-toggle-view.ts](hexa-grayscale-vue/src/content/ui/grayscale-toggle/grayscale-toggle-view.ts), [hexa-grayscale-vue/ui/popup/src/App.vue](hexa-grayscale-vue/ui/popup/src/App.vue).

### clip-volt

ClipVault is the first example in this repository that feels like a real application with ongoing state, not just a UI toggle. It listens for copy events, captures clips, stores them in the background, applies privacy and domain rules, and keeps popup, content, and devtools surfaces synchronized.

- Main functionality: capture copied text from pages, persist clip history, configure storage and privacy rules, and inspect or remove clips from developer-facing tools.
- HexaJS core usage: [clip-volt/src/background/controller.ts](clip-volt/src/background/controller.ts) is the best controller example in the repo. It uses `@Controller` and `@Action` to expose typed background operations for config reads/updates and clipboard CRUD. [clip-volt/src/content/content.ts](clip-volt/src/content/content.ts) shows a content class that boots initial state from background APIs, listens to browser `copy` events, dispatches local actions, and forwards writes back to the background. [clip-volt/src/content/handler.ts](clip-volt/src/content/handler.ts) shows the handler side of the architecture, where broadcast messages are received and translated into store updates.
- State and effects: [clip-volt/src/background/store/background.reducer.ts](clip-volt/src/background/store/background.reducer.ts) models the background-side config and clip state. [clip-volt/src/content/store/content.reducer.ts](clip-volt/src/content/store/content.reducer.ts) keeps both raw and filtered clip collections in the content context. [clip-volt/src/content/store/content.effects.ts](clip-volt/src/content/store/content.effects.ts) is the clearest reducer/effect example here: it reacts to synced config and clip updates, then derives the filtered clip list based on domain scoping, exclusion rules, sensitivity filtering, expiry, and storage limits.
- UI surfaces: [clip-volt/ui/popup/src/App.tsx](clip-volt/ui/popup/src/App.tsx) shows a managed popup using `HexaUIClient` to fetch and update config. [clip-volt/ui/devtools/src/App.tsx](clip-volt/ui/devtools/src/App.tsx) shows a richer inspection surface that treats extension data like application state instead of an afterthought.
- What developers should notice: this is the strongest example if you want to understand how HexaJS turns extension messaging into explicit application layers: controller boundary, content boundary, handler boundary, store, reducer, effect, and UI clients.
- Start here: [clip-volt/src/background/controller.ts](clip-volt/src/background/controller.ts), [clip-volt/src/content/content.ts](clip-volt/src/content/content.ts), [clip-volt/src/content/handler.ts](clip-volt/src/content/handler.ts), [clip-volt/src/content/store/content.effects.ts](clip-volt/src/content/store/content.effects.ts), [clip-volt/ui/popup/src/App.tsx](clip-volt/ui/popup/src/App.tsx).

### smart-clipper

Smart Clipper is the most ambitious example in the repository. It lets the user start a clipping session from the popup or a keyboard shortcut, drag-select an area on the page, run OCR on the captured image, copy the result back to the clipboard, and inspect recent clips and failures in devtools.

- Main functionality: page selection, OCR orchestration, clipboard handoff, recent-result history, theme syncing, diagnostics, and error reporting.
- HexaJS core usage: [smart-clipper/src/background/controller.ts](smart-clipper/src/background/controller.ts) is a strong example of a background controller acting as the orchestration boundary for the whole extension. It uses `@Action` methods to coordinate browser tabs, runtime messaging, storage, command shortcuts, capture services, OCR services, and devtools sync. [smart-clipper/src/content/handler.ts](smart-clipper/src/content/handler.ts) shows how a content handler can inject views and turn runtime messages into actual UX, including a clipping overlay, progress UI, OCR completion, and clipboard fallback behavior. [smart-clipper/ui/popup/src/App.tsx](smart-clipper/ui/popup/src/App.tsx) shows a managed popup using `HexaUIClient` for typed background requests and `RuntimePort` for live cross-surface updates. [smart-clipper/ui/devtools/src/hooks/useDevtoolsData.ts](smart-clipper/ui/devtools/src/hooks/useDevtoolsData.ts) shows devtools consuming background state plus live runtime sync updates.
- Architecture lesson: unlike ClipVault, this example is not primarily about reducers and effects. Its value is showing that HexaJS also works well when the center of gravity is service orchestration, browser ports, multi-step workflows, and background-content-devtools coordination.
- What developers should notice: this is where HexaJS starts to feel like an application framework for extensions rather than a build tool. The popup, content overlay, background controller, devtools surface, and OCR pipeline are still connected through typed boundaries instead of ad-hoc message passing.
- Start here: [smart-clipper/src/background/controller.ts](smart-clipper/src/background/controller.ts), [smart-clipper/src/content/handler.ts](smart-clipper/src/content/handler.ts), [smart-clipper/ui/popup/src/App.tsx](smart-clipper/ui/popup/src/App.tsx), [smart-clipper/ui/devtools/src/hooks/useDevtoolsData.ts](smart-clipper/ui/devtools/src/hooks/useDevtoolsData.ts).

## Relationship To The Main HexaJS Repo

These projects depend on workspace packages such as `@hexajs-dev/cli`, `@hexajs-dev/core`, `@hexajs-dev/common`, `@hexajs-dev/ports`, and `@hexajs-dev/ui`.

Because of that, this repository is validated against the main [hexajs-dev/hexajs](https://github.com/hexajs-dev/hexajs) monorepo rather than as a standalone install.

## CI Validation

GitHub Actions validates every example with [.github/workflows/validate-examples.yml](.github/workflows/validate-examples.yml).

The workflow:

1. Checks out this repository.
2. Checks out `hexajs-dev/hexajs`.
3. Copies each example into `hexajs/examples`.
4. Installs the HexaJS workspace dependencies.
5. Builds the HexaJS packages.
6. Builds each example with its default Chrome target.
7. Builds each example again with `production:chrome`.

This makes sure framework changes and example changes still produce working builds together.

## Local Validation

To reproduce the CI flow locally:

1. Clone [hexajs-dev/hexajs](https://github.com/hexajs-dev/hexajs).
2. Sync the folders from this repository into `hexajs/examples`.
3. Run the workspace install and builds from the HexaJS repo root.

Example commands:

```bash
pnpm install --frozen-lockfile
pnpm run build:packages
pnpm --filter "./examples/clip-volt" run build
pnpm --filter "./examples/clip-volt" run production:chrome
```

Repeat the filtered example build commands for the other projects as needed.

## License

MIT. See [LICENSE](LICENSE).