

# DESIGN.md

## Visual System
- No Tailwind CSS, no CSS-in-JS, no CSS Modules, no component libraries.
- Use inline `style={{}}` props for component-level layout and styling.
- Use `globals.css` for tokens, pseudo-classes, responsive breakpoints, scrollbars, print styles, and preview pane content.
- All design tokens are CSS custom properties on `:root` in `globals.css`.

## Color Palette (`:root` variables)
| Variable | Hex | Usage |
|---|---|---|
| `--color-parchment` | `#F0EADC` | Page bg, code block bg, inline code bg |
| `--color-cream` | `#FEF9E1` | Editor/preview pane bg |
| `--color-lace` | `#FDF5E6` | Toolbar bg |
| `--color-crimson` | `#A31D1D` | Primary buttons, accents, inline code text, scrollbar thumb |
| `--color-maroon` | `#6D2323` | Button hover/active, label text |
| `--color-tan` | `#E5D0AC` | Borders, dividers, heading underlines, table borders |
| `--color-link` | `#0366d6` | Preview hyperlinks |

- Do NOT create custom colors outside these variables. Reference via `var(--color-*)` in both inline styles and CSS.

## Typography
- `--font-sans`: `-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol"` — use for all UI elements.
- `--font-mono`: `'Fira Code', Consolas, Monaco, 'Andale Mono', 'Ubuntu Mono', monospace` — use for editor textarea and code blocks.
- `--font-size-base`: `16px`, `--line-height-base`: `1.6`.

## Spacing & Layout
- `--toolbar-padding`: `8px 15px`, `--toolbar-gap`: `15px`.
- `--pane-padding`: `20px` for editor/preview panes.
- `--radius-button`: `4px`, `--radius-panel`: `5px`.
- `--container-height`: `calc(100vh - 50px)` for main content area.
- Do NOT use arbitrary magic numbers. Use these variables.

## Components

### Buttons
- Apply `.btn` class for hover/focus/active states (defined in `globals.css`).
- Default inline: `background: var(--color-crimson); color: white; font-weight: bold; font-size: 14px; border-radius: var(--radius-button); padding: 6px 12px; border: none; cursor: pointer;`
- `.btn:hover` → `background: var(--color-maroon);`
- `.btn:focus-visible` → `outline: 2px solid var(--color-crimson); outline-offset: 2px;`
- Disabled state: `background: #ccc; cursor: wait;`

### Dropdowns
- Trigger on hover/focus via global CSS pseudo-classes.
- Container: `background: white; box-shadow: 0 4px 12px rgba(0,0,0,0.15); border-radius: var(--radius-button);`
- `.dropdown-item:hover` → `background: var(--color-cream); color: var(--color-crimson);`

### Preview Pane (`#preview-content`)
- Style all rendered Markdown via descendant selectors in `globals.css`.
- Headings: `font-weight: 600; color: #000; border-bottom: 1px solid var(--color-tan); padding-bottom: 4px;`
- Code blocks: `background: var(--color-parchment); border: 1px solid var(--color-tan); border-radius: var(--radius-button); padding: 16px;`
- Inline code: `background: var(--color-parchment); color: var(--color-crimson); font-family: var(--font-mono); font-size: 85%; padding: 2px 6px; border-radius: 3px;`
- Tables: `border-collapse: collapse; border: 1px solid var(--color-tan);` with alternating row bg.
- Blockquotes: `border-left: 4px solid var(--color-tan); color: #666; padding-left: 16px;`
- Mermaid: `text-align: center; background: white; border: 1px solid var(--color-tan); border-radius: var(--radius-button); box-shadow: 0 1px 3px rgba(0,0,0,0.1); padding: 16px;`

### Scrollbars (global CSS)
- Webkit: `8px` width, `var(--color-crimson)` thumb, `var(--color-parchment)` track.
- Firefox: `scrollbar-width: thin; scrollbar-color: var(--color-crimson) var(--color-parchment);`

## Responsive Breakpoints
| Breakpoint | Behavior |
|---|---|
| `≤ 480px` | Compact spacing, smaller text |
| `> 768px` | Two-pane side-by-side, desktop toolbar |
| `≤ 768px` | Single-pane toggle view, hamburger menu, Editor/Preview toggle buttons |
| `> 991px` | Full-size headings, standard toolbar padding |

- Use a `useMediaQuery` hook for JS-driven layout switching; use `@media` in `globals.css` for pure CSS adjustments.

## `globals.css` Section Order
1. CSS Custom Properties (`:root`)  2. Base/reset (`box-sizing: border-box`)  3. Component classes (`.btn`, `.dropdown-item` + pseudo-states)  4. Preview pane (`#preview-content` descendants)  5. Scrollbar styles  6. MathJax/KaTeX overrides  7. Responsive `@media`  8. Print `@media print`

## Do NOT
- Do NOT use Tailwind, styled-components, Emotion, or CSS Modules.
- Do NOT install any UI component library (shadcn, Radix, Headless UI, MUI, etc.).
- Do NOT define colors as raw hex in components — always use `var(--color-*)`.
- Do NOT use pseudo-classes in inline styles — put them in `globals.css`.
- Do NOT deviate from the warm parchment + crimson visual identity.