# @hyrious/configs

Shared configs across my projects.

## Usage

```bash
npm i -g @hyrious/configs
```

### tsconfig.json

```json
{ "extends": "@hyrious/configs/tsconfig.json" }
```

Default choice for normal projects.

```json
{ "extends": "@hyrious/configs/tsconfig.strictest.json" }
```

The `strictest` adds these configs aside from `strict`:

- noFallthroughCasesInSwitch: true
- noImplicitOverride: true
- noPropertyAccessFromIndexSignature: true
- noUncheckedIndexedAccess: true

```json
{ "extends": "@hyrious/configs/tsconfig.casual.json" }
```

Less strict mode, it alters these settings from `strict`:

- noImplicitAny: false\
  It allows you to write `globalThis.debug = 1` without error.

- strictPropertyInitialization: false\
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

- useUnknownInCatchVariables: false\
  Make the `err` object in catch clause as type of `any`.
  Some people prefer `unknown` because it prevents stupid authors throwing `null` or `undefined` which
  causes `err.message` become a `TypeError: Cannot read properties of undefined`.
  But this is an insane idea and common codes just throw `new Error('message')`.

- noImplicitOverride: true\
  Must write an `override` annotation before these methods. I just like this mark.

- useDefineForClassFields: false\
  `class { foo = 1 }` becomes `class { constructor() { this.foo = 1 } }`.
