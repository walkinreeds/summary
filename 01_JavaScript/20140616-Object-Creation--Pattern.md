JavaScript 对象创建模式
=========

本篇主要是介绍创建对象方面的模式，利用各种技巧可以极大地避免了错误或者可以编写出非常精简的代码。

###模式1：命名空间（namespace）

**命名空间**可以**减少全局命名所需的数量**，**避免命名冲突或过度**。一般我们在进行对象层级定义的时候，经常是这样的：
```javascript
var app = app || {};
app.moduleA = app.moduleA || {};
app.moduleA.subModule = app.moduleA.subModule || {};
app.moduleA.subModule.MethodA = function () {
    console.log("print A");
};
app.moduleA.subModule.MethodB = function () {
    console.log("print B");
};
```
如果层级很多的话，那就要一直这样继续下去，很是混乱。namespace模式就是为了解决这个问题而存在的，我们看代码：
```javascript
// 不安全，可能会覆盖已有的MYAPP对象
var MYAPP = {};
// 还好
if (typeof MYAPP === "undefined") {
    var MYAPP = {};
}
// 更简洁的方式
var MYAPP = MYAPP || {};

//定义通用方法
MYAPP.namespace = function (ns_string) {
    var parts = ns_string.split('.'),
        parent = MYAPP,
        i;

    // 默认如果第一个节点是MYAPP的话，就忽略掉，比如MYAPP.ModuleA
    if (parts[0] === "MYAPP") {
        parts = parts.slice(1);
    }

    for (i = 0; i < parts.length; i += 1) {
        // 如果属性不存在，就创建
        if (typeof parent[parts[i]] === "undefined") {
            parent[parts[i]] = {};
        }
        parent = parent[parts[i]];
    }
    return parent;
};
```
调用代码，非常简单：
```javascript
// 通过namespace以后，可以将返回值赋给一个局部变量
var module2 = MYAPP.namespace('MYAPP.modules.module2');
console.log(module2 === MYAPP.modules.module2); // true

// 跳过MYAPP
MYAPP.namespace('modules.module51');

// 非常长的名字
MYAPP.namespace('once.upon.a.time.there.was.this.long.nested.property');
```

###模式2：定义依赖

有时候你的一个模块或者函数可能要引用第三方的一些模块或者工具，这时候最好将这些依赖模块在刚开始的时候就定义好，以便以后可以很方便地替换掉。
```javascript
var myFunction = function () {
    // 依赖模块
    var event = YAHOO.util.Event,
        dom = YAHOO.util.dom;

    // 其它函数后面的代码里使用局部变量event和dom
};
```

###模式3：私有属性和私有方法

JavaScript本书不提供特定的语法来支持私有属性和私有方法，但是我们可以通过闭包来实现，代码如下：

```javascript
function Gadget() {
    // 私有对象
    var name = 'iPod';
    // 公有函数
    this.getName = function () {
        return name;
    };
}
var toy = new Gadget();

// name未定义，是私有的
console.log(toy.name); // undefined

// 公有方法访问name
console.log(toy.getName()); // "iPod"

var myobj; // 通过自执行函数给myobj赋值
(function () {
    // 自由对象
    var name = "my, oh my";

    // 实现了公有部分，所以没有var
    myobj = {
        // 授权方法
        getName: function () {
            return name;
        }
    };
} ());
```

###模式4：Revelation模式

也是关于隐藏私有方法的模式，和《JavaScript Module模式》里的Module模式有点类似，但是不是return的方式，而是**在外部先声明一个变量**，然后**在内部给变量赋值公有方法**。代码如下：
```javascript
var myarray;

(function () {
    var astr = "[object Array]",
        toString = Object.prototype.toString;

    function isArray(a) {
        return toString.call(a) === astr;
    }

    function indexOf(haystack, needle) {
        var i = 0,
            max = haystack.length;
        for (; i < max; i += 1) {
            if (haystack[i] === needle) {
                return i;
            }
        }
        return -1;
    }

    //通过赋值的方式，将上面所有的细节都隐藏了
    myarray = {
        isArray: isArray,
        indexOf: indexOf,
        inArray: indexOf
    };
} ());

//测试代码
console.log(myarray.isArray([1, 2])); // true
console.log(myarray.isArray({ 0: 1 })); // false
console.log(myarray.indexOf(["a", "b", "z"], "z")); // 2
console.log(myarray.inArray(["a", "b", "z"], "z")); // 2

myarray.indexOf = null;
console.log(myarray.inArray(["a", "b", "z"], "z")); // 2
```

###模式5：链模式

**链模式**可以你连续可以调用一个对象的方法，比如obj.add(1).remove(2).delete(4).add(2)这样的形式，其实现思路非常简单，就是将this原样返回。代码如下：
```javascript
var obj = {
    value: 1,
    increment: function () {
        this.value += 1;
        return this;
    },
    add: function (v) {
        this.value += v;
        return this;
    },
    shout: function () {
        console.log(this.value);
    }
};

// 链方法调用
obj.increment().add(3).shout(); // 5

// 也可以单独一个一个调用
obj.increment();
obj.add(3);
obj.shout();
```

###模式6：函数语法糖

函数语法糖是为一个对象快速添加方法（函数）的扩展，这个主要是利用prototype的特性，代码比较简单，我们先来看一下实现代码：
```javascript
if (typeof Function.prototype.method !== "function") {
    Function.prototype.method = function (name, implementation) {
        this.prototype[name] = implementation;
        return this;
    };
}
```
扩展对象的时候，可以这么用：
```javascript
var Person = function (name) {
    this.name = name;
}
.method('getName',
            function () {
                return this.name;
            })
.method('setName', function (name) {
    this.name = name;
    return this;
});
```
这样就给Person函数添加了getName和setName这2个方法，接下来我们来验证一下结果：
```javascript
var a = new Person('Adam');
console.log(a.getName()); // 'Adam'
console.log(a.setName('Eve').getName()); // 'Eve'
```

###模式7：对象常量

对象常量是在一个对象提供set,get,ifDefined各种方法的体现，而且对于set的方法只会保留最先设置的对象，后期再设置都是无效的，已达到别人无法重载的目的。实现代码如下：
```javascript
var constant = (function () {
    var constants = {},
        ownProp = Object.prototype.hasOwnProperty,
    // 只允许设置这三种类型的值
        allowed = {
            string: 1,
            number: 1,
            boolean: 1
        },
        prefix = (Math.random() + "_").slice(2);

    return {
        // 设置名称为name的属性
        set: function (name, value) {
            if (this.isDefined(name)) {
                return false;
            }
            if (!ownProp.call(allowed, typeof value)) {
                return false;
            }
            constants[prefix + name] = value;
            return true;
        },
        // 判断是否存在名称为name的属性
        isDefined: function (name) {
            return ownProp.call(constants, prefix + name);
        },
        // 获取名称为name的属性
        get: function (name) {
            if (this.isDefined(name)) {
                return constants[prefix + name];
            }
            return null;
        }
    };
} ());
```
验证代码如下：
```javascript
// 检查是否存在
console.log(constant.isDefined("maxwidth")); // false

// 定义
console.log(constant.set("maxwidth", 480)); // true

// 重新检测
console.log(constant.isDefined("maxwidth")); // true

// 尝试重新定义
console.log(constant.set("maxwidth", 320)); // false

// 判断原先的定义是否还存在
console.log(constant.get("maxwidth")); // 480
```

###模式8：沙盒模式

**沙盒**（Sandbox）**模式**即时**为一个或多个模块提供单独的上下文环境**，而不会影响其他模块的上下文环境，比如有个Sandbox里有3个方法event,dom,ajax，在调用其中2个组成一个环境的话，和调用三个组成的环境完全没有干扰。

Sandbox实现代码如下：
```javascript
function Sandbox() {
    // 将参数转为数组
    var args = Array.prototype.slice.call(arguments),
    // 最后一个参数为callback
        callback = args.pop(),
        // 除最后一个参数外，其它均为要选择的模块
        modules = (args[0] && typeof args[0] === "string") ? args : args[0],
        i;

    // 强制使用new操作符
    if (!(this instanceof Sandbox)) {
        return new Sandbox(modules, callback);
    }

    // 添加属性
    this.a = 1;
    this.b = 2;

    // 向this对象上需想添加模块
    // 如果没有模块或传入的参数为 "*" ，则以为着传入所有模块
    if (!modules || modules == '*') {
        modules = [];
        for (i in Sandbox.modules) {
            if (Sandbox.modules.hasOwnProperty(i)) {
                modules.push(i);
            }
        }
    }

    // 初始化需要的模块
    for (i = 0; i < modules.length; i += 1) {
        Sandbox.modules[modules[i]](this);
    }

    // 调用 callback
    callback(this);
}

// 默认添加原型对象
Sandbox.prototype = {
    name: "My Application",
    version: "1.0",
    getName: function () {
        return this.name;
    }
};
```
然后我们再定义默认的初始模块：
```javascript
Sandbox.modules = {};

Sandbox.modules.dom = function (box) {
    box.getElement = function () {
    };
    box.getStyle = function () {
    };
    box.foo = "bar";
};

Sandbox.modules.event = function (box) {
    // access to the Sandbox prototype if needed:
    // box.constructor.prototype.m = "mmm";
    box.attachEvent = function () {
    };
    box.detachEvent = function () {
    };
};

Sandbox.modules.ajax = function (box) {
    box.makeRequest = function () {
    };
    box.getResponse = function () {
    };
};
```
调用方式如下：
```javascript
// 调用方式
Sandbox(['ajax', 'event'], function (box) {
    console.log(typeof (box.foo));
    // 没有选择dom，所以box.foo不存在
});

Sandbox('ajax', 'dom', function (box) {
    console.log(typeof (box.attachEvent));
    // 没有选择event,所以event里定义的attachEvent也不存在
});

Sandbox('*', function (box) {
    console.log(box); // 上面定义的所有方法都可访问
});
```
通过三个不同的调用方式，我们可以看到，三种方式的上下文环境都是不同的，第一种里没有foo; 而第二种则没有attachEvent，因为只加载了ajax和dom，而没有加载event; 第三种则加载了全部。

###模式9：静态成员

**静态成员**（Static Members）只是一个函数或对象提供的静态属性，可分为私有的和公有的，就像C#或Java里的public static和private static一样。

我们先来看一下公有成员，公有成员非常简单，我们平时声明的方法，函数都是公有的，比如：
```javascript
// 构造函数
var Gadget = function () {
};

// 公有静态方法
Gadget.isShiny = function () {
    return "you bet";
};

// 原型上添加的正常方法
Gadget.prototype.setPrice = function (price) {
    this.price = price;
};

// 调用静态方法
console.log(Gadget.isShiny()); // "you bet"

// 创建实例，然后调用方法
var iphone = new Gadget();
iphone.setPrice(500);

console.log(typeof Gadget.setPrice); // "undefined"
console.log(typeof iphone.isShiny); // "undefined"
Gadget.prototype.isShiny = Gadget.isShiny;
console.log(iphone.isShiny()); // "you bet"
```
而**私有静态成员**，我们可以利用其**闭包特性**去实现，以下是两种实现方式。

- 第一种实现方式：

```javascript
var Gadget = (function () {
    // 静态变量/属性
    var counter = 0;

    // 闭包返回构造函数的新实现
    return function () {
        console.log(counter += 1);
    };
} ()); // 立即执行

var g1 = new Gadget(); // logs 1
var g2 = new Gadget(); // logs 2
var g3 = new Gadget(); // logs 3
```
可以看出，虽然每次都是new的对象，但数字依然是递增的，达到了静态成员的目的。

- 第二种方式：

```javascript
var Gadget = (function () {
    // 静态变量/属性
    var counter = 0,
        NewGadget;

    //新构造函数实现
    NewGadget = function () {
        counter += 1;
   };

    // 授权可以访问的方法
    NewGadget.prototype.getLastId = function () {
        return counter;
    };

    // 覆盖构造函数
    return NewGadget;
} ()); // 立即执行

var iphone = new Gadget();
iphone.getLastId(); // 1
var ipod = new Gadget();
ipod.getLastId(); // 2
var ipad = new Gadget();
ipad.getLastId(); // 3
```
数字也是递增了，这是利用其内部授权方法的闭包特性实现的。

###总结

这是对象创建模式的下篇，两篇一起总共**9种模式**，是我们在日常JavaScript编程中经常使用的对象创建模式，不同的场景起到了不同的作用，希望大家根据各自的需求选择适用的模式。
