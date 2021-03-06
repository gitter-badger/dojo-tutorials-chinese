# 1.1HelloDojo

##简介
本篇入门，关于Dojo的加载和一些核心功能，另外还有基于AMD的模块加载，还有出错时如何寻找帮助。 

##入门
开始使用只需要把dojo.js文件引入web页面，方式和其他javascript文件一样。Dojo在主流CDNs上也有。

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Tutorial: Hello Dojo!</title>
</head>
<body>
    <h1 id="greeting">Hello</h1>
    <!-- load Dojo -->
    <script src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js"
            data-dojo-config="async: true"></script>
</body>
</html>
```
通常，你只要加载一个库的js文件就可以使用它的所有方法。Dojo以前也是这样的，但1.7版后源码采取异步模块定义(AMD)，可实现完全模块化的web应用开发。选择AMD是因为它使用纯javascript使源文件可在浏览器中工作，同时支持资源优化的构建过程以提高部署时的应用性能。
那么dojo.js加载后有什么功能是可用的？Dojo的AMD加载器，它定义了两个全局函数require和define。AMD细节见[Introduction to AMD tutorial](https://dojotoolkit.org/documentation/tutorials/1.10/modules/)。入门阶段只需要知道require用来加载并使用模块、define用来自定义模块。一个模块作为一个单独的javascript源文件。
几个HTML DOM操作的Dojo基本模块是dojo/dom and dojo/dom-construct。下面是如何加载和使用这几个模块：

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Tutorial: Hello Dojo!</title>
</head>
<body>
    <h1 id="greeting">Hello</h1>
    <!-- load Dojo -->
    <script src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js"
            data-dojo-config="async: true"></script>

    <script>
        require([
            'dojo/dom',
            'dojo/dom-construct'
        ], function (dom, domConstruct) {
            var greetingNode = dom.byId('greeting');
            domConstruct.place('<em> Dojo!</em>', greetingNode);
        });
    </script>
</body>
</html>
```
require的第一个参数是一系列你需要加载的模块的标识符。总之，这些映射直接对应文件名，如果你查看Dojo目录，就能找到定义这些模块的dom.js和dom-constuct.js。
AMD加载器进行异步操作，并且js异步操作执行回调，require的第二个参数就是回调函数。在回调函数中就是你提供的使用这些模块的代码。AMD加载器将模块作为参数传递给你的回调函数（它们以同样的顺序列在模块id数组中）。你可以随意命名这些参数，但为了代码一致性和可读性推荐使用基于模块id的命名。
在第18和19行可见dom和dom-construct模块用来通过id获取一个DOM节点并操作其内容。
AMD加载器会自动加载请求模块的所有子依赖，所以要只累出你直接使用的模块。

##定义AMD模块
下面有一个加载和使用模块的示例。定义和加载你的个人模块，你需要确认的是通过HTTP服务加载HTML文件。在你的目录下添加一个demo目录，将hellodojo.html包含在内，并在demo目录下创建一个myModule.js文件。

```
demo/
    myModule.js
hellodojo.html
```
在myModule.js中输入：

```
define([
    // The dojo/dom module is required by this module, so it goes
    // in this list of dependencies.
    'dojo/dom'
], function(dom){
    // Once all modules in the dependency list have loaded, this
    // function is called to define the demo/myModule module.
    //
    // The dojo/dom module is passed as the first argument to this
    // function; additional modules in the dependency list would be
    // passed in as subsequent arguments.

    var oldText = {};

    // This returned object becomes the defined value of this module
    return {
        setText: function (id, text) {
            var node = dom.byId(id);
            oldText[id] = node.innerHTML;
            node.innerHTML = text;
        },

        restoreText: function (id) {
            var node = dom.byId(id);
            node.innerHTML = oldText[id];
            delete oldText[id];
        }
    };
});
```
ADM define函数和require函数参数相似，一个模块id数组和一个回调函数。AMD加载器将回调函数的返回值存储为该模块的值，所以其他通过require（或者define）加载该模块的代码会接受到定义模块的返回值。

##CDN用法
从CDNs使用Dojo并加载本地模块需用一些额外配置（更多见AMD进阶和通过CDN使用模块教程）。更新hellodojo.html代码如下：

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Tutorial: Hello Dojo!</title>
</head>
<body>
    <h1 id="greeting">Hello</h1>
    <!-- configure Dojo -->
    <script>
        // Instead of using data-dojo-config, we're creating a dojoConfig
        // object *before* we load dojo.js; they're functionally identical,
        // it's just easier to read this approach with a larger configuration.
        var dojoConfig = {
            async: true,
            // This code registers the correct location of the "demo"
            // package so we can load Dojo from the CDN whilst still
            // being able to load local modules
            packages: [{
                name: "demo",
                location: location.pathname.replace(/\/[^/]*$/, '') + '/demo'
            }]
        };
    </script>
    <!-- load Dojo -->
    <script src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js"></script>

    <script>
        require([
            'demo/myModule'
        ], function (myModule) {
            myModule.setText('greeting', 'Hello Dojo!');

            setTimeout(function () {
                myModule.restoreText('greeting');
            }, 3000);
        });
    </script>
</body>
</html>
```
除了添加Dojo配置意外，还重新定义了主要代码，现在只需要载入demo/myModule，并且利用它完成页面上的文本操作。可见，定义和加载模块其实很简单。我们也修改了dojo.js的URL，使用遵守约定（http或https）的链接，防止一些浏览器因混合内容报出安全警告。
在AMD模块中组织代码可便于你创建在浏览器立即执行的模块化js资源，同时也易于调试。AMD模块中变量使用局部作用域，避免了搅乱全局命名空间，也提供了更快的名称解析。AMD是能够多重实现的标准规范，不会局限于单一实现，AMD模块可以被任何AMD加载器调用。
##等待DOM
实现web应用必须要考虑的就是确保浏览器在执行代码前DOM是可用的。plugin（插件）——一个特殊的AMD模块实现了此功能。插件可以像其他模块一样require，在模块标识符后加个感叹号（!）会激活他们的特殊功能。对于DOM ready事件，Dojo提供dojo/domReady插件。只要将这个插件作为一个依赖包含在任何require或define调用，那么DOM准备好之前就不会回调：

```
require([
    'dojo/dom',
    'dojo/domReady!'
], function (dom) {
    var greeting = dom.byId('greeting');
    greeting.innerHTML += ' from Dojo!';
});
```
上面的例子在greeting元素中添加一些文本，它只能在DOM加载后进行（先前不用这个是由于script元素放在body元素的地步，会将脚本延迟到DOM加载后进行）。再次，记得模块标识符以！结尾，否则dojo/domReady模块会和普通模块一样。
有时候，比如dojo/domReady，加载一个模块只是为了它的副作用，并不需要引用它。AMD加载器并不知道这个，它会将依赖数组里的每一个模块引用给回调函数，所以任何你不需要返回值的模块都要放在依赖数组的末尾，并且别在回调函数的参数列表中引用。
DOM操作函数的更多信息参见 [Dojo DOM Functions](https://dojotoolkit.org/documentation/tutorials/1.10/dom_functions/)。
##添加视觉效果
给页面添加动画，模块dojo/fx。下面使用slideTo方法实现滑行动画。

```
require([
    'dojo/dom',
    'dojo/fx',
    'dojo/domReady!'
], function (dom, fx) {
    // The piece we had before...
    var greeting = dom.byId('greeting');
    greeting.innerHTML += ' from Dojo!';

    // ...but now, with an animation!
    fx.slideTo({
        node: greeting,
        top: 100,
        left: 200
    }).play();
});
```
##使用Dojo资源
CDNs很方便，例子中使用它是因为你可以复制代码运行。但它有一些劣势：

 - 为了保证性能，每个模块都进行了压缩和优化，所以出现问题时调试比较困难。
 - 它要求你的应用必须连接互联网。
 - 要包含你自己的惯用模块需要更折腾。
 - 当你产品化的时候，定制化的Dojo能够针对你特定的应用和浏览器提高性能，但CDN实现不了。
 
按以下通用步骤使用Dojo：

1.下载Dojo
2.把Dojo放到你的项目文件中：
```
demo/
    myModule.js
dojo/
dijit/
dojox/
util/
hellodojo.html
```
3.本地加载dojo.js，比CDN更好。

```
<script src="dojo/dojo.js"></script>
```
4.更新配置文件包

```
var dojoConfig = {
    async: true,
    baseUrl: '.',
    packages: [
        'dojo',
        'dijit',
        'dojox',
        'demo'
    ]
};

```
##获取帮助
 [dojo-interest mailing list](http://mail.dojotoolkit.org/mailman/listinfo/dojo-interest) 
 [#dojo on irc.freenode.net](https://dojotoolkit.org/chat)
