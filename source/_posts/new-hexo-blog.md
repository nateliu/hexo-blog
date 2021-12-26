---
title: New Hexo Blog
date: 2021-12-21 11:31:31
tags:
---
Welcome to [Nate Liu's Blog](https://nateliu.github.io/)! This is step by step I created this blog with [Hexo](https://hexo.io/).
### Step 1 install git,nodejs

[Git](https://git-scm.com/downloads)
[NodeJS](https://nodejs.org/en/download/)


### Step 2 install hexo
``` bash
$ npm install hexo-cli -g
```

### Step 3 initilize hexo-blog
``` bash
$ hexo init hexo-blog
$ cd hexo-blog
$ npm install
$ hexo server
```

### Step 4 generate html pages
``` bash
$ hexo generate 
or
hexo g
```

### Step 5 testing in local
``` bash
$ hexo server 
or hexo s
```

### Step 6 deploy into github.io with One-command-deployment
``` bash
$ npm install hexo-deployer-git --save
```
modify _config.yml, go to the end and add following lines:
``` bash
deploy:
  type: git
  repo: https://github.com/nateliu/nateliu.github.io.git
  branch: master
```

### Step 7 launch with following lines
``` bash
$ hexo clean
$ hexo generate
$ hexo deploy
```

### Step 8 go to chrome to see the result.
My Hexo Blog: [My-Hexo-Blog](https://nateliu.github.io)

### Step 9 my source for hexo-blog
[hexo-blog](https://github.com/nateliu/hexo-blog)

### Step 10 using GitHub actions automatically deploy hexo
1. Generate public and private keys
``` bash
$ cd hexo-blog
$ git checkout main
$ ssh-keygen -t rsa -b 4096 -C "jinliangliu@163.com" -f github-deploy-key -N ""
```
Two files are generated in the directory:
- <span style="color:red">github-deploy-key.pub</span> —Public key file
- <span style="color:red">github-deploy-key</span> —Private key file
> Remember to add public and private keys to <span style="color:red">.gitignore</span> Medium!!!

2. GitHub adds public key
In GitHub, in hexo-blog project, follow the <span style="color:red">Settings->Deploye keys->Add deploy key</span> Find the corresponding page and add the public key. In this page, <span style="color:red">Title</span> You can customize it,<span style="color:red">Key </span>Add github-deploy-key.pub The contents of the document.
> Don’t copy more spaces!!!
Remember to check it <span style="color:red">Allow write access</span>. Otherwise, deployment will not be possible. Use below command to copy the contents into pasteboard
```bash
pbcopy < github-deploy-key.pub
```

3. GitHub adds private key
In GitHub, in nateliu.github.io project, follow the <span style="color:red">Settings->Secrets->Add a new secrets</span> Find the corresponding page and add the private key. In this pageNameYou can customize it,ValueAdd github-deploy-key The contents of the document.
> Don’t copy more spaces!!!
Use below command to copy the contents into pasteboard
```bash
pbcopy < github-deploy-key
```

4. Create compilation script
Create it in the hexo-blog branch (I’m here main branch)<span style="color:red">.github/workflows/ci.yml</span> The contents of the document are as follows:
```yml
name: Build and Deploy
on:
  push:
    branches:
      - main
jobs:
  hexo-blog:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source
        uses: actions/checkout@v2
        with:
          ref: main
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v2
        with:
          node-version: '${{ matrix.node-version }}'
      - name: Setup hexo
        env:
          ACTION_DEPLOY_KEY: ${{ secrets.DEPLOY }}
        run: |
          mkdir -p ~/.ssh/
          echo "$ACTION_DEPLOY_KEY" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts
          git config --global user.email "jinliangliu@163.com"
          git config --global user.name "nateliu"
          npm install hexo-cli -g
          npm install
      - name: Hexo deploy
        run: |
          hexo clean
          hexo generate
          hexo deploy
```
5. Hexo configuration
Modify in project root <span style="color:red">_config.yml</span>, add deployment related content:

``` bash
deploy:
  type: git
  repo: git@github.com:nateliu/nateliu.github.io.git 
  branch: master
```
> It likes Step 6 before mentioned, but update the <span style="color:red"> repo: </span>

6. Verification
Now the hexo-blog has been integrated with GitHub actions, push the code into main branch to automatically compile and deploy. specific
The execution process can be performed in <span style="color:red">ActionsView</span>

### Useful links
[Hexo GitHub pages](https://hexo.io/docs/github-pages)
[GitHub actions automatically deploy hexo](https://developpaper.com/github-actions-automatically-deploy-hexo/)
[Hexo Action](https://github.com/marketplace/actions/hexo-action#%F0%9F%8D%8Cexample-workflow---hexo-deploy)