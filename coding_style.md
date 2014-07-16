**编码风格**

1) 尽可能直接在函数定义的地方使用模式匹配。例如，在下面的写法中，match应该被折叠起来(collapse):
```scala
list map { item =>   
     item match {     
          case Some(x) => x     
          case None => default   
     } 
}
```

用下面的写法替代：
```scala
list map {
   case Some(x) => x
   case None => default 
}
```

它很清晰的表达了 list中的元素都被映射，间接的方式让人不容易明白。此时，传入map的函数实则为partial function。 

2）避免使用null，而应该使用Option的None。
```scala
import java.io._

object CopyBytes extends App {
     var in = None: Option[FileInputStream]
     var out = None: Option[FileOutputStream]
     try {
          in = Some(new FileInputStream("/tmp/Test.class"))
          out = Some(new FileOutputStream("/tmp/Test.class.copy"))
          var c = 0
          while ({c = in.get.read; c != −1}) {
             out.get.write(c)
    }
     } catch {
          case e: IOException => e.printStackTrace
     } finally {
          println("entered finally ...")
          if (in.isDefined) in.get.close
          if (out.isDefined) out.get.close
     }
} 
```

3）若在Class中需要定义常量，应将其定义为val，并将其放在该类的伴生对象中：
```scala
class Pizza (var crustSize: Int, var crustType: String) {
     def this(crustSize: Int) {
          this(crustSize, Pizza.DEFAULT_CRUST_TYPE)
     }
   
     def this(crustType: String) {
          this(Pizza.DEFAULT_CRUST_SIZE, crustType)
     }
   
     def this() {
          this(Pizza.DEFAULT_CRUST_SIZE, Pizza.DEFAULT_CRUST_TYPE)
     }
     override def toString = s"A $crustSize inch pizza with a $crustType crust"
}

object Pizza {
     val DEFAULT_CRUST_SIZE = 12
     val DEFAULT_CRUST_TYPE = "THIN"
}
```

4）合理为构造函数或方法提供默认值。例如：
```scala
class Socket (val timeout: Int = 10000)
```

5）如果需要返回多个值时，应返回tuple。 
```scala
def getStockInfo = {
     //
     ("NFLX", 100.00, 101.00)
}
```

6) 作为访问器的方法，如果没有副作用，在声明时建议定义为没有括号。

例如，Scala集合库提供的scala.collection.immutable.Queue中，dequeue方法没有副作用，声明时就没有括号：
```scala
import scala.collection.immutable.Queue

val q = Queue(1, 2, 3, 4)
val value = q.dequeue
```

7) 将包的公有代码（常量、枚举、类型定义、隐式转换等）放到package object中。
```scala
package com.agiledon.myapp

package object model {
     // field
     val MAGIC_NUM = 42 182 | Chapter 6: Objects
￼
     // method
     def echo(a: Any) { println(a) }
   
    // enumeration
     object Margin extends Enumeration {
          type Margin = Value
          val TOP, BOTTOM, LEFT, RIGHT = Value
     }
   
    // type definition
     type MutableMap[K, V] = scala.collection.mutable.Map[K, V]
     val MutableMap = scala.collection.mutable.Map
}
```

8) 建议将package object放到与包对象命名空间一致的目录下，并命名为package.scala。以model为例，package.scala文件应放在：
+-- com
     +-- agiledon
          +-- myapp
               +-- model
                    +-- package.scala

9) 若有多个样例类属于同一类型，应共同继承自一个sealed trait。
```scala
sealed trait Message
case class GetCustomers extends Message
case class GetOrders extends Message
```

**注**：这里的sealed，表示trait的所有实现都必须声明在定义trait的文件中。


10) 考虑使用renaming clause来简化代码。例如，替换被频繁使用的长名称方法：
```scala
import System.out.{println => p}

p("hallo scala")
p("input") 
```

11) 在遍历Map对象或者Tuple的List时，且需要访问map的key和value值时，优先考虑采用Partial Function，而非使用_1和_2的形式。例如：
```scala
val dollar = Map("China" -> "CNY", "US" -> "DOL")

//perfer
dollar.foreach {
     case (country, currency) => println(s"$country -> $currency")
}

//avoid
dollar.foreach ( x => println(s"$x._1 -> $x._2") )
```

或者，考虑使用for comprehension：
```scala
for ((country, currency) <- dollar) println(s"$country -> $currency")
```

12) 遍历集合对象时，如果需要获得并操作集合对象的下标，不要使用如下方式：
```scala
val l = List("zero", "one", "two", "three")

for (i <- 0 until l.length) yield (i, l(i))
```

而应该使用zipWithIndex方法：
```scala
for ((number, index) <- l.zipWithIndex) yield (index, number)
```

或者：
```scala
l.zipWithIndex.map(x => (x._2, x._1))
```

当然，如果需要将索引值放在Tuple的第二个元素，就更方便了。直接使用zipWithIndex即可。

zipWithIndex的索引初始值为0，如果想指定索引的初始值，可以使用zip：
```scala
l.zip(Stream from 1)
```

13) 应尽量定义小粒度的trait，然后再以混入的方式继承多个trait。例如ScalaTest中的FlatSpec：
```scala
class FlatSpec extends FlatSpecLike ...

trait FlatSpecLike extends Suite with ShouldVerb with MustVerb with CanVerb with Informing …
```

小粒度的trait既有利于重用，同时还有利于对业务逻辑进行单元测试，尤其是当一部分逻辑需要依赖外部环境时，可以运用“关注点分离”的原则，将不依赖于外部环境的逻辑分离到单独的trait中。

14) 优先使用不可变集合。如果确定要使用可变集合，应明确的引用可变集合的命名空间。不要用使用import scala.collection.mutable._；然后引用 Set，应该用下面的方式替代：
```scala
import scala.collections.mutable
val set = mutable.Set()
```

这样更明确在使用一个可变集合。

15) 在自己定义的方法和构造函数里，应适当的接受最宽泛的集合类型。通常可以归结为一个: Iterable, Seq, Set, 或 Map。如果你的方法需要一个 sequence，使用 Seq[T]，而不是List[T]。这样可以分离集合与它的实现，从而达成更好的可扩展性。

16) 应谨慎使用流水线转换的形式。当流水线转换的逻辑比较复杂时，应充分考虑代码的可读性，准确地表达开发者的意图，而不过分追求函数式编程的流水线转换风格。例如，我们想要从一组投票结果(语言，票数)中统计不同程序语言的票数并按照得票的顺序显示：
```scala
val votes = Seq(("scala", 1), ("java", 4), ("scala", 10), ("scala", 1), ("python", 10))
val orderedVotes = votes
   .groupBy(_._1)
   .map { case (which, counts) =>
     (which, counts.foldLeft(0)(_ + _._2))
   }.toSeq
   .sortBy(_._2)
   .reverse
```

上面的代码简洁并且正确，但几乎每个读者都不好理解作者的原本意图。一个策略是声明中间结果和参数：
```scala
val votesByLang = votes groupBy { case (lang, _) => lang }
val sumByLang = votesByLang map { 
     case (lang, counts) =>
          val countsOnly = counts map { case (_, count) => count }
          (lang, countsOnly.sum)
}
val orderedVotes = sumByLang.toSeq
   .sortBy { case (_, count) => count }
   .reverse
```

代码也同样简洁，但更清晰的表达了转换的发生(通过命名中间值)，和正在操作的数据的结构(通过命名参数)。

17) 对于Options对象，如果getOrElse能够表达业务逻辑，就应避免对其使用模式匹配。许多集合的操作都提供了返回Options的方法。例如headOption等。
```scala
val x = list.headOption getOrElse 0
```

这要比模式匹配更清楚：
```scala
val x = list match 
     case head::_ => head
     case Nil: => 0
```

18) 当需要对两个或两个以上的集合进行操作时，应优先考虑使用for表达式，而非map，flatMap等操作。此时，for comprehension会更简洁易读。例如，获取两个字符的所有排列，相同的字符不能出现两次。使用flatMap的代码为：
```scala
 val chars = 'a' to 'z'
 val perms = chars flatMap { a =>
   chars flatMap { b =>
     if (a != b) Seq("%c%c".format(a, b))
     else Seq()
   }
 }
```

使用for comprehension会更易懂：
```scala
 val perms = for {
   a <- chars
   b <- chars
   if a != b
 } yield "%c%c".format(a, b)
``` 