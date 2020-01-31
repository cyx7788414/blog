# 问题描述
最近在对Angular项目做压缩，但压缩后测试时经常发现有provider报错。  
具体原因，是代码压缩时，将参数简写，导致依赖注入失败。
# 示例代码
压缩前：
```js
app.register.directive('xxdirective', function($compile, $sce) {
    return {
        restrict: 'A',
        scope: {
        },
        controller: function($scope, $element) {        
        },
        compile: function($element, $attrs) {
            return {
                pre: function() {
                },
                post: function($scope, $element, $attrs, $ctrls) {       
                }
            };
        }
    };
});
```
变量名简写后：
```js
app.register.directive('xxdirective', function(r, e) {
    return {
        restrict: 'A',
        scope: {
        },
        controller: function(r, e) {        
        },
        compile: function(r, e) {
            return {
                pre: function() {
                },
                post: function(r, e, s, c) {       
                }
            };
        }
    };
});
```
可以看到，参数全部被替换成为了单字符，这就导致依赖注入无法通过参数名匹配到对应模块。
# 解决方案
为了解决这个问题，angular有一个数组替代函数的方案，来规避这种现象，即将参数以文本形式在数据中函数前列出，与函数参数一一对应，如下：
```js
app.register.directive('xxdirective', ['$compile', '$sce', function($compile, $sce) {
    return {
        restrict: 'A',
        scope: {
        },
        controller: ['$scope', '$element', function($scope, $element) {  
        //controller这里可以这么写    
        }],
        compile: function($element, $attrs) {
        //但compile这里不能用上面的方法，会报错“xx.compile is not a function”，要求为一个函数
            return {
                pre: function() {
                },
                post: function($scope, $element, $attrs, $ctrls) {
                //这里也不能
                }
            };
        }
    };
}]);
```
但这时发生了一个问题，controller属性能以数组形式进行依赖注入，但compile属性不行，如果改为数组形式，报错信息会显示要求compile的值为一个函数。  
找了一通以后，有一个不常见的Annotation方式，能够解决问题，即为function定义$inject属性，以此种形式声明注入。  
修改后的代码如下：
```js
app.register.directive('xxdirective', ['$compile', '$sce', function($compile, $sce) {
    var forCompile = function($element, $attrs) {//定义函数
            var compilePost = function($scope, $element, $attrs, $ctrls) {//定义函数
                };
            compilePost.$inject = ['$scope', '$element', '$attrs', '$ctrls'];//为function定义$inject属性，以此种形式声明注入
            return {
                pre: function() {
                },
                post: compilePost //属性仍为function
            };
        };
    forCompile.$inject = ['$element', '$attrs'];//为function定义$inject属性，以此种形式声明注入
    return {
        restrict: 'A',
        scope: {
        },
        controller: ['$scope', '$element', function($scope, $element) {
        }],
        compile: forCompile //属性仍为function
    };
}]);
```
由于数组形式的可读性比较好，建议能写数组形式时采用数组形式，Annotation方式的用来救急
# 参考文献
1. [【AngularJS学习笔记】AngularJS 压缩JS](http://blog.csdn.net/qq673318522/article/details/50677992)