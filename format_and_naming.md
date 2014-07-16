**格式与命名**

1) 代码格式
用两个空格缩进。避免每行长度超过100列。在两个方法、类、对象定义之间使用一个空白行。

2) 优先考虑使用val，而非var。

3) 当引入多个包时，使用花括号：
```scala
import jxl.write.{WritableCell, Number, Label}
```

当引入的包超过6个时，应使用通配符_：
```scala
import org.scalatest.events._
```

4）若方法暴露为接口，则返回类型应该显式声明。例如：
```scala
    def execute(conn: Connection): Boolean = {
      executeCommand(conn, sqlStatement) match {
        case Right(result) => result
        case Left(_) => false
      }
    }
```

5) 集合的命名规范
xs, ys, as, bs等作为某种Sequence对象的名称；
x, y, z, a, b作为sequence元素的名称。
h作为head的名称，t作为tail的名称。

6）避免对简单的表达式采用花括号；
```scala
//suggestion
def square(x: Int) = x * x

//avoid
def square(x: Int) = {
     x * x
}
```

7) 泛型类型参数的命名虽然没有限制，但建议遵循如下规则：
A            代表一个简单的类型，例如List[A]
B, C, D      用于第2、第3、第4等类型。例如：
                 class List[A] {
                     def map[B](f: A => B): List[B] = ...
                 }
 N           代表数值类型

**注意：**在Java中，通常以K、V代表Map的key与value，但是在Scala中，更倾向于使用A、B代表Map的key与value。