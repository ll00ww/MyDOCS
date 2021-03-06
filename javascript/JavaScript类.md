### JavaScript类(ES5)
在JavaScript中也可以定义对象的类，让每个对象共享某些属性，这种“共享”的特性是非常有用的。
类的成员或实例都包含一些属性，用以存放或定义它们的状态，其中有些属性定义了它们的行为(通常
称为方法)。这些行为通常是由类定义的，而且为所有实例所共享。在JavaScript中，类的实现是基于
原型继承机制的。如果两个实例都从同一个原型对象上继承了属性，我们说它们是同一个类的实例。

###### 对象创建表达式
在JavaScript中使用new关键字来创建一个对象，当使用new关键字调用一个函数时，首先创建一个空
对象，然后JavaScript通过指定参数并将这个新对象作为this的值来调用一个指定的函数。这个函数
可以使用this来初始化这个新创建对象的属性。那些被当成构造函数的函数不会返回一个值，并且这
个新创建并被初始化后的对象就是整个对象创建表达式的值。如果一个构造函数确实返回了一个对象
值，那么这个对象作为整个对象创建表达式的值，而新创建的对象就被废弃了。

###### 构造函数
使用关键字new来调用构造函数会自动创建一个新对象，因此构造函数本身只需要初始化这个新对象的
状态即可。调用构造函数的一个重要特征是，构造函数的prototype属性被用做新对象的原型。这意味
着通过同一个构造函数创建的所有对象都继承自一个相同的对象，因此它们都是同一个类的成员。

###### 构造函数标识
原型对象是类的唯一标识，当且仅当两个对象继承自同一个原型对象时，它们才是属于同一个类的实例
。而初始化对象的状态的构造函数则不能作为类的标识，两个构造函数的prototype属性可能指向同一个
原型对象。那么这两个构造函数创建的实例是属于同一个类的。

###### 原型对象的constructor属性
任何JavaScript函数都可以用做构造函数，并且调用构造函数式需要用到一个prototype属性的。因此，
每个JavaScript函数方法都自动拥有一个prototype属性。这个属性的值是一个对象，这个对象包含唯一
一个不可枚举属性constructor。原型对象和构造函数的关系如图（1-1）所示：

![图1-1][JavaScript_constructor]

###### JavaScript中Java式的类继承
* 构造函数象：构造函数为JavaScript的类定义了名字。任何添加到这个构造函数对象中的属性都是类字
段和类方法(如果属性值是函数的话就是类方法)。
* 原型对象：原型对象的属性被类的所有实例所继承，如果原型对象的属性值是函数的话，这个函数就作为
类的实例的方法来调用。
* 实例对象：类的每个实例都是一个独立的对象，直接给这个实例定义的属性是不会为所有实例对象所共享
的。定义在实例上的非函数属性，实际上是实例的字段。
在JavaScript中定义类的步骤可以缩减为一个分三步的算法。第一步，先定义一个构造函数，并设置初始化
新对象的实例属性。第二步，给构造函数的prototype对象定义实例的方法。第三步，给构造函数定义类字段
和类属性。

###### 类和类型的判断
typeof运算符是一个一元运算符，放在其单个操作数的前面，操作数可以是任意类型。返回值为标识操作数
类型的一个字符串。表1-1列出了任意值在typeof运算后的返回值：

| X | typeof x |
|:----:|:--------:|
|undefined|"undefined"|
|null|"object"|
|true或false|"boolean"|
|任意数字或NaN|"number"|
|任意字符串|"string"|
|任意内置对象(非函数)|"object"|
|任意宿主对象|由编译器各自实现的字符串，但不是"undefined"、"boolean"、"number"或"string"|

由于所有对象和数组的typeof运算结果是"object"而不是"function"，因此它对于区分对象和其他原始值来
说是很有帮助的。如果想区分对象的类，则需要使用其他的手段，比如使用instanceof运算符、class特性以
及constructor属性。

对象的类属性(class attribute)是一个字符串，用以表示对象的类型信息。ECMAScript3和ECMAScript5都未
提供设置这个属性的方法，并只有一种间接的方法可以查询它。默认的toString()方法(继承自Object.prototype)
返回了如下这种格式的字符串：`[object class]`，因此，要想获得对象的类，可以调用对象的toString()方法，
然后提取已返回字符串的第8个到倒数第二位置之间的字符。由于toString()方法经常被重写，必须间接的调用
Function.call方法，可以查考如下方法：
```
function classof(o) {
    if (o === null) return "Null";
    if (o === undefined) return "Undefined";
    return Object.prototype.toString.call(o).slice(8,-1);
}
```
这个函数对null和undefined进行了特殊处理，对于内置构造函数和宿主对象都可以很好的区分其类型，但是对于
对象直接量和Object.create创建的对象的类属性是"Oject"，那些自定义构造函数创建的对象也是一样，类属性也
是Object，因此对于自定义的类来说，没办法通过类属性类区分对象的类。

instanceof运算符做操作数是待检测其类的对象，右操作数是定义类的构造函数。如果o继承自c.prototype，则表
达式`o instanceof c`的值为true。这里的继承可以不是直接继承，如果o所继承的对象继承自另一个对象，后一个
对象继承自c.prototype，这个表达式的运算结果也是true。构造函数是类的公共标识，但原型是唯一的标识。尽管
instanceof运算符的右操作数是构造函数，但计算过程实际上是检测对象的继承关系，而不是检测对象的构造函数。
如果你想检测对象的原型链上是否存在某个特定的原型对象，可以使用isPrototypeOf()方法。instanceof运算符和
isPrototype()方法的缺陷是，我们无法通过对象来获得类名，只能检测对象是否属于指定的类名。

另一种识别对象是否属于某个类的方法时使用constructor属性。因为构造函数是类的公共标识，所以最直接的方法
就是使用constructor属性。同样JavaScript中也并非所有的对象都包含constructor属性。在每个新创建的函数原型
上默认会有constructor属性，使用constructor属性检测对象属于某个类的技术的不足之处和instanceof一样。在多
个执行上下文的场景中它是无法正常工作的。

###### 标准转换方法
* toString()：当对象被需要当成字符串使用时，JavaScript会自动调用这个方法返回一个表示这个对象的字符串，如果
没有实现这个方法，类会默认从Object.prototype中继承toString()方法。
* valueof()：当对象需要被当成数字使用时，会自动调用valueof()方法，将对象转换成原始值。
* toJSON()：这个方法是由JSON.stringify()自动调用。JSON格式用于序列化对象，可以处理原始值、数组和纯对象。它
和类无关，当对一个对象执行序列化操作时，它会忽略对象的原型和构造函数。

###### 子类
在面向对象编程中，类B可以继承自另一个类A。我们将A称为父类(superclass)，将B称为子类(subclass)。B的实例从A继
承了所有的实例方法。类B可以定义自己的实例方法，有些方法可以重载类A中的同名方法，如果B的方法重载A中的方法，
B中的重载方法可能会调用A中的重载方法，这种做法称为“方法链”(method chaining)。同样，子类的构造函数B()有时需
要调用父类的构造函数A()，这种做法称为“构造函数链”(constructor chaining)。子类还可以有子类，当涉及类的层次
结构时，往往需要定义抽象类(abstract class)。抽象类中定义的方法没有实现。抽象类中的抽象方法时在抽象类的具体子
类中实现的。

JavaScript的对象可以从类的原型对象中继承属性(通常继承的是方法)。如果O是类B的实例，B是A的子类，那么O也是一定
从A中继承了属性。为此，首先要确保B的原型对象继承自A的原型对象。参看如下代码：
```
    B.prototype = inherit(A.prototype);
    B.prototype.constructor = B;
```
这两行代码是在JavaScript中创建子类的关键。如果不这样做，原型对象仅仅是一个普通对象，它只继承自Object.prototype，
这意味着你的类和所有的类一样是Object的子类。

[JavaScript_constructor]: ../image/JavaScript_constructor.png "图1-1"