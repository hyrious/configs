# `tsconfig.json`

All configs are made with the assumption that TypeScript is only used for
type checking and generating declarations. Not used for transpile files
which should be done by bundlers.

Requires TypeScript &ge; 5.4 to use `"module": "Preserve"`.

## Variant 1 - `tsconfig.json`

Minimal strict config for most people. Some gotchas:

- `"types": []` means `node_modules/@types/*` won't be imported to the global
  so for example the `setTimeout()` from `@types/node` won't pollute your
  front-end codes.

  If you need `@types/node` in your project or in the `scripts` folder, you can
  create a tsconfig there with such content:

  ```json
  {
    "extends": "@hyrious/configs/tsconfig.json",
    "compilerOptions": {
      "types": ["node"]
    }
  }
  ```

- `"verbatimModuleSyntax": true` means you have to mark type-only imports with
  the `type` prefix, this is often helpful for bundlers to determine whether
  an imported file is really used.

  ```ts
  // `foo` is NOT imported in the file, bundler can delete it!
  import type { A } from 'foo'
  // `foo` is imported, any side effects in `foo` should be preserved.
  import { type A } from 'foo'
  // literally same as the previous ↑ statement
  import 'foo'
  ```

  But if your bundler reads package.json `"sideEffects"` field (invented by webpack,
  supported by esbuild and rollup), which could mark `foo` as side-effect free.
  Then `import 'foo'` can also be dropped.

  You can turn off this setting if you believe your bundler could handle these cases.

## Variant 2 - `tsconfig.strictest.json`

Same config as variant 1, plus the 4 stricter settings:

```json
"noFallthroughCasesInSwitch": true,
"noImplicitOverride": true,
"noPropertyAccessFromIndexSignature": true,
"noUncheckedIndexedAccess": true
```

## Variant 3 - `tsconfig.node.json`

Since Node.js 22.18 we can run `node script.ts` directly. This variant allows that use case.

See [the section in VS Code 1.107 changelog](https://code.visualstudio.com/updates/v1_107#_updated-build-scripts-run-directly-as-typescript) for more details.

Bonus: you can run Electron main scripts in TypeScript directly too.

## Variant 4 - `tsconfig.casual.json` (My Favorite)

Less strict mode, it alters these settings from `strict`:

- `"noImplicitAny": false`

  It allows you to write `globalThis.debug = 1` without error.

- `"strictPropertyInitialization": false`

  It stops reporting error on uninitialized properties in constructor,
  which allows you to write an `initialize()` pattern of code.

  ```ts
  class Rectangle {
    x: number; y: number; width: number; height: number
    constructor() { this.initialize.apply(this, arguments) }
    initialize(x = 0, y = 0, width = 0, height = 0) { /* ... */ }
  }
  ```

  Why this pattern? Because you can reuse other classes' constructors without hack.

  ```ts
  // This is a valid and common usage of JavaScript.
  Rectangle.prototype.initialize.call(somethingNotRectangle, ...args)
  ```

- `"useUnknownInCatchVariables": false`

  Make the `err` object in catch clause as type of `any`.
  Some people prefer `unknown` because it prevents stupid authors throwing `null` or `undefined` which
  causes `err.message` become a `TypeError: Cannot read properties of undefined`.
  But this is an insane idea and common codes just throw `new Error('message')`.

- `"noImplicitOverride": true`

  Must write an `override` annotation before these methods. I just like this mark.

- `"useDefineForClassFields": false`

  `class { foo = 1 }` becomes `class { constructor() { this.foo = 1 } }`.
  This is the only way you can write clean ES6 code without lowering helpers.

## Bonus: Write a TypeScript Library like [Marijn](https://github.com/marijnh)

Marijn is the author of CodeMirror, ProseMirror, Acorn and [Eloquent JavaScript](https://eloquentjavascript.net/).

He doesn't use any formatter or linter other than TypeScript itself, which gives
him the most free way to write codes. Here's the basic setup if you want to do something like him:

1. Init the project, I have a minimal [template](https://github.com/hyrious/create/blob/main/cli.js) to do so:

   ```bash
   mkdir something-useful && cd something-useful
   npm create @hyrious [--cli] [--vite] [--dual] [--public] [--pnpm-better-defaults]
   # --cli     generate a CLI entrypoint
   # --vite    generate index.html and main.ts
   # --dual    build to dist/index.js and dist/index.mjs
   # --public  generate .github/workflows/npm-publish.yml
   ```

2. Edit `tsconfig.json` to aim your target. This is helpful to avoid using anything new.
   For example, if you want to only target ES6 browser:

   ```json
   {
     "extends": "@hyrious/configs/tsconfig.casual.json",
     "compilerOptions": {
       "lib": ["ES6", "DOM"],
       "target": "ES6"
     }
   }
   ```

3. Install `rollup` and prepare a config:

   ```bash
   npm add -D rollup rollup-plugin-esbuild @rollup/plugin-node-resolve
   ```

   ```js
   // rollup.config.mjs
   import { nodeResolve } from '@rollup/plugin-node-resolve'
   import { defineConfig } from 'rollup'
   import esbuild from 'rollup-plugin-esbuild'
   import pkg from './package.json' with { type: 'json' }

   export default defineConfig({
     input: 'src/index.ts',
     plugins: [esbuild({ define: { '__VERSION__': `"${pkg.version}"` } }), nodeResolve()],
     external: mapExternal(Object.keys({
       ...pkg.peerDependencies,
       ...pkg.dependencies,
     })),
     output: [
       { file: 'dist/index.js', format: 'cjs' },
       { file: 'dist/index.mjs', format: 'esm' },
     ],
   })

   function escapeRegExp(string) {
       return string.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
   }

   function mapExternal(names) {
     return names.concat(names.map(name => new RegExp('^' + escapeRegExp(name) + '/')))
   }
   ```

4. In case you want to generate bundled types, I have a [package](https://github.com/hyrious/dts) to do that:

   ```bash
   npm add -D @hyrious/dts
   npx dts src/index.ts -o dist/index.d.ts
   ```
