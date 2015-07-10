#从 Java 中调用 Scala 函数
我们对前面定义的计算 24 的代码，稍作修改，可以在 Java 中调用，在通常情况下在 Java 中调用 Scala 函数非常简单，反之从 Scala 中调用 Java 代码也很简单，这是因为Scala代码最终也要编译成 Java class 文件。以后我们将详细介绍 Java 和 Scala 之间的互操作的用法，这里简要介绍下如何在 Java 代码中调用之前我们定义的计算 24 的算法。
在 Scala 二十四点游戏(9): 完整的代码和计算结果我们给出了完整的代码，其中的 Test 由 App 派生，如果我们希望把 Test 定义的一些方法如 cal24,cal24once 作为库函数调用，我们无需让 Test 由 App 派生，另外我们再定义 cal24once  的一个重载版本：
```
object Cal24 {

   ...
   def cal24once(a:Int,b:Int,c:Int,d:Int) {
   cal24once(List(a,b,c,d))
  }

  def hello {
    println("Hello from Scala")
  }

}
```
我们把这段代码存成 Cal24.scala. 下面我们使用 scalac  对它进行编译：
```
scalac Cal24.scala
```
然后我们列出有 scalac  编译后生成的 class 文件：
```
Add.class                                          Cal24.class
Add$.class                                         Cal24$.class
BinaryOp.class                                     Cal24.scala
BinaryOp$class.class                               Divide.class
Bracket$$anonfun$matchBracket$1.class              Divide$.class
Bracket$$anonfun$matchBracket$2.class              Multiply.class
Bracket.class                                      Multiply$.class
Bracket$.class                                     Rational.class
Cal24$$anonfun$cal24$1$$anonfun$apply$1.class      Rational$.class
Cal24$$anonfun$cal24$1.class                       Subtract.class
Cal24$$anonfun$cal24once$1$$anonfun$apply$2.class  Subtract$.class
Cal24$$anonfun$cal24once$1$$anonfun$apply$3.class  
Cal24$$anonfun$cal24once$1.class                   
Cal24$$anonfun$calculate$1.class
```
其中 Cal24  定义了我们所需的库函数，我们可以使用 javap 看看它对应的 java 类定义：
```
root@ubuntu:/sdb/Scala/Cal24# javap Cal24
Compiled from "Cal24.scala"
public final class Cal24 {
  public static void hello();
  public static void cal24once(int, int, int, int);
  public static void cal24once(scala.collection.immutable.List<java.lang.Object>);
  public static void cal24(scala.collection.immutable.List<java.lang.Object>);
  public static scala.Tuple3<java.lang.String, java.lang.String, Rational> calculate(java.lang.String, scala.collection.immutable.List<java.lang.Object>);
  public static Rational eval(java.lang.String);
  public static scala.collection.immutable.List<java.lang.String> templates();
}
```
可以看到 Scala 的 object  (singleton对象）对应到 Java 的 public final class, Cal24 的函数为 static
然后我们定义一个 TestCal24.java
```
public class TestCal24{
    public static void main(String[] args) {
        Cal24.cal24once(5,2,3,4);
    }
}
```
然后我们使用 javac 来编译 TestCal24.java ,此时我们需要指明 scala 库的位置
```
javac -cp $SCALA_HOME/lib/scala-library.jar:. TestCal24.java
```
-cp (class path) 指明Scala类定义的路径

然后运行编译后的TestCal24 代码：
```
root@ubuntu:/sdb/Scala/Cal24# java -cp $SCALA_HOME/lib/scala-library.jar:. TestCal24
List(5, 2, 3, 4):(N-(N-N))*N:(5-(2-3))*4
```
调用成功，这样你可以在Java应用（包括Android应用中使用Scala的函数库）