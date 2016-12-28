
JavaScript作为一门灵活的语言，也是面向对象的，所以许多设计模式可以在Javascript中很轻松地实现。比如命令模式，直接把命令函数作为参执行函数的参数，然后把执行函数赋值给调用者的一个属性就可以了，简直不要太方便。

但是也有让我挠头的，比如今天要说的单例模式。

Java中常见的单例模式是这样的：

    public class ClassicSingleton
    {
        private static ClassicSingleton uniqueInstanse; // 静态的单例变量
        
        private ClassicSingleton(){ // 私有化构造器
        }
        
        // 公有的获取单例的方法，保证构造器只被初始化一次
        public static ClassicSingleton getInstance(){ 
            if (uniqueInstanse == null)
            {
                uniqueInstanse = new ClassicSingleton();
            }
            return uniqueInstanse;
        }
    }

代码中的注释已经解释的比较清楚了，我们只能通过`getInstance()`方法获取这个类的实例，而且保证了内存中只会有这么一个实例。

但是事情到了JavaScript这边就有些麻烦了，因为JS中没有`class`关键字，对象是创建是以函数的形式；而且也没有`private`关键字，在作用域内，所有的变量和函数的封装程度都是一样的。

所以要用到闭包。

闭包的概念，wiki是这么解释的：
> In computer science, a closure is a function together with a referencing environment for the nonlocal names (free variables) of that function.

含义就是**一个函数以及引用它的环境可以赋值给一个（外部）自由变量。**

一般来说，JavaScript的作用域是函数内部可以调用外部变量，但是外部不能调用内部变量。

闭包给了访问局部变量的一种方法。

    var db = (function () { // 第一对括号表示将函数表达式化
    // 创建一个隐藏的object, 这个object持有一些数据
    // 从外部是不能访问这个object的
        var data = {};
    // 创建一个函数, 这个函数提供一些访问data的数据的方法
        return function (key, val) {
            if (val === undefined) {
                return data[key]
            } // 如果var没有定义，这是个getter
            else {
                return data[key] = val
            } // 否则，这是个setter
        };
    // 我们可以调用这个匿名方法
    // 返回这个内部函数，它是一个闭包
    })(); // 这个匿名函数已经执行，并把返回的函数赋值给db
    
    // 测试代码
    db('x'); // 返回 undefined
    db('x', 1); // 设置data['x']为1
    db('x'); // 返回 1
    // 我们不可能访问data这个object本身
    // 但是我们可以设置它的成员
    
如上所示，db被赋值为一个函数，这个函数是用来访问局部变量`data`的，而这个`data`是在一个已经执行过的匿名函数里声明，直接取取不到，所以相当于私有化了，而它的访问函数赋给了外部变量，相当于是公有化了。

这样就给我们实现单例模式提供一个思路，具体代码如下：

    var singleA = (function () { // 第一个括号表示闭包
        var SingleAClass = function (message) { // 私有化的构造器
            this.msg = message;
            this.setMsg = function(msg){
                this.msg = msg;
            }
        };
        var instanceA; // 放在匿名闭包里，相当于私有化了
        var info = {
        // 这是singleA的成员方法，封装了singleAClass的创建过程，公有化的工厂
            getInstanceA: function (message) { 
                if (!instanceA) {
                    instanceA = new SingleAClass(message);
                }
                return instanceA;
            }
        };
        return info; // 由于闭包载入后执行，现在singleA = info
    })(); // 这一对括号表示将第一行闭包的函数立即执行
    var singleB = {
        getA: function (msg) {
            return singleA.getInstanceA(msg);
        },
        showA:function (a){
            alert(a.msg);
            a = null; // 并没有释放全局单例，只是释放了参数a
        },
        getAndShowA:function(msg){
            a = this.getA(msg);
            this.showA(a);
        }
    };
    // 执行测试：
    singleB.getAndShowA("0");// 打印0
    singleB.getAndShowA("1"); // 还是会打印0,因为instance已经存在
    var a = singleB.getA("2"); // 并没有创建新实例，而是返回0的实例
    a.setMsg("3"); // 这次是取出了单例，并设置为3
    singleB.showA(a); // 打印3

> 注意：如上代码所示，我们在调用者`SingleB`里是没办法释放单例的资源的，这也是所有使用闭包的情况下会出现的事——容易造成内存泄露，尤其是容易出现循环引用，所以在工程开发中要注意。
