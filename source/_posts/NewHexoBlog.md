---
title: NewHexoBlog
date: 2017-04-29 21:35:15
tags:
---
## Create Hexo Blog step by step

### Step 1 install git,nodejs

[Git](https://git-scm.com/downloads)
[NodeJS](https://nodejs.org/en/download/)


### Step 2 install hexo
``` bash
$ npm install -g hexo
$ npm install
```

### Step 3 create folder and initilize 
``` bash
$ cd d:\GitHub
$ mkdir hexo-blog
$ hexo init
```

### Step 4 generate html pages
``` bash
$ hexo generate (hexo g)
```

### Step 5 testing in local
``` bash
$ hexo server (hexo s)
```

### Step 6 deploy into github
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
