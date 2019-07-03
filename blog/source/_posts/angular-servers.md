---
title: Angularjs Services常用服务
date: 2017-08-20 20:22:02
tags:
  - Services
categories:
  - Angularjs
---
- Constant
- Value
- Factory
- Service
- Run
- Provider
- Decorator

应用里大部分的业务逻辑和持久化数据都应该放在service里
service可以用来永久保存应用的数据，并且这些数据可以在不同的controller之间使用

#### Constant
- 定义常量，从注册后就不会在改变
- constant创建服务返回一个json对象,这个对象里可以有参数,可以有方法,一般constant创建的服务不会去修改它的内容
- 可以在注入到任何方法中调用
- constant服务不能通过decorator进行装饰
```js
angular.module('myApp', [])
  .constant('getData',{
        url:'http://localhost:5500/products',
        name:'ttxs',
        age:28,
        getId:function(){
            return 1
        }
    })
```

#### Value
- value创建服务返回一个json对象,这个对象里可以有参数,可以有方法,如果属性和方法需要被修改内容,就用value来创建服务
- 可以注入到controller，directive
- value可以被装饰

constant和value主要就是用于存放一些数据或方法以供使用,区别是constant一般是存放固定内容,value存放可能会被修改的内容

#### Factory
- 一个可注入的函数，调用factory时只是调用普通的function，所以factory可以返回任何东西，函数需要有返回值obj，而service可以不用返回
- Factory 一般就是创建一个对象，然后在对这个对象添加方法与数据，最后将些对象返回即可
- 和constant,value的区别:factory服务是有一个处理过程,经过这个过程,才返回结果的

```js
angular.module('myApp', [])
    .factory('getData',function(){
        var myname = 'ttxs';
        var age = 28;
        var id = 1;
        return {
            name: myname,
            age: age,
            getId: function(){
                return id
            }
        }
    });
```
#### Service
- 可注入的构造器，它用在controller中通信或者共享数据,适合使用在功能控制比较多的service里面
- service里可以不返回东西，因为angularJS会调用new关键字来创建对象
- seivce定义的服务不能在.config中使用！只有provider定义的才可以
- 这里的值都应该使用this定义
- 自定义服务return 返回值必须为对象,不能为字符串,数字等
```js
var app = angular.module('myApp', []);
app.controller('myController', function($scope, myService) {
    $scope.getPrivate = function() {
        alert(myService.getPrivate());
    };
    $scope.getPUbluc = function() {
        alert(myService.variable);
    };
});
 
app.controller('myController2', function($scope, myService) {
	// do something
});
 
app.service('myService', function() {
     console.log('myService');
    var privateValue = "I am Private";
    this.variable = "This is public";
    this.getPrivate = function() {
        return privateValue;
    };
});
```
#### Run
- 在注入启动之后执行某些操作，而这些操作需要在页面对用户可用之前执行，使用run方法；即在config方法之后controller方法之前调用
- 使用场景：远程加载模板，需要在使用前加入缓存，或者在操作前判断用户是否登录，未登录需先跳转到登录页面


#### Provider
- $provide服务负责告诉Angular如何创造一个新的可注入的东西：即服务;服务会被叫做供应商的东西来定义，你可以使用$provide来创建一个供应商。你需要使用$provide中的provider()方法来定义一个供应商，同时你也可以通过要求$provide被注入到一个应用的config函数中来获得$provide服务
- provider必须有一个$get方法，是所有封装函数都是由provider封装的
- provider是一个可配置的factory

```js
var myApp = angular.module('myApp',[],function($provide){
 
    // 自定义服务
    $provide.provider('CustomService',function(){
 
        this.$get = function(){
            return {
                message : 'CustomService Message'
            }
        }
    });
 
    // 自定义工厂,返回值为任意值
    $provide.factory('CustomFactory',function(){
        return [1,2,3,4,5,6,7];
    });
 
    // 自定义服务, 返回值必须为对象,不能为字符串,数字等
    $provide.service('CustomService2',function(){
        return ['xxx'];
        // return 'abc';
    })
``` 

#### 总结
1) 服务的实例被注入到控制器以后,都是一个引用对象,无论被注入多少个控制器中,实际都指向同一个对象,无论修改其中的哪一个,其它所有的服务都会被改变
2) constant服务不能通过decorator进行装饰
3) 固定的参数和方法,使用constant;可能被修改的参数和方法,使用value
4) 逻辑处理后得到的参数或方法,使用factory
5) Service 是用"new"关键字实例化的。因此，你应该给"this"添加属性，然后 service 返回"this"。你把 service 传进 controller 之后，在controller里 "this" 上的属性就可以通过 service 来使用
6) Providers 是唯一一种你可以传进 .config() 函数的 service。当你想要在 service 对象启用之前，先进行模块范围的配置，那就应该用 provider
7) Factory/service是第一个注入时才实例化，而provider不是，它是在config之前就已实例化





















