### Monorepo built with yarn workspaces + typescript project references

Step by step guide -

- Create the following dir structure

```
    workspace
        -   packages
                - app
                    - src
                        index.ts
                - lib
                    - src
                        index.ts
```

- Run `tsc --init` and `yarn init` inside the packages
- Turn on `composite: true`, `declarations: true`, `declarationMapping: true` inside each `tsconfig.json` files. This ensures these packages can be referenced with `typescript references`
- Set `rootDir: "src"` `include: ["src"]`, `outDir: "dist"` in both `tsconfig.json` files
- add a typescript reference to `app/tsconfig.json` to reference the lib.

```
  "references": [{ "path": "../lib" }],

```

- If typescript detects a package's sources `includes` a file outside it's `rootDir` (e.g. an import tries to refer to it) it will start complianing with references enabled. Unless, a project reference has been added to the dependant project.
- Now compile the project with `tsc -b packages/app`, clean the outputs with `tsc -b --clean packages/app`
- Typescript caches the output of referenced projects in `tsconfig.tsbuildinfo` files.
- With typescript project references, the import path is still `import sayHi from ../../lib/src` this is where `yarn workspaces` come in handy. yarn workspaces help seperate out your `typescript projects` to standalone `npm packages` and connect them via `symlinks`.
- create a yarn workspace by running `yarn init` at the root (i.e. under the workspaces folder)
- Add the following config in `workspaces\package.json`

```
  "workspaces": [
    "packages/*"
  ],
```

- Now `yarn` should pick up these packages
- These could now be linked with `yarn workspace app add lib`
- Somehow this didn't work for me, instead i had to add the reference directly in `app`'s `package.json`

```
 "dependencies": {
    "haritha-lib": "*"
  }
```

- Now run `yarn` at the root (to force run `yarn --force`)
- Verify the workspace dependencies have been create with `yarn workspace info`

````
{
  "app": {
    "location": "packages/app",
    "workspaceDependencies": [
      "haritha-lib"
    ],
    "mismatchedWorkspaceDependencies": []
  },
  "haritha-lib": {
    "location": "packages/haritha-lib",
    "workspaceDependencies": [],
    "mismatchedWorkspaceDependencies": []
  }
}```
````

- `yarn` with hoist the dependencies to the root `node_modules` folder. only package specific deps and executable commands will live in package specific `node_modules` folders.
- Now the import can be simplified to `import sayHi from 'lib/src'`
- Set the entrypoint in `lib/package.json` to simplify this further

```
  "main": "dist/index.js",
```

- Now the import is simply `import sayHi from 'lib'` 🎉
- If `tsc` has trouble findings the types of the `lib` package. Set `typings` \ `types` to `dist\index.d.ts`
- Compile again with `tsc -b packages\app
- Optional: pin `node` and `yarn` versions with `volta`
- Install `volta` globally
- Run `volta pin node@18` and `volta pin yarn@1`
- Voila! 🎉🎉
