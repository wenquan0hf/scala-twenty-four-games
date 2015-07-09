#实现全排列
穷举法计算二十四的算法的重要一个步骤，是把数字进行全排列，比如对于一个三个数的列表 List(1,2,3)，其全排列如下：
```
List(1, 2, 3)
List(1, 3, 2)
List(2, 1, 3)
List(2, 3, 1)
List(3, 1, 2)
List(3, 2, 1)
```
解决这种问题的一个策略是采用“分而治之”的方法，首先把问题分解成小的问题，比如N个数的全排列可以分解成 N-1的全排列再加 1 个数的排列，然后对每个小的问题给出解决方案。
由此前面可以写出如下的一个递归算法：
```
def permutations(l:List[Int]):List[List[Int]] = {
    l match {
      case Nil => List(List())
      case (head::tail) =>
        for(p0 <- permutations(tail);i<-0 to (p0 length);(xs,ys)=p0 splitAt i)  yield xs:::List(head):::ys
    }
  }
```
空列表的全排列为空，N 个数的全排列为 N-1 个数的全排列和 1 个数的全排列，对于每个 N-1 的排列，依次插入剩下的一个数，就构成了一个新的全排列。
测试如下：
```
scala> permutations(List(1,2,3)).mkString("\n")
res3: String =
List(1, 2, 3)
List(2, 1, 3)
List(2, 3, 1)
List(1, 3, 2)
List(3, 1, 2)
List(3, 2, 1)
```
再看看 1,1,2 的情况：
```
scala> permutations(List(1,1,3)).mkString("\n")
res4: String =
List(1, 1, 3)
List(1, 1, 3)
List(1, 3, 1)
List(1, 3, 1)
List(3, 1, 1)
List(3, 1, 1)
```
有重复的排列，我们可以直接借助于 List 的 distinct 方法过滤掉重复的值。
```
scala> permutations(List(1,1,3)).distinct.mkString("\n")
res5: String =
List(1, 1, 3)
List(1, 3, 1)
List(3, 1, 1)
```
这样全排列的算法就成了，其实List自身已经提供了 permutations 方法，不需要自行实现:-)
```
scala> List(1,2,3,4).permutations.mkString("\n")
res6: String =
List(1, 2, 3, 4)
List(1, 2, 4, 3)
List(1, 3, 2, 4)
List(1, 3, 4, 2)
List(1, 4, 2, 3)
List(1, 4, 3, 2)
List(2, 1, 3, 4)
List(2, 1, 4, 3)
List(2, 3, 1, 4)
List(2, 3, 4, 1)
List(2, 4, 1, 3)
List(2, 4, 3, 1)
List(3, 1, 2, 4)
List(3, 1, 4, 2)
List(3, 2, 1, 4)
List(3, 2, 4, 1)
List(3, 4, 1, 2)                                                              
List(3, 4, 2, 1)
List(4, 1, 2, 3)
List(4, 1, 3, 2)
List(4, 2, 1, 3)
List(4, 2, 3, 1)
List(4, 3, 1, 2)
List(4, 3, 2, 1)
```