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

```json
{ "extends": "@hyrious/configs/tsconfig.strictest.json" }
```

The `strictest` adds these configs aside from `strict`:

- noFallthroughCasesInSwitch: true
- noImplicitOverride: true
- noPropertyAccessFromIndexSignature: true
- noUncheckedIndexedAccess: true
