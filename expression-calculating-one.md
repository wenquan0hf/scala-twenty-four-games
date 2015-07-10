#表达式计算(一)
前面我们基本对 Scala 编程做了完整的介绍，教程可以参见[ Scala 开发教程](http://www.imobilebbs.com/wordpress/%E6%95%99%E7%A8%8B/scala%E5%BC%80%E5%8F%91%E6%95%99%E7%A8%8B)，但是实践才是最有效的学习方法，我们可以通过一些较为实用的例子来练习 Scala 编程。
我们首先通过我们小时候经常玩的算 24 ，通过 scala 实现算二十四，编程计算二十点的算法大多为穷举法，我们先从最简单的算法开始，计算表达式，还记得以前学习数据结构使用C语言实现算二十四，需要把表达式首先转成逆波兰形式，然后通过栈来计算表达式的值，我们看看如果通过Scala的函数化编程来实现表达式的计算。
算二十四使用基本的四则运算，加减乘除，外加括号。为简单起见，我们先不考虑带括号的四则运算，后面再逐步扩展，算 24 ，比较有名的一个例子是 5 5 5 1 ，我们最终的结果是需要使用 Scala 实现二十四算法，给出 5 5 5 1 的算法，并可以计算任意四个数，如果有解，给出解，如果穷举后无解，给出无解的结果。

四则运算具有以下二元计算基本表现形式：
（表达式） op (表达式）

其中表达式可以是个数字，或是另外一个表达式， op 可以为 + – * /。

比如 3 + 2 ; 3 + 4*3 等等。
对于此类二元运算，我们可以设计一个 Extractor ，分解出二元表达式的左右操作数，还记的我们之前介绍的 Email 的 Extractor 的例子吗？[Scala 专题教程-Extractors(1):分解Email地址的例子](http://www.imobilebbs.com/wordpress/archives/5056)
我们模仿分解 Email 的例子，写出一个分解加法的 Extractor ，如下：
```
object Add {
	def apply(expr1:String,expr2:String) = expr1 + "+" + expr2
	def unapply(str:String) :Option[(String,String)] ={
		val parts = str split "\\+"
		if(parts.length==2) Some(parts(0),parts(1)) else None
	}
}
```
测试一下看看
```
scala> Add.unapply("3+2")
res1: Option[(String, String)] = Some((3,2))
```
可以看到 Add 对象成功分解了表达式 3+2 的两个操作数 3 和 2。

设计一个 eval 函数，来计算加法的结果，为简单起见，我们先只考虑整数的情况
```
def eval(str:String):Int = str match{
	case Add(expr1,expr2) => eval(expr1) + eval(expr2)
	case _ => str toInt
}
```
这个 eval 函数计算一个表达式的值（目前只支持加法），如果输入的是一个加法表达式，分解后计算每个操作时的值，如果不能分解，那么这是个整数，输出该整数：
```
scala> eval("3+5")
res0: Int = 8
    
scala> eval("4+8")
res1: Int = 12
```

计算结果成功，不过这个实现有些局限，我们看看 3+5+3，什么情况：
```
scala> eval("3+5+3")
java.lang.NumberFormatException: For input string: "3+5+3"
  at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
  at java.lang.Integer.parseInt(Integer.java:492)
  at java.lang.Integer.parseInt(Integer.java:527)
  at scala.collection.immutable.StringLike$class.toInt(StringLike.scala:241)
  at scala.collection.immutable.StringOps.toInt(StringOps.scala:30)
  at .eval(<console>:10)
  ... 32 elided
```

出错了，这是因为 Extractor 对象 Add 的实现，只考虑了表达式只包含一个“+”的情况，我们修改下 Add 的实现，不使用 split（使用split,如果有有两个+以上，parts.length 就>2)，而是使用 indexOf 来查找”+”号：
```
object Add{
  val op:String="+"
  def apply(expr1:String,expr2:String) = expr1 + op + expr2
  def unapply(str:String) :Option[(String,String)] ={
    val index=str indexOf(op)
    if(index>0)
      Some(str substring(0,index),str substring(index+1))
    else None
  }
}
```
再来计算”3+5+3″等表达式的值：
```
scala> eval("3+5+3")
res0: Int = 11

scala> eval("1+2+3+4+5+6+7+8+9+10")
res1: Int = 55
```
同样，我们可以复制Add，修改op的值，实现 Subtract，Divide，Multiply，为避免重复代码，我们可以设计一个抽象 Trait：
```
trait BinaryOp{
  val op:String
  def apply(expr1:String,expr2:String) = expr1 + op + expr2
  def unapply(str:String) :Option[(String,String)] ={
    val index=str indexOf(op)
    if(index>0)
      Some(str substring(0,index),str substring(index+1))
    else None
  }
}

object Multiply  extends {val op="*"} with BinaryOp
object Divide  extends {val op="/"} with BinaryOp
object Add  extends {val op="+"} with BinaryOp
object Subtract  extends {val op="-"} with BinaryOp
```
同样修改 eval 的实现如下：
```
def eval(str:String):Int = str match {
 
    case Add(expr1,expr2) => eval(expr1)  +  eval(expr2)
    case Subtract(expr1,expr2) => eval(expr1)  -  eval(expr2)
    case Multiply(expr1,expr2) => eval(expr1)  * eval(expr2)
    case Divide(expr1,expr2) => eval(expr1)  /  eval(expr2)
    case _ => str toInt

  }
```
测试如下：
```
scala> eval("4*5-2/2")
res3: Int = 19

scala> eval("4*5-5*4")
res4: Int = 0

scala> eval("4*5-5*4-2/2")
res5: Int = 1

scala> eval("4*5-5*4+2/2")
res6: Int = 1
```

这个简单的计算表达式的函数 eval 就这么简单的实现了，而且代码也很直观（尽管还有不少局限性），我们下篇再看看带括号的情况。