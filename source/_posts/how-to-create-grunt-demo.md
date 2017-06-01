---
title: how-to-create-grunt-demo
date: 2017-06-01 21:27:20
tags:
---
## How to create grunt demo step by step

``` bash
$ grunt -version
```
grunt-cli v1.2.0

### Step 1 install grunt, grunt-cli, grunt-init
``` bash
$ npm install -g grunt
$ npm install -g grunt-cli
$ npm install -g grunt-init
```

### Step 2 install template for grunt-init, example as jquery
``` bash
$ cd  USERNAME\.grunt-init
$ git clone https://github.com/gruntjs/grunt-init-jquery.git jquery
```
or if not windows, please just use
``` bash
git clone https://github.com/gruntjs/grunt-init-jquery.git ~/.grunt-init/jquery
```

### Step 3 usage
``` bash
$ grunt-init jquery
```

### Step 4 run test
``` bash
$ grunt qunit
```
Running "qunit:files" (qunit) task
Testing test/test.html ....OK
>> 5 assertions passed (58ms)

## Reference to offical website 
[Getting Started](https://gruntjs.com/getting-started)
