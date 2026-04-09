

# AGENTS.md

## Tech Stack
- **Language:** TypeScript with strict mode (`strict: true`, `noUncheckedIndexedAccess`, all strict family checks enabled). All files `.ts`/`.tsx`.
- **Framework:** Next.js 14+ with App Router, static export (`output: 'export'`). No server runtime.
- **Styling:** Tailwind CSS with custom theme, purged in production.
- **Target:** ES2017+ JavaScript output in `tsconfig.json`.

### npm Dependencies (exact pin unless noted)
- `markdown-it@13.0.1` + `@types/markdown-it` + `markdown-it-footnote@3.0.3`
- `marked@^9.0.0` (caret range)
- `katex@0.16.9` (exact pin)
- `highlight.js@11.9.0` (exact pin)
- `mermaid@^9.4.3` (caret range)
- `jspdf@2.5.1`, `html2canvas@1.4.1` (exact pins)

### CDN-loaded
- MathJax 2.7.7 via `cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.7/MathJax.js?config=TeX-AMS_HTML`
- Load in root `layout.tsx` `<head>` via Next.js `<Script strategy="beforeInteractive">`
- Include SRI `integrity` attribute (SHA-384/SHA-512) and `crossorigin="anonymous"`

## Architecture

### Project Structure
```
src/
├── app/                    # Next.js App Router (pages & layouts)
│   ├── layout.tsx          # Root layout with providers
│   └── page.tsx            # Main editor page
├── features/
│   ├── editor/             # Markdown input pane (textarea, autosave, paste detection)
│   ├── preview/            # Rendered HTML preview
│   ├── toolbar/            # Engine selectors, export dropdown, CSS toggle, mobile menu
│   ├── export/             # PDF/MD/TXT download logic
│   └── custom-css/         # Custom CSS panel and injection
├── shared/
│   ├── components/         # Reusable UI primitives (buttons, button groups, dropdowns)
│   ├── hooks/              # useLocalStorage, useMediaQuery, etc.
│   ├── providers/          # React context providers (engine selection, theme state)
│   └── styles/             # Global styles, design tokens, CSS variables
└── lib/
    ├── markdown/           # Engine abstraction: markdown-it & marked behind common interface
    ├── math/               # Engine abstraction: MathJax & KaTeX behind common interface
    ├── mermaid/            # Mermaid rendering wrapper
    └── highlight/          # highlight.js wrapper
```

### Key Patterns
- **Engine Abstraction Layer** in `lib/`: Wrap markdown-it/marked and MathJax/KaTeX behind common interfaces. Switching engines is a state change, not a code change.
- **React Context** for global state: engine selections, mobile view mode, custom CSS state. Do NOT use an external state library (no Redux, no Zustand, no Jotai).
- **Dynamic imports** for heavy client-only libraries: load KaTeX, Mermaid, highlight.js, jsPDF, html2canvas via `next/dynamic` or `React.lazy`. Eager-load only markdown-it.
- **`'use client'`** directives on pages/components using browser APIs (editor, preview, math rendering).
- **localStorage persistence** via a `useLocalStorage` custom hook. Use exact keys: `markdownEditorContent` and `markdownEditorCustomCSS`. Wrap all localStorage access in `try/catch`, fall back to in-memory state.
- Render user HTML via `dangerouslySetInnerHTML` with post-processing via refs.
- This is effectively a single-page app (`/`). Next.js provides the shell, layout system, and build tooling.

### MathJax v2 Configuration
Preserve exactly in a `text/x-mathjax-config` block:
```javascript
MathJax.Hub.Config({
  showProcessingMessages: false,
  tex2jax: { inlineMath: [['$','$'],['\\(','\\)']] },
  TeX: { equationNumbers: { autoNumber: "AMS" } },
  "HTML-CSS": { linebreaks: { automatic: true } },
  SVG: { linebreaks: { automatic: true } }
});
```
- Create `types/mathjax.d.ts` declaring the global `MathJax` object with typed `Hub.Config`, `Hub.Queue`, `InputJax.TeX`.
- `lib/math/mathjax.ts` provides `typesetElement(el: HTMLElement): Promise<void>`.
- Maintain a MathJax queue guard to prevent concurrent typesetting.

### Engine Configurations (preserve from source)
- **markdown-it:** `html: true, linkify: true, typographer: true`
- **marked:** `gfm: true, pedantic: false`
- **Mermaid:** `startOnLoad: false, theme: 'default'`
- **KaTeX auto-render delimiters:** `$$`, `$`, `\\[`, `\\(`

## Migration Approach
- **Full rewrite** — do NOT copy source JavaScript. Reimplement all logic as idiomatic React (components, hooks, context).
- Use the source `script.js` and `index.html` as the **behavioral specification**: match rendering output, engine switching behavior, edge cases, scroll preservation, autosave timing.
- Replace DOM manipulation with React state + refs, event listeners with React event handlers, global `Editor` object with React context + component state, `innerHTML` with `dangerouslySetInnerHTML`.

### Implementation Order
1. **Shell & Core Editing:** Next.js scaffold, two-pane layout, markdown-it pipeline, debounced render, `useLocalStorage` autosave.
2. **Engine Switching & Rich Rendering:** Engine abstractions, math engines, highlight.js, Mermaid, toolbar wired to context.
3. **Export, Custom CSS, Paste Detection:** PDF/MD/TXT export, CSS panel, LaTeX paste auto-wrapping.
4. **Responsive & Resilience:** Mobile layout with hamburger/pane toggle, library loading states, keyboard accessibility, print styles.

## Security
- Do NOT add HTML sanitization (no DOMPurify). Users render their own content; `html: true` in markdown-it is intentional.
- Do NOT add a Content Security Policy meta tag — it conflicts with MathJax inline styles, custom CSS injection, and CDN loading.
- Do NOT sanitize custom CSS input.
- MathJax CDN script must include SRI integrity hash and `crossorigin="anonymous"`.
- No sensitive data in localStorage — no encryption needed.

## Performance
- **Debounce delay:** 300ms on editor input → preview re-render.
- **Autosave interval:** 5000ms.
- Scroll position preservation during re-renders via `requestAnimationFrame`.
- Initial JS target: under ~150KB gzipped (excluding dynamically loaded libs).

## Accessibility
- All toolbar buttons, dropdowns, selectors must be keyboard-focusable and operable.
- Focus indicators: `outline: 2px solid #A31D1D` with `outline-offset: 2px`.
- Use semantic HTML: `<button>`, `<label>`, `<textarea>` — do NOT use divs as buttons.
- `aria-label` on icon-only buttons (hamburger menu), `aria-expanded` on dropdowns.
- Match source app colors exactly — do NOT attempt WCAG AA contrast corrections.

## Responsive Breakpoints
- `< 480px`: compact mobile
- `480–768px`: small tablet
- `768px+`: full two-pane desktop layout
- Below 768px: single-pane toggle (Editor/Preview) with hamburger menu.
- Tables, code blocks, math must be horizontally scrollable on small screens.

## Print Styles
In `globals.css` under `@media print`:
- Body: `11pt`, `line-height: 1.4`
- Code blocks: `9pt`, `pre-wrap`, `page-break-inside: avoid`
- Tables: full width, `page-break-inside: avoid`
- Images/SVGs: `max-width: 100%`, `page-break-inside: avoid`
- Headings: `page-break-after: avoid`

## Error Handling
- Rendering errors: catch per-engine, show red error text in preview pane.
- Mermaid errors: per-block error display with expandable details and source code.
- Math errors: orange warning prepended to preview, non-fatal.
- localStorage errors: `try/catch`, fall back to in-memory state.
- PDF generation errors: alert dialog, restore button state.

## Testing

### Frameworks
- **Jest** + **React Testing Library** for unit/component tests
- **Playwright** for E2E browser tests
- `ts-jest` for TypeScript, `jest-environment-jsdom` for browser-like env

### Test File Location
Place `__tests__/` directories inside each feature/lib module. E2E tests in `src/e2e/`.

### Mocking Strategy
- **MathJax:** Global stub in `jest.setup.ts` for Jest/RTL. Real MathJax only in Playwright.
- **Mermaid:** `jest.mock('mermaid')` in component tests. Real in E2E.
- **highlight.js:** Runs in jsdom — test directly, no mock.
- **markdown-it / marked:** Pure JS — no mocking needed.
- **KaTeX:** npm package works in jsdom. Mock `renderMathInElement` in component tests.
- **jsPDF / html2canvas:** Mock in component tests. Validate in E2E.
- **localStorage:** `jest.spyOn(Storage.prototype, ...)` for error cases.

### What NOT to Test
- Do NOT write visual regression / pixel comparison tests.
- Do NOT test third-party library internals.
- Do NOT unit-test Tailwind CSS classes.
- Do NOT test CDN availability.

## Browser Support
- Chrome, Firefox, Safari, Edge (latest 2 versions), Safari iOS, Chrome Android.
- Do NOT support IE11.