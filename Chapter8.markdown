#备忘与诀窍

目前为止，之前的章节已经覆盖了Angular所有功能结构中的大多数，包括指令,服务,控制器,资源以及其它内容.但是我们知道有时候仅仅阅读是不够的.有时候，我们并不在乎那些功能机制是如果运行的,我们仅仅想知道如何用AngularJS去做实现一个具体功能。

在这一章中，我么视图给出完整的样例代码,并且对这些样例代码仅仅给出少量的信息和解释，这些代码解决是我们在大多数Web应用中碰到的通用问题.这些代码没有具体的先后次序,你尽可以跳到你关心的小节先睹为快或者按着本书内容顺序阅读,选那种阅读方式由你决定.

这一章中我们将要给出代码样例包括以下这些：

1、封装一个jQuery日期选择器(DatePicker)
2、团队成员列表应用:过滤器和控制器之间的通信
3、AngularJS中的文件上传
4、使用socket.IO
5、一个简单的分页服务.
6、与服务器后端的协作

##封装一个jQuery日期选择器

这个样例代码文件可以在我们GitHub页面的chatper8/datepicker目录中找到.

在我们开始实际代码之前，我们不得不做出一个设计决策：我们的这个组件的外观显示和交互设计应该是什么样子,假设我们想定义的的日期选择器在HTML里面使用像以下代码这样:

    <input datepicker ng-model="currentDate" select="updateMyText(date)" ></input>
    
也就是说我们想修改input输入域,通过给她添加一个叫datepicker的属性,来给它添加一些更多的功能(就像这样:它的数据值绑定到一个model上,当一个日期被选择的时候,输入域能被提醒修改).那么我们如何做到这一点哪?

我们将要复用现存的功能组件:jQueryUI的datepicker(日期选择器),而不是我们从头自己构建一个日期选择器.我们只需要把它接入到AngularJS上并且理解它提供的钩子(hooks):

    angular.module('myApp.directives', [])
        .directive('datepicker', function() {
            return {
            // Enforce the angularJS default of restricting the directive to
            // attributes only
            restrict: 'A',
            // Always use along with an ng-model
            require: '?ngModel',
            scope: {
                // This method needs to be defined and
                // passed in to the directive from the view controller
                select: '&' // Bind the select function we refer to the
                            // right scope
            },
            link: function(scope, element, attrs, ngModel) {
                if (!ngModel) return;

                var optionsObj = {};
                optionsObj.dateFormat = 'mm/dd/yy';
                var updateModel = function(dateTxt) {
                    scope.$apply(function () {
                        // Call the internal AngularJS helper to
                        // update the two-way binding
                        ngModel.$setViewValue(dateTxt);
                    });
                };
                
                optionsObj.onSelect = function(dateTxt, picker) {
                    updateModel(dateTxt);
                    if (scope.select) {
                        scope.$apply(function() {
                            scope.select({date: dateTxt});
                        });
                    }
                };

                ngModel.$render = function() {
                    // Use the AngularJS internal 'binding-specific' variable
                    element.datepicker('setDate', ngModel.$viewValue || '');
                };
                element.datepicker(optionsObj);
            }
        };
    });

    上面代码中的大多数都非常简单直接,但是我们还是来看一下其中一些较重要的部分.

###ng-model

我们可以得到一个ng-model属性,这个属性的值将会被传入到指令的链接函数中.`ng-model`(这个属性对于这个指令运行是必须的,因为指令定义中的`require`属性定义--见上代码)这个属性帮助我们定义属性和绑定到`ng-model`上的(js)对象在指令的上下文中的行为机制.这儿有两点你需要注意一下:

`ngModel.$setViewValue(dateTxt)`

当Angular外部某些事件(在这个示例中就是jQueryUI日期选择器组件中某日期被选定的事件)发生时,上面这条语句会被调用.这样就可以通知AngularJS更新模型对象.这种语句一般是在某个DOM事件发生时被调用.

`ngModel.$render`

这是`ng-model`的另外一部分.这个可以协调Angular在模型对象发生变化时如何更新视图.在我们的示例中,我们仅仅给jQueryUI日期选择器传递了发生了改变的日期值.

###绑定选择值

取代使用属性值然后用它计算成作用域对象(scope)的一个字符串属性的做法(在这个案例中，嵌套在指令内部的函和对象不是直接可操作的),我们使用了函数方法绑定("&"作用域对象绑定-注意看上面scope对象定义部分代码).这就在scope作用域对象上建立了一个叫select的方法.
