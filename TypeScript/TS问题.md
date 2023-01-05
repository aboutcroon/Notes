## Fix - Cannot find module 'X' Error in TypeScript

**The "Cannot find module or its corresponding type declarations" error occurs when TypeScript cannot locate a third-party or local module in our project. To solve the error, make sure to install the module and try setting `moduleResolution` to `node` in your `tsconfig.json` file.**



If the module is a third-party module, make sure you have it installed.

```shell
npm install module-name

npm install --save-dev @types/module-name
```



If it is a third-party module you're having problems with, try removing your `node-modules` directory and your `package-lock.json` file, re-run `npm install` and reload your IDE.

```shell
rm -rf node_modules package-lock.json

npm install
```



❗️Make sure to reload your IDE as VSCode often glitches and needs a reboot.（很多时候都是因为没有重启vscode所以发现ts检查还在报错，重启后就不报错了）

If that doesn't help or TypeScript can't locate your local modules, try setting `moduleResolution` to `node` in your `tsconfig.json` file.

```json
// tsconfig.json
{
  "compilerOptions": {
    "moduleResolution": "node",
    // 👇️ ... rest
  }
}
```



参考：https://bobbyhadz.com/blog/typescript-cannot-find-module

