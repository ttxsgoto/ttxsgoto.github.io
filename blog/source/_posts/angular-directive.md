---
title: Angularjs Directive自定义指令
date: 2017-08-22 19:56:41
tags:
  - Directive
categories:
  - Angularjs
---
#### 创建自定义指令

- 定义指令
```js
.directive('unorderedList', function() {
    return function(scope, element, attrs) {
        // scope:指令被应用到的视图的作用域
        // element:指令被应用到的html元素
        // attrs:html元素的属性
        var data = scope[attrs["unorderedList"]];
        var name = attrs['Name'];
        }
    });
```

#### 自定义指令属性
##### restrict
可选参数，标识符在模板中作为元素，属性，类，注释或组合，默认为A
    - E 元素名使用  <my-directive>123</my-directive>
    - A 属性使用 <div my-directive> 
    - C 类名使用 <div class="my-directive"></div>
    - M 注释使用 <!-- directive: my-directive --> 
##### template
指令内容表示为html

    - 模板内容html文本，这个内容会根据replace参数的设置替换节点或只替换节点内容
    - 一个函数，可以接受两个参数tElement和tAttrs
        - tElement：是指使用此指令的元素
        - tAttrs：实例的属性
```html
<hello-world title = '这是一个directive'></hello-world>
 
app.directive("helloWorld",function(){  
        return{  
         restrict:'EAC',  
         template: function(tElement,tAttrs){  
            var _html = '';  
            _html += '<div>' +'hello '+tAttrs.title+'</div>';  
            return _html;  
         }  
     };  
 });  

```
##### templateUrl
外部模板文件
    - 加载模板所要使用的URL
    - 可以加载当前模板内对应的text/ng-template script id
    - 大体同template

##### replace
指定模板内容是否替换掉指令所应用的元素
    - 如果配置为true则替换指令所在元素,但是class和属性还是会，如果为false或者不指定，则把当前指令追加到所在元素内部
    - 对于restrict为元素E 在最终效果中是多余的，所有replace通常设置为true
    - 当replace属性为true的时候,template的最外层必须用一整个标签包裹起来
```html
<!--js-->
angular.module('myApp',[])
    .directive("myDirective",function () {
        return{
            restrict: "EACM",
            template: "<h3>hello ttxs</h3>",
            replace: true
        }
    })
<!--html-->
    <div>
        <my-directive></my-directive>
        <div my-directive></div>
        <div class="my-directive"></div>
        <!-- directive: my-directive -->
    </div>

```
 
##### compile
指令编译的三个阶段

    1. 标准浏览器API转化-将html转化成dom，即自定义的html标签需要符合html格式
    2. angular compile - 搜索匹配directive，按照priority排序，并执行directive上的compile方法
    3. angular link 执行directive上的link方法，进行scope绑定及事件绑定

- compile函数用来对模板自身进行转换，仅在编译阶段运行一次
- compile中直接返回的函数时postLink,表示link参数需要执行的函数，也可以返回一个对象里面包括preLink和postLink
- 当定义了compile参数，将忽略link参数，因为compile里返回的就是该指令需要执行的link函数
- 想在dom渲染前对它进行操作，并不需要scope参数所在所有相同directive里共享某些方法，这时应该定义compile，性能比较好

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
</head>
<body>
<div ng-app="myApp">
    <div ng-controller="firstController">
        <!--1.将div转化为dom结构-->
        <!--2. 默认的优先级为0，哪个先定义先使用-->
        <div ng-repeat="user in users" custom-directive>
        <!--<div custom-directive>-->
        </div>
    </div>
</div>
 
<script type="text/javascript" src="angularjs.js"></script>
<script type="text/javascript" src="app/index.js"></script>
</body>
</html>
 
<!--js-->
var myApp = angular.module('myApp', [])
    .directive("customDirective", function () {
        return{
            restrict: 'ECAM',
            template: '<div> {{user.name}} </div>',
            replace:true ,
            compile: function (tElement,tAttrs,transclude) {
                console.info('编译阶段...,用于修改dom元素或结构');
                tElement.append(angular.element("<span>abc</span>"))
                console.info(tElement); // 元素
                console.info(tAttrs);   // 元素属性
                console.info(transclude);   // transclude对象
                return{
                    // 编译阶段之后，指令连接到子元素之前运行
                    pre: function preLink(scope,iElement,iAttrs,controller) {
                        console.log('preLink......')
 
                    },
                    // 表示所有子元素指令都连接之后才运行
                    post: function postLink(scope,iElement,iAttrs,controller) {
                        iElement.on('click', function () {
                            scope.$apply(function () {
                                scope.user.name = 'click --> abd';  // 进行一次脏检查
                            })
                        });
                        console.log('postLink......')
                    }
 
                };  // 这里return 的就是link函数
 
                // postLink
                // return function () {
                //     console.info('compile function');
                // }
 
            },
 
            // 该link函数表示的就是postlink
            link:function () {
                console.info('Link.....')
            }
        }
    })
 
    .controller('firstController', ['$scope', function ($scope) {
        $scope.users = [
            {
                id:10,
                name:'ttxsgoto01'
            },
            {
                id:20,
                name:'ttxsgoto02'
            }
        ];
    }]);

```

##### link
指令需要处理大量DOM操作时，使用link方法；当只返回一个链接函数时，所创建的指令只能被当作一个属性来使用

    - 对特定的元素注册事件
    - 需要用到scope参数来实现dom元素行为
```js
link: function(scope, element, attrs, ctrl, linker){
    // scope: 指令所在作用域
    // element: 指令元素
    // attrs: 指令元素的属性的集合
    // ctrl: 需要和require属性一起使用，用于调用其他指令的方法,指令之间的互相通信
    // linker: transclude()函数
    // do something
}
```

##### require
字符串或者数组

    - 字符串代表另一个指令的名字，作为link函数的第四个参数
    - 对应前缀查找控制器的行为
        
        - 没有前缀，指令会在自身提供的控制器中进行查找，如果找不到任何控制器，则会抛出一个error
        - ？如果在当前的指令没有找到所需的控制器，则会将null传给link连接函数的第四个参数
        - ^如果在当前的指令没有找到所需的控制器，则会查找父元素的控制器
        - ?^组合

##### priority
指令的优先级，可选参数，若在单个DOM元素上有多个指令，则优先级高的先执行

##### terminal
bool型，可选参数，true/false ，若设置为true，则优先级低于此指令的其他指令则无效，不会被调用优先级相同任然会执行


##### scope
bool值或者对象，可选参数，默认为false，表示继承父级作用域
- 如果值为true，表示继承父作用域，并创建自己的作用域(子作用域),即使同一个控制器里数据也不共享
- 如果为对象，{}则表示创建一个全新的隔离作用域,不能使用父级对应的属性
   通过绑定策略来访问父作用域的属性:
   - 通过属性值进行绑定，可读取控制器中定义的属性值，使用@来进行单向文本（字符串）绑定，单项读取父级元素不能改变，这里引用的父级的属性只能是字符串，不能为对象，左右两边都是属性
        ```html
        <div isolated-directive other-name="{{ name }}"></div> <!--{{ name }}为父作用域的值 -->
 
        angular.module('myApp')
            .directive("isolatedDirective", function () {
                return {
                    scope: {
                        name: '@otherName'
                        },
                    template: 'Name: {{ name }}'
                    };
            });
        ```
  - 使用'='创建在指令的独立作用域和外部作用域中的双向绑定对象
        ```html
        <div isolated-directive other-name="name"></div> <!--{{ name }}为父作用域的值 -->
        angular.module('myApp')
            .directive("isolatedDirective", function () {
                return {
                    scope: {
                        name: '=otherName'
                        },
                    template: 'Name: {{ name }}'
                    };
            });
        ```
注意：这里@和= 在使用上的区别，一是功能的不同，二是调用方式不同，@使用other-name={{name}},=使用other-name="name"

  - 使用'&'调用父作用域中属性包装成一个函数或者父作用域的函数，从而以函数的方式读写父作用域的属性;允许传入一个可被指令内部调用的函数
        ```html
        <div isolated-directive action="click()"></div>
 
        angular.module('myApp')
            .controller("myController", function ($scope) {
                $scope.value = "hello world";
                $scope.click = function () {
                        $scope.value = Math.random();
                    };
                })
            .directive("isolatedDirective", function () {
                return {
                    scope: {
                        action: "&"
                        },
                    template: '<input type="button" value="data" ng-click="action()"/>'
                    }
                })
        <!-- 被传入到指令action属性的click()函数在控制器中定义, 当ng-click实际触发控制器中定义的action()函数 -->
        ```
```
当为false时候，儿子继承父亲的值，改变父亲的值，儿子的值也随之变化，反之亦如此。（继承不隔离）
当为true时候，儿子继承父亲的值，改变父亲的值，儿子的值随之变化，但是改变儿子的值，父亲的值不变。（继承隔离）
当为{}时候，没有继承父亲的值，所以儿子的值为空，改变任何一方的值均不能影响另一方的值。（不继承隔离）

```
```html
<head>
    <title>Directive Scopes</title>
    <script src="angular.js"></script>
    <link href="bootstrap.css" rel="stylesheet" />
    <link href="bootstrap-theme.css" rel="stylesheet" />
    <script type="text/ng-template" id="scopeTemplate">
        <div class="panel-body">
            <p>Name: <input ng-model="data.name" /></p>
            <p>City: <input ng-model="city" /></p>
            <p>Country: <input ng-model="country" /></p>
        </div>
    </script>
    <script type="text/javascript">
        angular.module("exampleApp", [])
            .directive("scopeDemo", function () {
                return {
                    template: function() {
                        return angular.element(
                            document.querySelector("#scopeTemplate")).html();
                    },
                    scope: true,    //同一个控制器里数据也不共享
                    scope: {
                        local: "@nameprop"  //单项绑定，说明：属性local的值来自一个nameprop特性的单项绑定获得
                        local: "=nameprop" //双向绑定
                        cityFn: "&city" //&符号说明指定特性的值绑定到一个函数，左边为一个函数调用，右边为一个属性
                    } //隔离作用域
                }
            })
        .controller("scopeCtrl", function ($scope) {
            $scope.data = { name: "Adam" };
            $scope.city = "London";
        });
    </script>
 
</head>
```
##### transclude
布尔值或者字符element，默认值为false，
    true:提取包含在指令那个元素里面的内容，再将它放置在指令模板的特定位置。当我们开启transclude之后，我们就可以使用ng-transclude来指明应该在什么地方放置transclude的内容
```html
# html
<head>
    <title>Ttxsgoto</title>
    <script src="angular.js"></script>
    <script src="app.js"></script>
    <link href="bootstrap.css" rel="stylesheet" />
    <link href="bootstrap-theme.css" rel="stylesheet" />
    <script type="text/ng-template" id="transclude.html">
        <div>
            abc:{{title}}<br>
            def:<div ng-transclude></div>
        </div>
    </script>
 
</head>
<body>
    <input ng-model="title" /><br>
    <textarea cols="30" rows="4" ng-model="text"></textarea>
    <div transclude-directive>{{text}}</div>
</body>
 
# js
.directive("transcludeDirective", function () {
        return{
            restrict: "EACM",
            templateUrl: "transclude.html",
            replace: true,
            transclude: true
        }
 
    })
```

##### controller
可以为字符串或者函数，可以直接在指令内部定义为匿名函数，同样可以注入任何服务
- 如果为字符串，则将字符串当做是控制器的名字，来查找注册在应用中的控制器的构造函数
- 直接在指令内部定义匿名函数

##### controllerAs
不用将属性和方法挂载到$scope上，而是this上；设置控制器别名

