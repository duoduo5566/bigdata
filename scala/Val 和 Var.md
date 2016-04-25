# Val 和 Var

标签（空格分隔）： scala

---

Scala允许你来决定一个变量是否可变。不可变(Immutable )表示只支持只读操作，而可变(mutable)则支持读写操作。

不可变的变量在进行声明时，需要使用`val`关键字，如下：

    val age:Int = 22

在这里，变量age在声明的时候就已经初始化了一个值。由于它声明时使用了`val`关键字，age变量就不能够再重新分配一个新值了。任何试图这么做的操作，结果都将会返回一个 `reassignment to val` 错误。

可变变量在声明时，使用`var`关键字。和val不一样的是，使用`var`声明的变量可以重新指定不同的值，或者指向不同的对象。但是，它们在声明的时候都必须进行初始化操作。

    var age:Int = 22
    age = 35
对于`val`变量与`var`变量在声明时需要初始化这个规则，这里有一个例外。当它们被用作构造参数时，`var`与`val`声明的变量将在对象初始化时才进行初始化操作。另外，子类中可以覆盖父类中声明的`val`变量。

该你动手了！记住，`var`变量是可以重新赋值的。

    var a = 5
    a should be(_)
    a = 7
    a should be(_)

    // What happens if you uncomment these lines?
    // a = 7
    // a should be (7)

记住`val`并没有锁定变量的内部状态，仅是锁定它的分配。让我们思考下一个用`val`声明的数组。

    val stringArray:Array[String] = new Array(6)
stringArray数组是可以被修改的，但是它的引用不能够被修改而指向其它的数组。比如：

    stringArray = new Array(33)
结果将返回一个`reassignment to val`错误，但是

    stringArray(3) = "foo"
将不会返回任何错误。