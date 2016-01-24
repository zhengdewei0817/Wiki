# 设计模式之路－单例模式 --  郑德伟，笔记，2016-01-22

本系列文章来自曾探的《设计模式与开发事件》
单例模式的定义：保证一个类仅有一个实例，并且提供一个访问他的全局访问点。

###实现简单的单例模式
###透明的单例模式
###用代理实现单例模式
###JS中的单例模式
###惰性单例
###通用的惰性单例模式

####实现简单的单例模式
要想实现单例模式很简单 ，只需要标注是实例化的对象是否存在即可，用JS代码实现如下：    
```
var Person = function(name){
    this.name = name;
    this.instance = null;
}
Person.getInstance = function(name){
    if(!this.instance){
        this.instance = new Person(name);
    }
    return this.instance;
}
var xiaoming = Person.getInstance('xiaoming');
var baby = Person.getInstance('baby');
alert(xiaoming === baby);  //true
```
这就是最近单的单例模式，然是有一个问题，就是增加了不透明性，如果我不知道Person可以实现单例，我便不能实现，这样的单例意义其实不大。

####透明的单例模式
现在我们要实现透明的单例模式效果。
```
var createDiv = (function(){
    var instance;
    var createDiv = function(html){
        if(instance){
            return instance;
        }
        this.html = html;
        this.init();
        return instance = this;
    }
    createDiv.prototype.init = function(){
        var div = document.createElement('div');
        div.innerHTML = this.html;
        document.body.appendChild(div);
    }
    return createDiv;
})()
var aa = new createDiv('aaa1');
var bb = new createDiv('bbb1');
alert(aa === bb);//true
```
这里通过使用闭包的方式，将this保存到instance中，实现单例模式，但是有一个问题，这只能实现单例功能，如果要创建多个不同的div，则要对代码进行修改，并且代码过于复杂

####代理模式实现单例
为了解决上面的问题，我们现在采用代理实现单例模式
```
var createDiv = (function(){
       var createDiv = function(html){
           this.html = html;
           this.init();
       }
       createDiv.prototype.init = function(){
           var div = document.createElement('div');
           div.innerHTML = this.html;
           document.body.appendChild(div);
       }
       return createDiv;
   })()
var proxy = (function(){
    var instance;
    return function(html){
        if(!instance){
            instance = new createDiv(html)
        }
        return instance;
    }
})()
var aa = new proxy('aaa2');
var bb = new proxy('bbb2');
alert(aa === bb);//true
```

虽然跟透明模式相似，但是代理模式把单例的逻辑单独提出，这样创建div的功能作为独立的功能存在，代码结构清楚，复用性强


####JS中的单例模式
这里是JS 所以不需要传统的OOP语言也能实现单例模式，利用的就是JS中的全局变量
var a = {} 
这里的a是唯一的存在，除非你重新对a进行赋值，单也会出现其他问题，例如全局变量的污染，这时候就需要采用空间命名的方式，
```
var zdw = {
    event:{},
    dom:{}
}
```

####惰性单例模式
简单的说，就是需要的时候才创建的单例模式，惰性单例模式是最重要的单例模式，有用的程度超乎我们的想象。
比如，页面有一个弹出窗口，点击后才会显示，可能大多数时间是不会显示的，因此不需要一开始就时候就占用DOM节点数，只有在点击的时候，才会出现，但是又不能每次点击的时候创建多个，因此 就用到了惰性单例模式
```
var createAlertLayer = (function(){
    var div;
    return function(){
        if(!div){
            div = document.createElement('div');
            div.style.display = 'none';
            div.innerHTML = '弹窗'
            document.body.appendChild(div)
        }
        return div
    }
})()
document.getElementById('btn').onclick = function(){
    var layer = new createAlertLayer();
    layer.style.display = 'block'
}
```
这样就实现了惰性单例模式，但是依然有一个问题，就是没有把单例逻辑与业务逻辑进行分离，因此 我们需要一个通用的惰性单例

####通用的惰性单例
不管什么模式，上面的代码都是判断一对象是否存在，如果存在，则返回对象本身，如果不存在，则创建实例并赋值给对象，然后在返回对象本身
因此可以把单例模式抽出来作为独立部分存在
```
var getInstance = function(f){
    var result;
    return function(){
        return result || (result = f.apply(this,arguments));
    }
}
var createAlertLayer = (function(){
    var div;
    return function(){
        if(!div){
            div = document.createElement('div');
            div.style.display = 'none';
            div.innerHTML = '弹窗'
            document.body.appendChild(div)
        }
        return div
    }
})()
var createInstance = getInstance(createAlertLayer);
document.getElementById('btn').onclick = function(){
    var layer = new createInstance();
    layer.style.display = 'block'
}
```
通过对单例逻辑的抽出，就实现了业务逻辑的分离，解决了之前的问题。

