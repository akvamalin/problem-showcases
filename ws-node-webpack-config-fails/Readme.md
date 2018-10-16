# ws-node-webpack-config-fails

TODO: Add this nasty bug description.

Key notes:
1. webpack config webpack-node-externals missing node_modules in path
2. it says package.json should be required using node require function
3. As it does not see that node_modules are external -> it bundles all the files together
4. when node index.bundle.js is executed, it uses requre function to require package.json.
5. But package.json path is now not the correct one or missing.

TODO: Describe it in details
