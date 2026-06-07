# Inline CSS Modules

If you use CSS Modules, why are we putting our CSS in another file? VS Code — and most other editors — will syntax-highlight CSS in a JavaScript template literal just fine.

This is the way (_btw don't):

```ts
import { css } from 'inline-css-modules'

const styles = css`
  .page {
    display: flex;
    gap: 1rem;
  }
  .title {
    font-weight: 700;
  }
`

export const Header = () => (
  <div className={styles.page}>
    <h1 className={styles.title}>Inline CSS Modules</h1>
  </div>
)
```

Your bundler rewrites every `` css`...` `` block at build time: class selectors are scoped (`.page` → `.page_a1b2`), the literal becomes a plain object (`{ page: "page_a1b2" }`), and the scoped CSS is extracted into a real stylesheet that flows through your usual CSS pipeline. **Nothing ships to runtime** — the `css` tag exists only for types and editor highlighting.

## Why

- **Colocation.** Styles live next to the markup that uses them, in the same file.
- **Zero runtime.** No CSS-in-JS engine, no injected `<style>` tags. The output is identical in spirit to CSS Modules — a static class map plus an extracted `.css` file.
- **Type-safe.** Get autocomplete and checked class names with a generic.
- **Real CSS Modules.** Local scoping by default, with `:global` / `:local` and `composes` escape hatches.
- **No dependencies.** Native string handling, nothing to audit.

## Install

```sh
npm install inline-css-modules
```

## Setup

Add the plugin for your bundler.

### Vite

```ts
import { defineConfig } from 'vite'
import { inlineCssModules } from 'inline-css-modules/vite'

export default defineConfig({
  plugins: [inlineCssModules()],
})
```

Scoped CSS routes through Vite's own pipeline, so PostCSS, HMR in dev, and extraction/minification in build all work as they do for an imported stylesheet.

### esbuild

```ts
import * as esbuild from 'esbuild'
import { inlineCssModules } from 'inline-css-modules/esbuild'

await esbuild.build({
  entryPoints: ['src/main.tsx'],
  bundle: true, // required — emits the .css output, same as CSS Modules
  outdir: 'dist',
  plugins: [inlineCssModules()],
})
```

## Usage

### Typed class names

Pass the local names as a generic for autocomplete and type checking:

```ts
const styles = css<{ page: string; title: string }>`
  .page { display: flex; }
  .title { font-weight: 700; }
`

styles.page    // ✓ string
styles.unknown // ✗ type error
```

### `:global` and `:local`

Opt class names out of (or back into) scoping — function, switch, and block forms are all supported:

```ts
css`
  :global(.no-scope) { color: red; }

  .card :global .legacy-child { color: blue; }

  :global {
    body { margin: 0; }
  }
`
```

### `composes`

Compose local classes from the same block; the result is folded into the exported map:

```ts
css`
  .base   { padding: 1rem; }
  .button { composes: base; background: black; }
`
// styles.button === "base_a1b2 button_c3d4"
```

> `composes: ... from "file"` is not supported — an inline block has no external file to compose from.

### Custom scoped names

```ts
inlineCssModules({
  generateScopedName: ({ local, filename, hash }) =>
    `${local}__${hash}`,
})
```

## Notes

- **Static CSS only.** A `${}` interpolation inside a `` css`...` `` block is a build error — move dynamic values to CSS custom properties set at runtime.
- **Editor highlighting.** The styled-components / lit-html VS Code extensions highlight CSS in a `css` tag automatically; no extra config needed.
- **Testing.** Run your test files through the same transform (or the `inline-css-modules/transform` core) so `` css`...` `` blocks are replaced before they reach the runtime.

## License

MIT
