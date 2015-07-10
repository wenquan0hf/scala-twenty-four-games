#算法之一
前面我们定义了表达式的算法，通常的 24 点常用的算法，尽管都是穷举，也有几个常用的不同的算法，其中之一有人称为动态规划算法：
把多元运算转化为两元运算，先从四个数中取出两个数进行运算，然后把运算结果和第三个数进行运算，
再把结果与第四个数进行运算。在求表达式的过程中，最难处理的就是对括号的处理，而这种思路很好的避免了对括号的处理。基于这种思路的一种算法： 因为能使用的 4 种运算符 – * / 都是2元运算符，所以本文中只考虑 2 元运算符。2元运算符接收两个参数，输出计算结果，输出的结果参与后续的计算。
由上所述，构造所有可能的表达式的算法如下：
(1) 将 4 个整数放入数组中
(2) 在数组中取两个数字的排列，共有 P(4,2) 种排列。对每一个排列，
(2.1) 对 – * / 每一个运算符，
(2.1.1) 根据此排列的两个数字和运算符，计算结果
(2.1.2) 改表数组：将此排列的两个数字从数组中去除掉，将 2.1.1 计算的结果放入数组中
(2.1.3) 对新的数组，重复步骤 2
(2.1.4) 恢复数组：将此排列的两个数字加入数组中，将 2.1.1 计算的结果从数组中去除掉
可见这是一个递归过程。步骤 2 就是递归函数。当数组中只剩下一个数字的时候，这就是表达式的最终结果，此时递归结束。
在程序中，一定要注意递归的现场保护和恢复，也就是递归调用之前与之后，现场状态应该保持一致。
在上述算法中，递归现场就是指数组，2.1.2 改变数组以进行下一层递归调用，2.1.3 则恢复数组，以确保当前递归调用获得下一个正确的排列。
括号 () 的作用只是改变运算符的优先级，也就是运算符的计算顺序。所以在以上算法中，无需考虑括号。括号只是在输出时需加以考虑。

使用这个算法的一个Scala实现如下：
```
def solve(vs: List[Int],n: Int = 24){
    def isZero(d: Double) = Math.abs(d) < 0.00001
    
    //解析为恰当的中缀表达式
    def toStr(any: Any): String = any match {
        case (v: Double,null,null,null) => v.toInt.toString
        case (_,v1: (Double,Any,Any,Any),v2: (Double,Any,Any,Any),op) => 
               if(op=='-'&&(v2._4=='+'||v2._4=='-'))
                   "%s%c(%s)".format(toStr(v1),op,toStr(v2))
               else if(op=='/'){
                   val s1 = if(v1._4=='+'||v1._4=='-') "("+toStr(v1)+")" else toStr(v1)
                   val s2 = if(v2._4==null) toStr(v2) else "("+toStr(v2)+")"
                   s1 + op + s2
               }
               else if(op=='*'){
                   val s1 = if(v1._4=='+'||v1._4=='-') "("+toStr(v1)+")" else toStr(v1)
                   val s2 = if(v2._4=='+'||v2._4=='-') "("+toStr(v2)+")" else toStr(v2)
                   s1 + op + s2
               }
               else toStr(v1) + op + toStr(v2)
    }
    
    //递归求解
    val buf = collection.mutable.ListBuffer[String]()
    def solve0(xs: List[(Double,Any,Any,Any)]): Unit = xs match {
        case x::Nil => if(isZero(x._1-n) && !buf.contains(toStr(x))){ buf += toStr(x); println(buf.last)}
        case _      => for{ x @ (v1,_,_,_) <- xs;val ys = xs diff List(x)
                              y @ (v2,_,_,_) <- ys;val rs = ys diff List(y)
                         }{   solve0((v1+v2,x,y,'+')::rs)
                              solve0((v1-v2,x,y,'-')::rs)
                              solve0((v1*v2,x,y,'*')::rs)
                              if(!isZero(v2)) solve0((v1/v2,x,y,'/')::rs)
                         }
    }
    solve0(vs map {v => (v.toDouble,null,null,null)})
}
```
测试如下：
```
scala> solve(List(5,5,5,1))
(5-1/5)*5
5*(5-1/5)

scala> solve(List(3,3,8,8))
8/(3-8/3)
```
这个算法的来源于网上，很简短的代码就实现了算 24 的算法，Scala 还是比较强大的:-)
不过我们这里还是采用另外一种方法，来介绍Scala编程的多个方面。
这个算法就是列出 4 个数字加减乘除的各种可能性。我们可以将表达式分成以下几种：首先我们将 4 个数设为 a,b,c,d,，将其排序列出四个数的所有排序序列组合（共有 24 种组合）。再进行符号的排列表达式，其中算术符号有+，—，*，/，（，）。其中有效的表达式有 a*(b-c/b)，a*b-c*d,等等。列出所有有效的表达式。其中我们用枚举类型将符号定义成数字常量。