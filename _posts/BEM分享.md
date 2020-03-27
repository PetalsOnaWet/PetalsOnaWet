---
layout:     post
title:     BEM的规范和使用
subtitle:   
date:       2020-03-27
author:     PetalsOnaWet
header-img: 
catalog: true
tags:
    - 前端
    - css
    - 前端规范
---
# `BEM`的规范和使用

## 什么是`BEM`?

`BEM` 分别代表着：`Block`（块）、``Element``（元素）、`Modifier`（修饰符），是一种组件化的 ``CSS`` 命名方法和规范，由俄罗斯 Yandex 团队所提出。
	
        .block {}

    	.block__`element` {}

    	.block__`element`--modifier {}

    



## `Bem`能做什么，能对我们有什么帮助?

> _在计算机科学领域，只有2个非常难解决的问题：一个是缓存失效，而另一个则是命名。-Phil Karlton_ 

` `Bem`创建的目的是将用户界面划分成独立的（模）块，使开发更为简单和快速，利于团队协作开发。`

+ 无冲突&复用性高

    具体来说我们会遇到在团队多人开发中出现``css``命名冲突，``bem``是如何做到这一点的呢？由于项目开发中，每个组件都是唯一无二的，其名字也是独一无二的，组件内部元素的名字都加上组件名，并用元素的名字作为选择器，自然组件内的样式就不会与组件外的样式冲突了。
    这是通过组件名的唯一性来保证选择器的唯一性，从而保证样式不会污染到组件外。很大的增强了代码的可复用性
+ 命名简单

    很多同事在写``css``时候，不知道起什么名字，想破了脑袋，``Bem``也可以解决这个问题。

+ 风格统一

    `Bem`有利于团队风格的统一，举个例子，``css``命名时到底时使用连字符还是中划线哪里用下划线?每个人都会有不同的看法以及潜意识的写法，很难去说服他人。
    `Bem`类似于最终的拍板，大家统一用一个，而非同一个团队多种风格。
    这对上下游合作，内部合作都会极大的降低沟通成本

+ 结构化

    `BEM`方法可以使得你的`CSS`代码结构性很好，命名很具有逻辑性，从而更加容易理解。
## `Bem`如何使用，它的规范和用法是什么?

+	`Block`: 表示块，可以抽象成一个组件
    +	独立的实体，它本身是有意义的。
    +	例子 `header，container，menu，checkbox，input`
    +	名字包含拉丁字母，数字和句号。
    +	块的名称应该是全局唯一的（可通过增加前缀方式确保全局唯一）
    +	`CSS`
        *	`CSS`中只能使用类名（不能是ID）
+ `Element`: 表示元素，组件下的一个元素，多个元素形成一个组件
        *	`Block`的一部分，当把它独立取出来时，没有任何实际意义。任何元素在语义上都是它自己的block紧密相连的。(出了Block就不能用）
    + 名字可能包含拉丁字母，数字，句号以及下划线。`CSS` class名由`block`名成加两个下划线再加`element`的名字，最后组织成一个块名。
    + `HTML` 任何的在`Block`中的`DOM`节点，都是一个``element``。在一个已知的block中，所有的`element`在语义上都是相等的。这意味着元素名称不能定义成几层，比如`block__elem1__elem2`。

        ```
        Good 

        .block__elem{color:#042};

        Bad

        .block .block__elem {color:#042;}

        div.block__elem {color:#042;}
        
        不依赖当前页面的其它block或者`element`
        
        ```
    + PS

        原则上不会出现2层以上选择器嵌套
        如果组件中包含多个层次的后代元素，不要试图在类名中表现出每一层。`BEM` 不是为了传达结构深度。`BEM` 表达的是组件中的一个后代元素，类名是由块和元素组成。下面的例子里，.photo__caption__quote 不是正确的 `BEM` 写法，.photo__quote 才是。


        ```<!-- 这样写 -->
        <figure class="photo">
        <img class="photo__img" src="me.jpg">
        <figcaption class="photo__caption">
            <blockquote class="photo__quote">
            Look at me!
            </blockquote>
        </figcaption>
        </figure>

        <style>
        .photo { }
        .photo__img { }
        .photo__caption { }
        .photo__quote { }
        </style>


        <!-- 不要这样写 -->
        <figure class="photo">
        <img class="photo__img" src="me.jpg">
        <figcaption class="photo__caption">
            <blockquote class="photo__caption__quote"> <!-- 类名中出现的后代元素不能多于一个 -->
            Look at me!
            </blockquote>
        </figcaption>
        </figure>

        <style>
        .photo { }
        .photo__img { }
        .photo__caption { }
        .photo__caption__quote { }
        </style>

        ```
+	M:表示修饰符，可以用来表示元素或块的状态，比如激活状态，颜色，大小
    +	块或元素上的标志。用它们来改变外观或行为。
    +	名字可以包含拉丁字母，数字，句号以及下划线。`CSS`的class可以由block或者`element`名称后面加--组成，例如.block--mod或者.block__elem--mod，以及.block--color-black .block--color-red。复杂的modifier里由短线替代空格。
    +	-HTML Modifier是一个额外的加在block或者`element` class名之后一个class 名。只为他们负责修改的blocks或者`element`s添加class，然后保持原有的class不变
        ``` 
            Good
            
            <div class="block block--mod">...</div>
            <div class="block block--size-big block--shadow-yes">...</div>

            Bad

            <div class="block--mod">...</div>

            修饰符是配合基础类使用的，不能单独使用，修饰符中也不会包含基础样式。

        ```
    
    + 如果你发现自己总是以相同的方式修改某个组件里的多个元素，那么可以考虑将修饰符添加到组件的基类名上，并根据这个修饰符的含义，调整每个后代元素的样式。这种方式会增加选择器权重，但让修改组件样式变得更简单了

	
        ```
        <!-- 这样写 -->
        <figure class="photo photo--highlighted">
        <img class="photo__img" src="me.jpg">
        <figcaption class="photo__caption">Look at me!</figcaption>
        </figure>

        <style>
        .photo--highlighted .photo__img { }
        .photo--highlighted .photo__caption { }
        </style>

        <!-- 不要这样写 -->
        <figure class="photo">
        <img class="photo__img photo__img--highlighted" src="me.jpg">
        <figcaption class="photo__caption photo__caption--highlighted">Look at me!</figcaption>
        </figure>
        <style>
        .photo__img--highlighted { }
        .photo__caption--highlighted { }
        </style>
        ```

+ __和--连接符

    __主要用来表示块(B)和元素(E)间的连接

    --用来表示块或者元素与状态的连接

    ```
    基本例子

    Html

    <ul class="menu">
    <li class="menu__item menu__item--selected">Menu Item 1</li>
    <li class="menu__item">Menu Item 2</li>
    <li class="menu__item">Menu Item 3</li>
    </ul>

    `Css`

    .menu {
    list-style: none;
    }
    .menu__item {
    font-weight: bold;
    }
    .menu__item--selected {
    color: red;
    }
    ```
## `Element`ui是如何使用`bem`的?

`Elemntui` 通过`mixins.s`css`` 来实现B E M的生成

我们大概看下其中B的生成

```
$namespace: 'el';
$`element`-separator: '__';
$modifier-separator: '--';
$state-prefix: 'is-';


@mixin b($block) {
  $B: $namespace+'-'+$block !global;

  .#{$B} {
    @content;
  }
}
```
这时就能很清楚的看到`block`的生成就是基于``BEM``规范中，块是设计或布局的一部分，具有唯一地意义，利用命名空间`el`加上中划线，以及传入的`block`的名字，构建出`block`的样式，我们通过alert作为假设，在b中通过`!global`提升了一个`$B:el-alert`。通过插值语句`#{ }`生成了 `“.el-alert”`。然后通过`@content`向生成的B中导入内容。 这里大概完成了生成`B`的工作。

`Sass`源码在这里不做过多分享，我们今天主要关注，编译后的``element`ui `对于``Bem``的使用

我们以表格为例

 
比如
`table`
![table](/img/lizi_biaoge.png)

`Button`

![button](/img/lizi_button.png)
 

`Alert`

![alert](/img/lizi_alert.png)

 
## 参考

https://juejin.im/post/5d9c0733f265da5b7525a72d


https://segmentfault.com/a/1190000014687099