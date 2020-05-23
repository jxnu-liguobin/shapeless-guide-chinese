# 第一章 简介

此书是一本关于如何使用[shapeless](https://github.com/milessabin/shapeless)的指南，shapeless是Scala语言的一个泛型编程库。由于shapeless包含的内容过多，所以此书只是专注于一些非常有意思的使用案例，而不是包罗万象的涵盖shapeless的一切，希望借此描绘出此工具和编程模式的全景。

在本章开始处，先来介绍一下什么是泛型编程以及是什么原因使得Scala开发者对于shapeless库如此兴奋。

## 1.1 什么是泛型编程？ <a id="11-&#x4EC0;&#x4E48;&#x662F;&#x6CDB;&#x578B;&#x7F16;&#x7A0B;&#xFF1F;"></a>

类型（Type）很有用，因为它是明确的——它向我们展示不同的代码片段如何组合到一起，帮助我们消除BUG并在编写代码的时候指导我们解决问题。

然而有时类型又太具体导致大量重复编码，所以有些情形下我们想利用不同类型之间的相似性来去除重复编码工作。例如，考虑以下两个类型定义：

```text
case class Employee(name: String, number: Int, manager: Boolean)

case class IceCream(name: String, numCherries: Int, inCone: Boolean)
```

这两个类代表不同的数据类型，但是它们又非常相似——都包含三个字段且类型相同。假设我们要实现一个对它们都通用的操作，例如序列化到CSV文件。尽管这两类数据相似，但是我们不得不写两个不同的方法。分别如下：

```text
def employeeCsv(e: Employee): List[String] = 
    List(e.name, e.number.toString, e.manager.toString)

def iceCreamCsv(c: IceCream): List[String] = 
    List(c.name, c.numCherries.toString, c.inCone.toString)
```

泛型编程能够克服像上面这样由于不同数据类型带来的重复操作。shapeless很容易实现将具体的类型泛型化，这样就可以使用同一段代码来操作不同的类型。

比如，我们能用如下代码将Employee和IceCream实例转换成同一类型。如果不理解以下代码也不用担心，本书会在接下来的章节中详细介绍它们。

```text
import shapeless._

val genericEmployee = Generic[Employee].to(Employee("Dave", 123, false)) 
// genericEmployee: String :: Int :: Boolean :: shapeless.HNil 
// = Dave :: 123 :: false :: HNil

val genericIceCream = Generic[IceCream].to(IceCream("Sundae", 1, false)) 
// genericIceCream: String :: Int :: Boolean :: shapeless.HNil 
//= Sundae :: 1 :: false :: HNil
```

现在两个值变成了相同类型，都是异构的列表（简称HList），它包含一个字符串（String）、一个整型（Int）和一个布尔（Boolean）对象。接下来我们将研究HList类型和它在shapeless中所扮演的重要角色。目前为止关键在于用同一个函数来序列化各自的值，而这些值是上面两种类型被泛型化后的值。代码如下：

```text
def genericCsv(gen: String :: Int :: Boolean :: HNil): List[String] = 
    List(gen(0), gen(1).toString, gen(2).toString)

genericCsv(genericEmployee) 
// res2: List[String] = List(Dave, 123, false)

genericCsv(genericIceCream) 
// res3: List[String] = List(Sundae, 1, false)
```

这个简单的例子展示了泛型编程的精髓。通过重新探究这些问题，我们不但用泛型代码块解决了问题，而且写出了适用于多种类型的精简代码。使用shapeless进行泛型编程可以消除大量的冗余代码，使Scala应用程序更容易阅读、编写和维护。

听上去是不是很有诱惑？想想这些，让我们一起入坑吧！

## 1.2 关于此书 <a id="12-&#x5173;&#x4E8E;&#x6B64;&#x4E66;"></a>

该书共分为两部分。

第一部分介绍类型类（type class）派生，它使我们仅用一些泛型规则来为任何代数数据类型（algebraic data type，简称ADT）创建类型类实例。第一部分包含四个章节。

* 第二章介绍泛型表示（generic representation），以及shapeless中名为Generic的类型类，Generic能够为任何一个样例类（case class）或密封特质（sealed trait）创建一个泛型编码器，将其转化为泛型。
* 第三章介绍用Generic派生自定义类型类实例，并创建一个将Scala中的数据编码为CSV格式的类型类，但该例子所用的技术可以扩展到许多情形。此外还介绍了shapeless中的Lazy类型，可以处理像列表（list）以及树（tree）等类型的递归数据。
* 第四章介绍前几章涉及的理论和编程模式，特别是针对依赖类型（dependent type）、类型依赖函数（dependently typed function）以及类型级别编程（type level programming），这些能使我们进入更高级的shapeless应用。
* 第五章介绍LabelledGeneric，它是Generic的一个变体，它将字段名称和类型名称转换为其泛型表示的一部分。还介绍了一些理论知识：字面类型（ literal）、单例类型（singleton）、幻象类型（phantom）和标记类型（type tagging）。我们会创建一个在输出中保持字段和类型名称不变的JSON编码器，以此来演示LabelledGeneric。

第二部分介绍在shapeless.ops包中提供的“ops类型类”，它来源于一个处理泛型表示工具的扩展库。在接下来的三章仅为大家介绍入门理论，而不是介绍每一个操作（op）的细节。

* 第六章从宏观上介绍ops类型类，并给出了一个例子，通过将几个简单操作串联，从而组成一个强大的”样例类迁移（case class migration）”工具。
* 第七章介绍多态函数（ploymorphic functions）亦称Poly，并展示在ops类型类中如何使用多态函数对“泛型表示”进行映射（mapping）、平面映射（flat mapping）和折叠（fold）操作。
* 第八章介绍shapeless的Nat类型，它在类型级别表示自然数。介绍几个相关的ops类型类并用Nat建立我们自己的ScalaCheck（一个Scala测试框架）中的Arbitrary（随机数生成）类。

## 1.3 源码和例子 <a id="13-&#x6E90;&#x7801;&#x548C;&#x4F8B;&#x5B50;"></a>

此书是开源的，你可以在[Github](https://github.com/underscoreio/shapeless-guide)中找到其markdown格式。本书会持续更新，所以请检查上述Github仓库以获取最新版本。

书中的大多数例子已经实现，你可以在[shapeless-guide-code](https://github.com/underscoreio/shapeless-guide-code)中找到它们。细节请参考README文件。

我们推荐shapeless2.3.2版以及Typelevel Scala2.11.8+版或者Lightbend Scala2.11.9+/2.12.1+版。

本书中大多数示例都使用了2.12.1，这个版本的Scala引入了中缀类型表示，类型在控制台的输出将会更加清晰，如下：

```text
val repr = "Hello" :: 123 :: true :: HNil
// repr: String :: Int :: Boolean :: shapeless.HNil = Hello :: 123 ::
// true :: HNil
```

而如果你使用的是旧版本的Scala，你将会看到前缀类型表示，就像下面这样：

```text
val repr = "Hello" :: 123 :: true :: HNil
// repr: shapeless.::[String,shapeless.::[Int,shapeless.::[Boolean,
// shapeless.HNil]]] = "Hello" :: 123 :: true :: HNil
```

不要恐慌，除了结果的打印形式不同（中缀和前缀的语法）之外，这些类型是相同的。如果您发现前缀类型表示难以阅读，我们建议将Scala升级到较新的版本，只需要添加以下内容在build.sbt中：

```text
 scalaOrganization := "org.typelevel" 
 //scalaOrganization只支持0.13.13+，可在project/build.properties修改
 scalaVersion := "2.12.1"
```



## 1.4 致谢 <a id="14-&#x81F4;&#x8C22;"></a>

感谢Miles Sabin、Richard Dallaway、Noel Welsh、Travis Brown和我们的[shapeless-guide库中的贡献者](https://github.com/underscoreio/shapeless-guide/graphs/contributors)，他们为这本指导提供了不可估量的贡献。

特别感谢Sam Halliday，他的优秀的[Shapeless for Mortals](https://github.com/fommil/shapeless-for-mortals)库为我们提供了最初的灵感和骨架。

最后，感谢Rob Norris和他的[Tut](https://github.com/tpolecat/tut)库的跟随者，他们对我们的例子做了精心的验证，确保其编译无误。

