# Scala 二十四点游戏(2):表达式计算(二)
在上篇[Scala二十四点游戏(1):表达式计算(一)](expression-calculating-one.md)我们使用Scala实现了四则运算，但还不支持带括号的情况，本篇我们接着看看如处理带括号的情况，
比如表达式 1+2+(3*5)+3+3*(3+(3+5))

括号的情况稍微有些复杂，一层括号比较简单，对于嵌套括号的情况，需要匹配同一层次的括号，好在我们只需要匹配最外面一层括号，其它的可以通过递归函数的方法依次匹配。这里我们定义一个方法，通过栈结构来匹配最外一层括号：
```
import scala.collection.mutable.Stack  

def matchBracket(str:String):Option[(Int,Int)] ={
    val left = str.indexOf('(')
    if(left >=0) {
      val stack = Stack[Char]()
      val remaining = str substring (left+1)
      var index=0
      var right=0
      for(c <-remaining if right==0){
        index=index + 1
        c match{
          case '(' => stack push c
          case ')'  => if (stack isEmpty)  right= left+index else stack pop
          case _ =>
        }

      }

      Some(left,right)
    }else  None
  }
```

这个方法匹配最外面一层括号，并给出他们在字符中的位置，我们做个简单的测试。
```
scala> val str="1+2+(3*5)+3+3*(3+(3+5))" 
str: String = 1+2+(3*5)+3+3*(3+(3+5))

scala> val Some((left,right))=matchBracket(str)
left: Int = 4
right: Int = 8

scala> str.charAt(left)
res0: Char = (
```
这个函数成功找到匹配的括号。

对于每个包含括号的表达式，可以有如下形式  

part1 ( expr ) part2

因此我们可以实现如下的Bracket 对象来匹配括号表达式
```
object Bracket{

  def matchBracket(str:String):Option[(Int,Int)] ={
    val left = str.indexOf('(')
    if(left >=0) {
      val stack = Stack[Char]()
      val remaining = str substring (left+1)
      var index=0
      var right=0
      for(c <-remaining if right==0){
        index=index + 1
        c match{
          case '(' => stack push c
          case ')'  => if (stack isEmpty)  right= left+index else stack pop
          case _ =>
        }

      }

      Some(left,right)
    }else  None
  }

  def apply(part1:String,expr:String,part2:String) =part1+ "(" + expr + ")"+ part2
  def unapply(str:String) :Option[(String,String,String)] ={
     Bracket.matchBracket(str) match{
      case Some((left:Int,right:Int)) =>{
        val part1 = if (left == 0) "" else str substring(0, left )
        val expr = str substring(left + 1, right)
        val part2 = if (right == (str length)-1) "" else str substring (right+1)
        Some(part1, expr, part2)
      }
      case _ => None
    }
  }
}
```
修改之前的eval 函数，首先匹配括号表达式：
```
def eval(str:String):Int = str match {
    case Bracket(part1,expr,part2) => eval(part1 +  eval(expr) + part2)
    case Add(expr1,expr2) => eval(expr1)  +  eval(expr2)
    case Subtract(expr1,expr2) => eval(expr1)  -  eval(expr2)
    case Multiply(expr1,expr2) => eval(expr1)  * eval(expr2)
    case Divide(expr1,expr2) => eval(expr1)  /  eval(expr2)
    case _ => str toInt

 }
```
做些简单的测试：
```
scala> eval ("1+(3+(4+2)+3+(3+5)+3)+5")
res1: Int = 29

scala> eval ("1+2+(3*5)+3+3*(3+(3+5))")
res2: Int = 54
```
这样整数的四则运算的算法基本实现了，当然还不是很完善，比如负数，错误处理等，不过这些对我们解决24问题不是很重要，我们暂时忽略这些问题。