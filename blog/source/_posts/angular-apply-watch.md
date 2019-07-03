---
title: Angularjs $apply和$watch方法
date: 2017-09-16 20:36:13
tags:
  - Angularjs
categories:
  - Frontend
---
#### $apply说明
手动触发脏检查，当我们更改一个不在AngularJS执行上下文中的数据模型(model)，需要人为的调用$apply()来提醒AngularJS数据发生变化

```html
<div ng-controller="firstController">
    {{date}}
</div>
 
<script>
$scope.date = new Date();
    setInterval(function(){
        $scope.$apply(function()    {
                $scope.date = new Date();
                // 触发脏检查
            })
    },1000);
</script>
```
#### digest说明
当调用ng开头的指令或者服务，在这种情况下，AngularJS就会自动调用$digest()触发$digest循环。当$digest循环开始的时候，他就会启动每一个监听器(watcher)。这些监听器(watcher)会去检查当前的数据模型(model)中的值是否与最后一次计算的值相同，如果不相同，那么，对应的监听函数就会被执行
#### $watch说明
- 在digest执行时，如果watch观察到value和上次执行时值不一样时，就会被触发
- angularjs内部的watch实现了页面随model的及时更新
- $watch(watchFn, watchAction, deepWatch)

    - watchFn 表达式或函数的字符串
    - watchAction(newvalue,oldvalue,scope) watchFn发生变化时被调用
    - deepWatch 可选布尔值命令检查被监控的对象的每个属性是否发生变化

- $watch会返回一个函数，想要注销这个watch可以使用函数

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
</head>
<body>
    <div ng-app="">
 
        <div ng-controller="firstController">
            <input type="text" value="" ng-model="name"/>
            改变次数:{{count}}-{{name}}
        </div>
    </div>
    <script type="text/javascript" src="app/index.js"></script>
<script type="text/javascript" src="angularjs.js"></script>
</body>
</html>
   
<script>
var firstController = function($scope){
 
    $scope.name = 'ttxsgoto';
    $scope.data = {
        name :'ttxs',
        count:20
    }
    $scope.count = 0;
  
    // 监听一个model 当一个model每次改变时 都会触发第2个函数
    $scope.$watch('name',function(newValue,oldValue){
  
        ++$scope.count;
  
        if($scope.count > 30){
            $scope.name = '已经大于30次了';
        }
    });
  
    $scope.$watch('data',function(){
 
    },true)
}
</script>
```





