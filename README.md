cocos-utils
===========

Cocos utilities for Cocos2d-html5 NPM supporting.

A tool to help developers coding cocos2d-html5 easily.


## Installing
* Install `nodejs`...
* Install `ant`...
* Then type:

```bash
npm install cocos-utils -g
```

You know, it may take a long time to install modules from npm in China, because there is a "Wall".
In this case, you can try this:

```bash
npm --registry "http://registry.cnpmjs.org" install cocos-utils -g
```

Type `cocos help` to check whether the installing is successful.



## HelloWorld

##### Install all modules of cocos2d-html5

```bash
cd your/workspace/
cocos install
```

##### Create project of cocos2d-html5

(under your workspace)

```bash
cocos new helloworld
cd helloworld
cocos build
```

##### Visit dev version
* Be sure that your project has been published in a webserver, then visit `http://serverhost:port/your/project/path/index.html` in your browser.

##### Publishing

(under helloworld)

```bash
cocos publish
```

##### Visit release version
* Be sure that your project has been published in a webserver, then visit `http://serverhost:port/your/project/path/release.html` in your browser.

## Structure of project

```script
- node_modules (dir of engine modules)
    -cocos2d-html5 (core for engine)

- helloworld (project dir)
    - cfg
        - res.js (Config of path of resources, generated by cocos genRes)
        - jsRes.js (Config of path of js sources, generated by cocos genRes)
        - resCfg.js (Config of dependencies for all resources)

    - res (Path of resources)
        - Normal (Normal version)
        - HD (HD version)
        - Audio

    - src (Dir to put sources)
    - test  (Dir to put test sources)

    - node_modules (dir of third party modules)

    - cocos2d.js (Boot config of game)
    - main.js (Main for game)
    - baseCfg.js (Base config for game to load js and so on)
    - index.html (Url of dev version)

    - mini.js (Generated by cocos publish)
    - build.xml (Generated by cocos publish, used for closer compiler)
    - release.html (Url of release version)

    - cocos.json (Config for cocos command)

    - package.json
```



## Files generated by the utils.
There are files generated by the utils, which you should not edit by yourself.
* files in __temp
* res.js
* jsRes.js
* baseCfg.js
* build.xml (after publishing)
* mini.js (after publish)
* sourcemap (after publish)


## cocos.json
This file is about config of `cocos` command.
If there is error occurs while using `cocos` command, check this file first.

[Click here to se more details](https://github.com/SmallAiTT/cocos-utils/wiki/cocos.json).


## resCfg
This is the main config for the dependencies of the project.
It is the core to load js and resources, just like what `resources.js` and `appFiles` of `cocos2d.js` do in the develop branch.
`cc.js` takes place of `jsLoader.js`, so `jsLoader` do not work in this npm branch.



Simply like this:

```script
resCfg[key] = {
    res : [],
    ref : [],
    sprite : "",
    layer : "",
    scene : "",
    args : {},
    ...//your custom config
}
```

Each property is optional.

All paths of resources are generated in `res.js`, and all paths of js are generated in `jsRes.js`.
Use these to configure `resCfg`.

The `key` should be a string. Simply you can use `moduleName`,
`js.moduleName.aJsSrc_js` configured in the jsRes.js of modules,
or just a simple string.

`ref` is short for `reference`, which contains the references for this part.

`res` is short for `resource`, whiche means the resources to be loaded for the part.

##### Base config part of module
The `resCfg["moduleName"]...` part is the base config of the module. e.g. a module named `m1`:

```script
var resCfg = cc.resCfg || {};
var jsRes = js.m1;
resCfg["m1"] = {
    ref : [jsRes.code01_js, jsRes.code02_js],
    res : [jsRes.a_png, jsRes.b_png]
};
```

It means that the base config of the module references `code01.js` and `code02.js`, and preloads `a.png` and `b.png`.

So `resCfg["m1"]` will be loaded when the project boots by default,
which means config for `code01.js` and `code02.js` will been loaded, and so as `a.png` and `b.png`.

##### Config for a js
Pay attention to this, if none config should be set for a js, you do not need to write the config followed.

```script
resCfg[jsRes.code03_js] = {
    res : [res.c_png],
    sprite : "MySprite",
    args : {a : "AAA"}
};
```

You can see that, `code03.js` has no references, but one resource named `c.png`.

The content of `code03.js` is simply like this:

```script
var MySprite = cc.Sprite.extend({...});
MySprite.create = function(args){...};
```

The `sprite` is used to test `code03.js`.
Be sure the there is a class name named `MySprite` in `code03.js`,
and the `MySprite` has a function called `create`.
`args` of the config will be passed to the `MySprite.create` function.
If the `create` function has no arguments, just keep `args` null.

Then, test mode is opened while the `config.test` is configured in `main.js`, such as:

```script
config.test = js.m1.code03_js;//config which js you want to test
```

Otherwise, the project will be ran as normal mode.

e.g. set `config.test = js.m1.code03_js`, visit index.html, then you will see the test case of `code03.js`.

Same as `layer`, `scene` and so on.

Custom interface of test unit will be provided in the future.


You can also write your test code in the `test` folder, such as a test js file named `code03Test.js`:

```script
var MySpriteTest = cc.Layer.extend({
    //TODO write code to test code03.js
    ...
});
MySpriteTest.create = function(args){...};
```

Run `cocos genJsRes` or `cocos build` to generate the path config of code03Test.js into jsRes.js.

Then configure:

```script
resCfg[jsRes.code03Test_js] = {
    ref : [jsRes.code03_js],
    layer : "MySpriteTest",
    args : {...}
}
```

Set `config.test = js.m1.code03Test_js`, visit `index.html`, then test case of MySpriteTest would be run.

Test codes will not be complied by running `cocos publish`.

By this way, you can test your js file easily,
without editing any code of source of your project,
and see the effort immediately.
You do not need to boot the whole game, and do a lot of "click" actions to reach the page which will be tested.


##### Reference
```script
resCfg[jsRes.code04_js] = {
    ref : [jsRes.code03_js]
};
resCfg[jsRes.code05_js] = {
    ref : [jsRes.code04_js]
};
```

In the above, `code03.js` is referenced by `code04.js` and `code04.js` is referenced by `code05.js`.

Not matter what has been changed in `code04.js`, nothing will be changed, while the interface of `code03.js` is not changed.

This will be good for team work, that everyone just care about the interfaces provided by others.

`code04.js` do not need to care about what resources are used in `code03.js`, or what scripts are referenced in `code03.js`.


##### Game Modules
```script
resCfg.gameModules = [jsRes.code05_js, ...];
```

This is for modules of game, such as home page, fighting and so on.

The engine will load resources and js for the modules by this config.
e.g. `code05.js` is the entry of fighting module, you press fight button to enter it (onClick function bellow):

```script
cc.loadGameModule(js.m1.code05_js, function(resArr){
    cc.LoaderScene.preload(resArr, function(){
        cc.Director.getInstance().replaceScene(...);
    });
});

```

And this config tells `cocos publish` which scripts should be compiled.
The base config part of resCfg will be compiled by default.

#### Last But Not Least
`resCfg.js` looks so complex, but it is easy to be used step by step to improve the efficiency of coding,
for you know which resources and js are required in this current js.
It is the core for uncoupling.

`resCfg.js` tells the dependencies of modules and scripts, so that `cocos publish` can get all the scripts to be compiled.




## package.json
Same as `package.json` of `npm`.

Third party modules will be configured in `dependencies`.
If you add or delete a module (dependencies in package.json or cocos.json),
you should run `cocos build` or `cocos genBaseCfg` once more.


## Test Case
By default, we have provided some test functions for you.

* Test sprite

If you have a Sprite class in your MySprite.js, and it may be this:

```script
var MySprite = cc.Sprite.extend({
    //TODO write code for the sprite
    ...
});
MySprite.create = function(args){
    var sprite = new MySprite();
    ...
    return sprite;
};
```

Then you configure it in `resCfg.js`:

```script
resCfg[jsRes.MySprite_js] = {
    res : [...],//if the sprite requires resources, optional.
    ref : [...],//if the sprite references others, optional.
    sprite : "MySprite",//the class name of MySprite, required when you want to test MySprite
    args : {...}//the args you want to pass into MySprite.create, optional.
}
```

Set `config.test = js.mymodule.MySprite_js`, visit `index.html`, then test case of `MySprite` would be run.

Make sure that there is a function called `create` of `MySprite`.

`args` is good for your project.
Sometimes `MySprite` requires data from others, you just need to use `args` to pass it into `MySprite`,
without changing the code of `MySprite` when testing.

By this way, you can test your code easily.

* Test layer

As same as above, by using `layer` instead of `sprite` in `resCfg`.

* Test scene

As same as above, by using `scene` instead of `sprite` in `resCfg`.

* Details about using default test function have been introduced in the chapter `resCfg`.

##### Custom Test Case
You can define your own test function for your project.
It is simply easy that, you just need to create a function, and register it.
Simply like this:

```script
function myTest(cfgName, cfg){
    //TODO write code for your own test function
}
cc.unitMap4Cust.myTest = myTest;
```

The engine will pass 2 arguments to your test function.
The first one `cfgName` is the argument you pass into `cc.test`,
and the second one is the value configured in resCfg of `cfgName`.

Configure in resCfg:

```script
resCfg[jsRes.MySprite_js] = {
    res : [...],
    ref : [...],
    myTest : "MySprite",
    args : {...}
}
```

Then MySprite will be run by the test function of `myTest`.


##### TEST_BASE
When you need to include scripts and resources for all of your test code, you can configure `TEST_BASE` in resCfg:

```script
resCfg[TEST_BASE] = {
    res : [res.a_png...],//resources for test by default
    ref : [jsRes.TestBase_js]//scripts for test by default
};
```

Then you do not need to configure `res.a_png` to the `res` or `jsRes.TestBase_js` to the `ref` of the scripts you want to test.

Pay attention to this, `TEST_BASE` will not be included while publishing.


## Notes
Do not forget run `cocos genRes` if you add or delete resources, rename the resources or change the paths of the resources.

Do not forget run `cocos genJsRes` if you add or delete js, rename js or change the path of js.

Do not forget run `cocos genBaseCfg` if you install or uninstall modules of cocos2d-html5 which are configured in dependencies of cocos.json,
or of third party which are configured in dependencies of package.json.

In fact, you can use `cocos build` which includes these three command above.


Be sure that each key in your `res.js` and `jsRes.js` is unique, which is generated by the file name. The rule of key is :

```script
a.png  :  a_png        //"." will be replaced by "_"
a-b.png  :  a_b_png    //"-" will be replaced by "_"
1a.png  :  _1a_png     //"_" will be add before if the first char is a number
a b.png  :  ab.png     //" " will be replaced by ""
```

`cc.js` will be replaced with `cc4publish.js` when publishing.