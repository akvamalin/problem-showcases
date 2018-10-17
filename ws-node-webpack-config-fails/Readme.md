# ws-node-webpack-config-fails

## Set-up

- Workspace project
- Build node.js typescript using @mercateo/ws tool

## Current problem

Executing bundled script results in an error:

```
$ node dist/index.js
Error: Cannot find module '../../package.json'
    at Function.Module._resolveFilename (module.js:557:15)
    at Function.Module._load (module.js:484:25)
    at Module.require (module.js:606:17)
    at require (internal/module.js:11:18)
    at Object.../../package.json (/home/yevhenii/source/problem-showcases/ws-node-webpack-config-fails/packages/typescript-node-project/dist/webpack:/package.json":1:1)
    at __webpack_require__ (/home/yevhenii/source/problem-showcases/ws-node-webpack-config-fails/packages/typescript-node-project/dist/webpack:/webpack/bootstrap:19:1)
    at Object.../../node_modules/some-cool-library-requires-package-json/src/utils/version.js (/home/yevhenii/source/problem-showcases/ws-node-webpack-config-fails/packages/typescript-node-project/dist/webpack:/home/yevhenii/source/problem-showcases/ws-node-webpack-config-fails/node_modules/some-cool-library-requires-package-json/src/utils/version.js:1:13)
    at __webpack_require__ (/home/yevhenii/source/problem-showcases/ws-node-webpack-config-fails/packages/typescript-node-project/dist/webpack:/webpack/bootstrap:19:1)
    at Object.../../node_modules/some-cool-library-requires-package-json/src/index.js (/home/yevhenii/source/problem-showcases/ws-node-webpack-config-fails/packages/typescript-node-project/dist/webpack:/home/yevhenii/source/problem-showcases/ws-node-webpack-config-fails/node_modules/some-cool-library-requires-package-json/src/index.js:1:17)
    at __webpack_require__ (/home/yevhenii/source/problem-showcases/ws-node-webpack-config-fails/packages/typescript-node-project/dist/webpack:/webpack/bootstrap:19:1)
error Command failed with exit code 1.
```

## Steps to reproduce

1. Clone project and change directory to typescript-node-project
2. Install dependencies and run bundled script

   ```bash
   $ git clone git@github.com:akvamalin/problem-showcases.git && cd problem-showcases/ws-node-webpack-config-fails/packages/typescript-node-project && yarn && yarn build && yarn start
   ```

3. Expect error to occur

## Error origin

Line 347 in `ws/src/lib/webpack/options.ts`

```typescript
export const externalsNode = [
  // require json files with nodes built-in require logic
  function(_context: any, request: any, callback: any) {
    if (/\.json$/.test(request)) {
      callback(null, 'commonjs ' + request);
    } else {
      callback();
    }
  },
  // in order to ignore all modules in node_modules folder
  WebpackNodeExternals({
    modulesDir: isWorkspace ? join(process.cwd(), '..', '..') : undefined
  })
];
```

- Currently this config expects webpack to exclude bundling external `node_modules` and to require all json files using node's `require`.
- `modulesDir: isWorkspace ? join(process.cwd(), '..', '..')` points to the workspace root directory and not `node_modules` directory and therefore Webpack-Node-Externals does not mark node_modules as external modules and they are bundled into single file.
- In a bundled file node's `require` pathes are wrong (as they are not bundled) and this causes an error to occur.

## Proposed solution

1. Add `node_modules` to `join(process.cwd(), '..', '..')`
2. Add another instance of WebpackNodeExternals to also exclude package `node_modules` (as mentioned in this [GitHub issue](https://github.com/liady/webpack-node-externals/issues/39#issuecomment-356647854)).

```typescript
export const externalsNode = [
  // require json files with nodes built-in require logic
  function(_context: any, request: any, callback: any) {
    if (/\.json$/.test(request)) {
      callback(null, 'commonjs ' + request);
    } else {
      callback();
    }
  },
  // default "node_modules" dir is excluded
  WebpackNodeExternals()
];

if (isWorkspace) {
  // ignore all modules in node_modules workspace root folder
  externalsNode.push(
    WebpackNodeExternals({
      modulesDir: join(process.cwd(), '..', '..', 'node_modules')
    })
  );
}
```
