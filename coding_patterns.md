**编码模式**


1) Loan Pattern: 确保打开的资源（如文件、数据库连接）能够在操作完毕后被安全的释放。

Loan Pattern的通用格式如下：
```scala
def using[A](r : Resource)(f : Resource => A) : A =
   try {
        f(r)
   } finally {
        r.dispose()
   }
```

这个格式针对Resource类型进行操作。还有一种做法是：只要实现了close方法，都可以运用Loan Pattern：
```scala
def using[A <: def close():Unit, B][resource: A](f: A => B): B = 
     try {
          f(resource)
     } finally {
          resource.close()
     }
```

以FileSource为例：
```scala
using(io.Source.fromFile("example.txt")) { 
    source => {
        for (line <- source.getLines) {
            println(line)
        }
    } 
}
```

2) Cake Pattern: 利用self type实现依赖注入

例如，对于DbAccessor而言，需要提供不同的DbConnectionFactory来创建连接，从而访问不同的Data Source。
```scala
trait DbConnectionFactory {
     def createDbConnection: Connection
}

trait SybaseDbConnectionFactory extends DbConnectionFactory…
trait MySQLDbConnectionFactory extends DbConnectionFactory…
```

运用Cake Pattern，DbAccessor的定义应该为：
```scala
trait DbAccessor {
     this: DbConnectionFactory => 

     //…
}
```

由于DbAccessor使用了self type，因此可以在DbAccessor中调用DbConnectionFactory的方法createDbConnection()。客户端在创建DbAccessor时，可以根据需要选择混入的DbConnectionFactory：
```scala
val sybaseDbAccessor = new DbAccessor with SybaseDbConnectionFactory
```

当然，也可以定义object：
```scala
object SybaseDbAccessor extends DbAccessor with SybaseDbConnectionFactory
object MySQLDbAccessor extends DbAccessor with MySQLDbConnectionFactory
```