# Inline CSS Modules
If you use CSS Modules, why are we putting our CSS in
another file? VS Code, and I imagine most other IDEs, will syntax
highlight CSS in a JavaScript template literal.

This is the way:

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

export const Header = () => {
  return (
    <div className={styles.page}>
      <h1 className={styles.title}>Inline CSS Modules</h1>
    </div>
  )
}
```
esbuild:
```ts
import * as esbuild from 'esbuild'
import { inlineCssModules } from 'inline-css-modules/esbuild'

await esbuild.build({
  entryPoints: ['src/main.tsx'],
  bundle: true,        // required, same as CSS Modules — emits the .css output
  outdir: 'dist',
  plugins: [inlineCssModules()],
})
```
