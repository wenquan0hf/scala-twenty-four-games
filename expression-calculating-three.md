#表达式计算(三)
在上篇中我们实现了整数的四则运算的算法，这里我们回到之前提到的 5 5 5 1 的例子，我们看看 eval ( ” 5 * ( 5 – 1/5) ” )的结果是多少？
```
scala> eval ("5*(5-1/5)")
res15: Int = 25
```
结果为25，我们知道这个结果应该是 24，这是因为前面我们的算法都是针对整数的, 1/5 =0 ，当然我们可以把整数改成浮点数，比如，修改 eval 如下：
```
def eval(str:String):Double = str match {
    ...
    case _ => str toDouble
}
```
重新计算 eval (“5*(5-1/5)”)
结果为 24.0，
但是浮点数带来了误差，不是特别理想，我们前面在介绍类和对象时，使用的 Rational 例子，任何有理数都可以表示成分数，因此可以利用这个  Rational 来得到表达式计算的精确结果。
```
class Rational (n:Int, d:Int) {
  require(d!=0)
  private val g =gcd (n.abs,d.abs)
  val numer =n/g
  val denom =d/g
  override def toString = numer + "/" +denom
  def +(that:Rational)  =
    new Rational(
      numer * that.denom + that.numer* denom,
      denom * that.denom
    )

  def -(that:Rational)  =
    new Rational(
      numer * that.denom - that.numer* denom,
      denom * that.denom
    )

  def * (that:Rational) =
    new Rational( numer * that.numer, denom * that.denom)

  def / (that:Rational) =
    new Rational( numer * that.denom, denom * that.numer)

  def this(n:Int) = this(n,1)
  private def gcd(a:Int,b:Int):Int =
    if(b==0) a else gcd(b, a % b)
}
```
利用 Rational 类，我们修改 eval 定义如下：
```
def eval(str:String):Rational = str match {
    case Bracket(part1,expr,part2) => eval(part1 +  eval(expr) + part2)
    case Add(expr1,expr2) => eval(expr1)  +  eval(expr2)
    case Subtract(expr1,expr2) => eval(expr1)  -  eval(expr2)
    case Multiply(expr1,expr2) => eval(expr1)  * eval(expr2)
    case Divide(expr1,expr2) => eval(expr1)  /  eval(expr2)
    case _ => new Rational (str.trim toInt,1)

  }
```
再看看 eval (“5*(5-1/5)”)的计算结果：
```
scala> eval ("5*(5-1/5)")
res16: Rational = 24/1
```
我们得出来表达式的精确结果，为分数表示，比如：
```
scala> eval ("4*6")
res17: Rational = 24/1

scala> eval ("4*6+3*3+5/7")
res18: Rational = 236/7
```
到目前为止我们有了计算四则运算的算法，下面24的算法就比较简单了，穷举法。

注：Scala 中表达式计算的算法还有不少其它方法，比如对表达式的分析可以利用 scala.util.parsing.combinator 提供的 API。