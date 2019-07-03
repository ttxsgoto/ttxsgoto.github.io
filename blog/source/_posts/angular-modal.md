---
title: Angularjs $modal模态框
date: 2017-09-02 14:23:07
tags:
  - Angularjs
categories:
  - Frontend
---
记录angular模态框的使用
#### html
```html
<!doctype html>
<html ng-app="Modaldemo">
<head>
    <script src="http://ajax.googleapis.com/ajax/libs/angularjs/1.6.1/angular.js"></script>
    <script src="http://angular-ui.github.io/bootstrap/ui-bootstrap-tpls-2.5.0.js"></script>
    <script src="app.js"></script>
    <link href="http://netdna.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
 
<div ng-controller="ModalDemoCtrl" class="modal-demo">
    <script type="text/ng-template" id="myModalContent.html">
        <div class="modal-header">  //头部
            <h3 class="modal-title" id="modal-title">I'm a modal!</h3>
        </div>
        <div class="modal-body" id="modal-body">    //中部
            <ul>
                <li ng-repeat="item in items">
                    <a href="#" ng-click="$event.preventDefault(); selected.item = item">{{ item }}</a>
                </li>
            </ul>
            Selected: <b>{{ selected.item }}</b>
        </div>
        <div class="modal-footer">  //底部
            <button class="btn btn-primary" type="button" ng-click="ok()">OK</button>
            <button class="btn btn-warning" type="button" ng-click="cancel()">Cancel</button>
        </div>
    </script>
 
    <script type="text/ng-template" id="stackedModal.html">
        <div class="modal-header">
            <h3 class="modal-title" id="modal-title-{{name}}">The {{name}} modal!</h3>
        </div>
        <div class="modal-body" id="modal-body-{{name}}">
            Having multiple modals open at once is probably bad UX but it's technically possible.
        </div>
    </script>
 
    <button type="button" class="btn btn-default" ng-click="open()">Open me!</button>
    <button type="button" class="btn btn-default" ng-click="open('lg')">Large modal</button>
    <button type="button" class="btn btn-default" ng-click="open('sm')">Small modal</button>
 
    <button type="button"
            class="btn btn-default"
            ng-click="open('sm', '.modal-parent')">
        Modal appended to a custom parent
    </button>
    <button type="button" class="btn btn-default" ng-click="toggleAnimation()">Toggle Animation ({{ animationsEnabled }})</button>
    <button type="button" class="btn btn-default" ng-click="openMultipleModals()">
        Open multiple modals at once
    </button>
    <div ng-show="selected">Selection from a modal: {{ selected }}</div>
    <div class="modal-parent">
    </div>
</div>
</body>
</html>
```
#### js
```js
myApp = angular.module('Modaldemo', ['ui.bootstrap']);
myApp.controller('ModalDemoCtrl',['$scope','$uibModal','$log','$document', function ($scope, $uibModal, $log, $document) {
    $scope.items = ['item1', 'item2', 'item3'];
    $scope.animationsEnabled = true;
 
    $scope.open = function (size, parentSelector) {
        var modalInstance = $uibModal.open({
            animation: $scope.animationsEnabled,    //打开时的动画开关
            ariaLabelledBy: 'modal-title',
            ariaDescribedBy: 'modal-body',
            backdrop: true,                          //控制弹框背景是否为暗影，默认为true
            templateUrl: 'myModalContent.html',     //模态框的页面内容
            // template: '<div>abc</div>',          //用于显示html标签
            keyboard: true,                         //当按下Esc时，模态对话框是否关闭，默认为ture
            controller: 'ModalInstanceCtrl',        //模态框的控制器,是用来控制模态框
            // controllerAs: 'ModalDemoCtrl',
            size: size,                             //模态框的大小尺寸
            appendTo: angular.element(document.getElementsByTagName('body')[0]),
            resolve: {                              //定义一个成员并将他传递给$modal指定的控制器,将主控制器中的参数传到模态框控制器中
                items: function () {                //items回调函数
                    return $scope.items;
                }
            }
        });
 
        modalInstance.result.then(function (selectedItem) { //接收模态框返回值的函数,确认处理函数
            console.log('selectedItem-->',selectedItem);    //模态框的返回值
            $scope.selected = selectedItem;
        }, function () {                                    //取消处理函数
            $log.info('Modal dismissed at: ' + new Date());
        });
    };
 
    $scope.openComponentModal = function () {
        var modalInstance = $uibModal.open({
            animation: $scope.animationsEnabled,
            component: 'modalComponent',
            resolve: {
                items: function () {
                    return $scope.items;
                }
            }
        });
        modalInstance.result.then(function (selectedItem) {
            $scope.selected = selectedItem;
        }, function () {
            $log.info('modal-component dismissed at: ' + new Date());
        });
    };
 
    $scope.openMultipleModals = function () {
        $uibModal.open({
            animation: $scope.animationsEnabled,
            ariaLabelledBy: 'modal-title-bottom',
            ariaDescribedBy: 'modal-body-bottom',
            templateUrl: 'stackedModal.html',
            size: 'sm',
            controller: function($scope) {
                $scope.name = 'bottom';
            }
        });
 
        $uibModal.open({
            animation: $scope.animationsEnabled,
            ariaLabelledBy: 'modal-title-top',
            ariaDescribedBy: 'modal-body-top',
            templateUrl: 'stackedModal.html',
            size: 'sm',
            controller: function($scope) {
                $scope.name = 'top';
            }
        });
    };
 
    $scope.toggleAnimation = function () {
        $scope.animationsEnabled = !$scope.animationsEnabled;
    };
}]);
 
// modal controller
myApp.controller('ModalInstanceCtrl',['$scope','$uibModalInstance','items', function ($scope, $uibModalInstance, items) {
    $scope.items = items;
    $scope.selected = {
        item: $scope.items[0]
    };
 
    $scope.ok = function () {
        console.log('ok functon');
        $uibModalInstance.close($scope.selected.item);  //关闭模态窗口并传递一个结果
    };
 
    $scope.cancel = function () {
        console.log('cancel functon');
        $uibModalInstance.dismiss('cancel');    //撤销模态关闭方法并传递一个原因
    };
}]);
```
