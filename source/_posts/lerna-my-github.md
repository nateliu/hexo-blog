---
title: lerna my react related projects into monorepos
date: 2022-01-21 10:15:22
tags:
---
Learned React for long times. And have created many repositories. But, it is not an easy way to findout if long time to used them. So I made all related react repository into my `best-practice-in-react` monorepos.
Here is my steps for creating lerna monorepos.

### 1. Prerequisite
```bash
yarn global add lerna
mkdir best-practice-in-react
cd best-practice-in-react
lerna init
touch README.md
```

The repository looks like below.
```json
best-practice-in-react/
  packages/
  package.json
  lerna.json
  README.md
```

For better performance, we will go with YARN. So how do we configure YARN with Lerna?
It's pretty simple! We just need to add `"npmClient": "yarn"` to `lerna.json`

### 2. Using YARN workspaces with Lerna
add `"useWorkspaces": true` to `lerna.json` and in `package.json`, we need to add 
```json
"workspaces": [
    "packages/*"
  ]
```
Once this is done, we can run command below, it is means forcefully telling Lerna to link the packages. As of now, we do not have anything under it, but we can run it to check if this setup can bootstrap and work properly.
```bash
yarn install
or
lerna bootstrap
```
Because we have added `workspaces` in `package.json`, both commands above are the same.

~~### 3. Installing create-react-app and create-react-library
We will use create-react-app & create-react-library to create a base for our libraries and modules.~~
```bash
yarn create react-library ui-components
yarn create react-library common-utils
yarn create react-app my-app
```
~~I don't know why create-library use react 16.13.1, so I changed~~

### 3. Create packages under in packages folder
```bash
yarn create react-app todo-app-js
lerna run start --scope todo-app-js
```
We can see the React website.

Next we add prefix `@best-practice-in-reace` in `todo-app-js` `package.json`.
```todo-app-js package.json
"name": "@best-practice-in-react/todo-app-js
```
Then run commands in terminate
```bash
lerna clean
lerna run start --scop @best-practice-in-react/todo-app-js
```

For convience, make a script in the `package.json`
```package.json
  "scripts": {
    "clean": "lerna clean",
    "bootstrap": "lerna bootstrap",  
    "start:todo-app-js": "lerna run start --scope @best-practice-in-react/todo-app-js"
  }
```
So, the command we can run in terminal as 
```bash
yarn clean
yarn bootstrap
yarn start:todo-app-js
```

Next we create another `typescript` version todo app under in `packages` folder.
```bash
yarn create react-app todo-app-ts --template=typescript
```

So far, both applications are created by `create-react-app`, next we can setup a react from zero.

### 4. Elegant commit
#### . commitizen && cz-lerna-changelog
```bash
yarn add commitizen -DW
yarn add cz-lerna-changelog -DW
```
`commitizen` used for formatting `git commit message`. It providers a interactive to retrieve commit message.
`cz-lerna-changelog` used for `Lerna` commit message specifically。

After install both pckages, we add `config` in root `package.json`. config `cz-lerna-changelog` to `commitizen`, mean time, we have to add scripts for `commitizen` because we save them in `devDenpendenies`. So we have to add `scripts` for excuting `git-cz`

```bash
 "scripts": {
    ...
    "commit": "git-cz",
    ...
 },
"config": {
    "commitizen": {
      "path": "./node_modules/cz-lerna-changelog"
    }
  },
```

At this time,we can use 
```bash
yarn commit 
```
to commit message with elegent. 

#### . commitlint && husky
We used `commitizen` above to standardize commits, but it's up to the developer to use `yarn commit`. What if you forget, or commit directly with `git commit`? The answer is to check the commit information at commit time, and if it doesn't meet the requirements, then you will not commit it and will be prompted. This is done by `commitlint`, and the timing is specified by `husky`. `husky` inherits all the hooks from Git, and when the hooks are triggered, husky can block illegal commits, pushes, and so on.

```bash
yarn add -DW @commitlint/cli @commitlint/config-conventional
yarn add -DW husky
```
Add husky config in root `package.json`
```json
"husky": {
  "hooks": {
    "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
  }
}
```
And add `commitlint.config.js` in root directory.
```bash
touch commitlint.config.js
```
with context:
```javascript
module.exports = { extends: ['@commitlint/config-conventional'] }
```

`commit-msg` is a hook that checks the commit message when `git commit` is triggered, and will use `commitlit` to check it when triggered. If you want to commit via `git commit` or other third-party tools after installation and configuration, you will not be able to commit if the commit message does not match the specification. This constrains developers from using `yarn commit` for commits.

> We can use 
> ```bash
> echo "build: change something" | yarn commitlint
> ```
> to Vverify that your commit message passes the commitlint check or not.

#### . standardjs && lint-staged
```bash
yarn add -DW standard lint-staged
```
```json
"husky": {
    "hooks": {
      "pre-commit": "lint-staged",
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }
  },
  "lint-staged": {
    "*.js": [
      "standard --fix",
      "git add"
    ]
  },
```
Add the `lint-staged` configuration to root `package.json`, as shown above, to perform `standard --fix` checksum on the js files in the staging area and repair them automatically. Then when to verify it, we use `husky` installed above again, husky's configuration add `pre-commit` hook to perform `lint-staged` verification operation, as shown above.
At this time, when the js file is submitted, it will automatically correct and verify the errors. This ensures uniform code style and improves code quality.

### 5. npm publish locally：Verdaccio
```bash
yarn global add verdaccio
verdaccio
touch .npmrc
```
``` json .npmrc content
 registry="http://localhost:4873/"
```
It's done! Whenever you run `lerna publish`, the package built from the child project will be published in the local npm repository, and when you run `lerna bootstrap`, `Verdaccio` will release it, allowing you to successfully pull the corresponding code from the remote npm repository.

### 5. Push this to github
New a repostory in github.com, and run commands below.
```bash
git remote add origin https://github.com/nateliu/best-practice-in-react.git
git branch -M main
git add .
git commit -m 'Initialize best-practice-in-react'
git push -u origin main
```

### 6. Import an existed repository into this monorepo
```bash
lerna import ../react-best-practice
lerna import ../react-component-library
lerna import ../webpack-react-startkit
lerna import ../webpack-redux-startkit
lerna import ../react-redux-startkit
```



