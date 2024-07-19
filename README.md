# @hyrious/configs

Shared configs across my projects.

## Usage

```bash
npm i -g @hyrious/configs
```

### [tsconfig.json](./docs/tsconfig.md)

- 0.1.3: Add [`"moduleDetection": "force"`](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-7.html#control-over-module-detection).

- 0.1.2: Make use of [`"module": "Preserve"`](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-4.html#support-for-require-calls-in---moduleresolution-bundler-and---module-preserve) (TypeScript &ge; 5.4)\
  If your TypeScript Compiler &lt; 5.4, you can lock `"@hyrious/configs": "0.1.1"`.

## License

MIT @ [hyrious](https://github.com/hyrious)
