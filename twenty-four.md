#计算 24 的算法
有了前面的准备工作，我们现在可以给出二十四游戏的算法，首先
我们合并表示式模板和输入的四个数字，计算出结果：
```
def calculate(template:String,numbers:List[Int])={
    val values=template.split('N')
    var expression=""
    for(i <- 0 to 3)  expression=expression+values(i) + numbers(i)
    if (values.length==5) expression=expression+values(4)
    (expression,template,eval(expression))
  }
```
做些简单的测试如下：
```
scala> calculate("N/N*N+N",List(6,9,9,10))
res0: (String, String, Rational) = (6/9*9+10,N/N*N+N,16\1)

scala> calculate("N/N*N+N",List(9,6,10,9))
res1: (String, String, Rational) = (9/6*10+9,N/N*N+N,24\1)

scala> calculate("(N-N/N)*N",List(5,1,5,5))
res2: (String, String, Rational) = ((5-1/5)*5,(N-N/N)*N,24\1)
```
我们让函数 calculate 返回三个值，合成的表达式，使用的模板，计算的结果（分数形式），我们使用一个三元组作为返回结果，这里也可以看到 Scala 函数返回，无需使用 return，函数体的最后一条语句的值作为返回结果。

说到这里，在之前的 Rational 的实现和 eval 函数的实现有一个小错误（表达式出现中的歧义），之前 Rational 的字符表现形式为
```
override def toString = numer + "/" +denom
```
使用到的“/”和 我们使用的四则运算的除号一样,这样对于这样的表达式8/1/3,就有两种解释 (8/1)/3 其中8/1 为计算的中间结果（Rational 对象，“/”为 Rational 字符串形式中的/），计算结果为８／３.
另外一种解释为8/（1/3） 其中1/3为输入时的除号。
为避免这种歧义，我们将 Rational 的“/”改为”\”，修改之前的相关定义：
```
class Rational (n:Int, d:Int) {
  require(d!=0)
  private val g =gcd (n.abs,d.abs)
  val numer =n/g
  val denom =d/g
  override def toString = numer + "\\" +denom
  ...
 }
 
object Rational  extends {val op="\\"} with BinaryOp
 
def eval(str:String):Rational = {

    str match {
      case Bracket(part1, expr, part2) => eval(part1 + eval(expr) + part2)
      case Add(expr1, expr2) => eval(expr1) + eval(expr2)
      case Subtract(expr1, expr2) => eval(expr1) - eval(expr2)
      case Multiply(expr1, expr2) => eval(expr1) * eval(expr2)
      case Divide(expr1, expr2) =>  eval(expr1) / eval(expr2)
      case "" => new Rational(0, 1)
      case Rational(expr1, expr2) =>   new Rational(expr1.trim toInt, expr2.trim toInt)
      case _ => new Rational(str.trim toInt, 1)

    }
} 
```
我们有了 calculate 函数之后，就可以根据数字的全排列，和可能的表达式模板，设计出 24 游戏的穷举算法：
```
def cal24(input:List[Int])={
    var found = false
    for (template <- templates; list <- input.permutations ) {
      try {
        val (expression, tp, result) = calculate(template, list)
        if (result.numer == 24 && result.denom == 1) {
          println(input + ":" + tp + ":" + expression)
          found = true
        }
      } catch {
        case e:Throwable=>
      }
    }
    if (!found) {
      println(input+":"+"no result")
    }
}
```
这个算法列出所有可能的算法，比如：
```
scala> cal24(List(5,5,5,1))
List(5, 5, 5, 1):(N-N/N)*N:(5-1/5)*5

scala> cal24(List(3,3,8,8))
List(3, 3, 8, 8):N/(N-N/N):8/(3-8/3)

scala> cal24(List(5,6,7,8))
List(5, 6, 7, 8):(N-(N-N))*N:(5-(8-7))*6
List(5, 6, 7, 8):(N-(N-N))*N:(7-(8-5))*6
List(5, 6, 7, 8):N*N/(N-N):6*8/(7-5)
List(5, 6, 7, 8):N*N/(N-N):8*6/(7-5)
List(5, 6, 7, 8):(N+N)*(N-N):(5+7)*(8-6)
List(5, 6, 7, 8):(N+N)*(N-N):(7+5)*(8-6)
List(5, 6, 7, 8):(N-N-N)*N:(5-8-7)*6
List(5, 6, 7, 8):(N-N-N)*N:(7-8-5)*6
List(5, 6, 7, 8):(N+N-N)*N:(5+7-8)*6
List(5, 6, 7, 8):(N+N-N)*N:(7+5-8)*6
```
对于5，6，7，8 由于加法和乘法的交互律，某些算法是等价的，我们可以根据使用的模板是否相同去掉这些等价的算法。
如果只需要计算一种算法，可以为 for 表达式加上条件：
```
def cal24once(input:List[Int])={
    var found = false
    for (template <- templates; list <- input.permutations if(!found)) {
      try {
        val (expression, tp, result) = calculate(template, list)
        if (result.numer == 24 && result.denom == 1) {
          println(input + ":" + tp + ":" + expression)
          found = true
        }
      } catch {
        case e:Throwable=>
      }
    }
    if (!found) {
      println(input+":"+"no result")
    }
  }
```
测试如下：
```
scala> cal24once(List(5,6,7,8))
List(5, 6, 7, 8):(N-(N-N))*N:(5-(8-7))*6

scala> cal24once(List(1,2,3,4))
List(1, 2, 3, 4):N*N*N*N:1*2*3*4

scala> cal24once(List(1,1,1,1))
List(1, 1, 1, 1):no result
```