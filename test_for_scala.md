**测试**

1) 测试类应该与被测试类处于同一包下。如果使用Spec2或ScalaTest的FlatSpec等，则测试类的命名应该为：被测类名 + Spec；若使用JUnit等框架，则测试类的命名为：被测试类名 + Test

2) 测试含有具体实现的trait时，可以让被测试类直接继承Trait。例如：
```scala
trait RecordsGenerator {
     def generateRecords(table: List[List[String]]): List[Record] {
          //...
     }
}

class RecordsGeneratorSpec extends FlatSpec with ShouldMatcher with RecordGenerator {
     val table = List(List("abc", "def"), List("aaa", "bbb"))
     it should "generate records" in {
          val records = generateRecords(table)
          records.size should be(2)
     }
}
```

3) 若要对文件进行测试，可以用字符串假装文件：
```scala
type CsvLine = String
def formatCsv(source: Source): List[CsvLine] = {
     source.getLines(_.replace(", ", "|"))
}
```

formatCsv需要接受一个文件源，例如Source.fromFile("testdata.txt")。但在测试时，可以通过Source.fromString方法来生成formatCsv需要接收的Source对象：
```scala
it should "format csv lines" in {
     val lines = Source.fromString("abc, def, hgi\n1, 2, 3\none, two, three")
     val result = formatCsv(lines)
     result.mkString("\n") should be("abc|def|hgi\n1|2|3\none|two|three")
}
```