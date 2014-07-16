**高效编码**

1) 应尽量避免让trait去extend一个class。因为这种做法可能会导致间接的继承多个类，从而产生编译错误。同时，还会导致继承体系的复杂度。 
```scala
class StarfleetComponent
trait StarfleetWarpCore extends StarfleetComponent
class Starship extends StarfleetComponent with StarfleetWarpCore
class RomulanStuff

// won't compile
class Warbird extends RomulanStuff with StarfleetWarpCore
```

2) 选择使用Seq时，若需要索引下标功能，优先考虑选择Vector，若需要Mutable的集合，则选择ArrayBuffer；
若要选择Linear集合，优先选择List，若需要Mutable的集合，则选择ListBuffer。

3) 如果需要快速、通用、不变、带顺序的集合，应优先考虑使用Vector。Vector很好地平衡了快速的随机选择和快速的随机更新（函数式）操作。Vector是Scala集合库中最灵活的高效集合。一个原则是：当你对选择集合类型犹疑不定时，就应选择使用Vector。

需要注意的是：当我们创建了一个IndexSeq时，Scala实际上会创建Vector对象：
```scala
scala> val x = IndexedSeq(1,2,3)
x: IndexedSeq[Int] = Vector(1, 2, 3)
```

4) 如果需要选择通用的可变集合，应优先考虑使用ArrayBuffer。尤其面对一个大的集合，且新元素总是要添加到集合末尾时，就可以选择ArrayBuffer。如果使用的可变集合特性更近似于List这样的线性集合，则考虑使用ListBuffer。

5) 如果需要将大量数据添加到集合中，建议选择使用List的prepend操作，将这些数据添加到List头部，最后做一次reverse操作。例如：
```scala
var l = List[Int]()
(1 to max).foreach {
     i => i +: l
}
l.reverse
```

6) 当一个类的某个字段在获取值时需要耗费资源，并且，该字段的值并非一开始就需要使用。则应将该字段声明为lazy。
```scala
lazy val field = computation()
```

7) 在使用Future进行并发处理时，应使用回调的方式，而非阻塞：
```scala
//avoid
val f = Future {
     //executing long time
}

val result = Await.result(f, 5 second)

//suggesion
val f = Future {
     //executing long time
}
f.onComplete {
     case Success(result) => //handle result
     case Failure(e) => e.printStackTrace
}
```

8) 若有多个操作需要并行进行同步操作，可以选择使用par集合。例如：
```scala
val urls = List("http://scala-lang.org",
  "http://agiledon.github.com")

def fromURL(url: String) = scala.io.Source.fromURL(url)
  .getLines().mkString("\n")

val t = System.currentTimeMillis()
urls.par.map(fromURL(_))
println("time: " + (System.currentTimeMillis - t) + "ms") 
```

9) 若有多个操作需要并行进行异步操作，则采用for comprehension对future进行join方式的执行。例如，假设Cloud.runAlgorithm()方法返回一个Futrue[Int]，可以同时执行多个runAlgorithm方法：
```scala
val result1 = Cloud.runAlgorithm(10)
val result2 = Cloud.runAlgorithm(20)
val result3 = Cloud.runAlgorithm(30)

val result = for {
  r1 <- result1
  r2 <- result2
  r3 <- result3
} yield (r1 + r2 + r3)
     
result onSuccess {
  case result => println(s"total = $result")
} 
```