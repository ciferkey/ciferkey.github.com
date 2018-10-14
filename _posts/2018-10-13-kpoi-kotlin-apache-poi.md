---
layout: single
classes: wide
title: "KPOI A Kotlin DSL for working with Apache POI"
description: "Leveraging Kotlin language features to build a DSL for writing spreadsheets with Apache POI"
categories:
  - Projects
---

<style type="text/css">
  .gist-meta {
    display: none;
  }
</style>


Sometimes in my line of work I have the *please* of trying to programmatically generate spreadsheets or get to work on someone else's code which generates spreadsheets. In Java land the go to library for doing this is [Apache POI](https://poi.apache.org/). The API for generating spreadsheets is very imperative and side effect based. I could not help be curious about what it might look like if the API was re-imagined with a [Domain Specific Language](https://martinfowler.com/books/dsl.html) written in [Kotlin](https://kotlinlang.org/). So I decided like most normal humans would to spend a couple weekends writing code that helps generate spreadsheets. 

### What is a DSL?

For those that not familiar with the concept a Domain Specific Language is a "computer language specialized to a particular application domain". For example SQL (when you exclude the many extensions that have been made to it over the years) is a DSL specifically for querying databases. Contrast this to general purpose programming languages like Java, Python, Kotlin, and Haskell which are designed to cover a wide variety of problems. Additionally some languages provide the ability to extend the syntax of the language to create DSLs *within* the language itself. This is fairly common in the ML (thats "Meta Language" not "Machine Learning") family of languages (Standard ML, OCaml, F Sharp) and languages that take inspiration from ML to varying degrees (Haskell, Scala, Rust) as well as modern languages not directly related to ML (Kotlin, Swift, Groovy). An example of a DSL you might have within a language is a [declarative way of generating HTML](https://kotlinlang.org/docs/reference/type-safe-builders.html).

This post is not a tutorial on writing a DSL itself as there are plenty of great articles on that already:

* [Writing DSLs in Kotlin (part 1)](https://proandroiddev.com/writing-dsls-in-kotlin-part-1-7f5d2193f277)
* [Kotlin DSLs: The Basics](https://dzone.com/articles/kotlin-dsl-basics)
* [Kotlin DSL: From Theory to Practice](https://dzone.com/articles/kotlin-dsl-from-theory-to-practice)
* [Create a DSL in Kotlin](https://kotlinexpertise.com/create-dsl-with-kotlin/)

Instead this post covers:

1. Highlighting the language features in Kotlin which make wrapping/calling existing Java code a pleasant experience.
2. Giving a medium scale example creating a Kotlin DSL.

The end result is that we can take Apache POI code in Java which looks like this:
<script src="https://gist.github.com/ciferkey/add42c48196376ad3058c6719a7e7ee0.js"></script>

To the Kotlin DSL which looks like:
<script src="https://gist.github.com/ciferkey/a0b1421b853b6c9ccaa4da308fe3fe58.js"></script>

### Designing the Apache POI DSL:

The design goals for the DSL were born from real world usage of POI:

* POI provides multiple spreadsheet [implementations](https://poi.apache.org/components/spreadsheet/) with different version compatibilities and performance characteristics. I wanted to build a single DSL for all for all the implementations that allows the underlying sheet type to be changed as needed.
* providing defaults for common operations but allowing all default to be easily overridden. For example you will not need to supply a row number when writing continuous rows but you can set a specific row number if you wish.

Apache POI is build of a hierarchy of concepts:

* You start off by creating a *Workbook*
* You can create *Sheets* from a Workbook (these are the tabs you would see at the bottom in Excel)
* You can create *Rows* from a Sheet
* You can create *Cells* from a Row

I want the DSL to allow you to define the workbook *declaratively* by creating a nested structure that mirrors the relation between the classes rather than *imperatively* by creating these objects and then trying to relate them to each other. 

### Implementing the DSL:
So how do we go about implementing a DSL to wrap a existing Java library? Well Kotlin provides some nice language features that we can build off of.

__Higher Order Functions, Lambdas as Blocks, Lambda as last parameter__

Other important fundamental language features are [lambdas and higher order functions](http://kotlinlang.org/docs/reference/lambdas.html#higher-order-functions). These concepts will be familiar for those who have worked in Java. The basic idea is that functions are "first class" can be passed as parameters to *other* functions.

In our case lets say we want a method which will create a Workbook for us, apply a supplied method to it which modifies the notebook and then returns the new notebook. In Java this would look like:
<script src="https://gist.github.com/ciferkey/d6cfb19f7c86d57021716fd50f1b9ee6.js"></script>

In Kotlin this could be done as:
<script src="https://gist.github.com/ciferkey/3ba7ce78ac9a77d7fcbb2e505c1cbd3a.js"></script>

So what on earth just happen in that last example method call? Basically if you have a method that only takes a single parameter which is a lambda then you can drop a lot of the syntax you would normally have to type out in Java. This is key to making a DSL! If you look closely at the final example you can see that it looks just like we are using a *statement* in the language such as the "if" statement. __This is why an internal DSL is often described as extending the language__. We are able to create new syntax which feels like its a part of the language itself!

__Receiver Types__

A second piece of syntax we can clean up with the in the DSL is the explicit reference to the Workbook. Previously we skipped naming the parameter we passed into the lambda and instead opted to use the default parameter name of "it". We can further simplify things by making use of [receiver types](https://kotlinlang.org/docs/reference/lambdas.html#function-types) so you don't have to provide the object type. The basic idea of a receiver type lets you make the explicit workbook parameter implicit so you can the methods on it directly without having to reference the workbook parameter. This means we can update the previous example as follows:
<script src="https://gist.github.com/ciferkey/add920dc3bfc1297f20c9ee9a11182a5.js"></script>

Now we can call setHidden and setSelectedTab directly without referring to "it".

__Property Access Syntax__

Why do we use getters and setters in Java? One main line of reasoning is that it enables encapsulation: if the underlying implementation of a class changes and we are directly referring to fields on a class then we must update all the places were we access fields that have changed. If we use accessor methods then we can change the underling implementation, update the methods and the external code can stay the same. Additionally getters and setters allow us to do things like check if the underlying value is initialized or for a setter we can do some form of type conversion.

However these examples typically are the *exception* rather than the *norm*. Unless you are writing a library that you are exposing to others then you have the ability to refactor code that uses as class as you refactor the class itself. In fact the change can be easy and comprehensive when done with a modern IDE. It would be preferable then to instead allow access to a field directly but to provide a way to override it if you encounter the rare circumstance when you need to. This is what language like C# and Kotlin allow you to do.

In Kotlin you directly refer to a field on a class. Then if you need to you can define a getter or setter. However instead of explicitly referring to the accessor method, the method will instead be implicitly used when accessing the field:
<script src="https://gist.github.com/ciferkey/9ee0e704c39fd587adc775d2423a6349.js"></script>

Additionally since Kotlin aims for a pleasant Java interop experience Java classes which have a getter can accessed using Kotlin conventions. This means if you have a Plain Old Java Object with getters and setters you can use it in Kotlin using the Kotlin property access syntax. This means we can update our previous example to:
<script src="https://gist.github.com/ciferkey/94c1018cbd09006f9efc152158e59a9b.js"></script>

__Default/Optional Parameters__

In Kotlin you can set default values for method parameters. This makes the parameters optional allowing you to omit them when you want. In the previous examples we have been hard-coding the type of Workbook used as HSSFWorkbook (this an Excel '97 compatible workbook, .xls). What is a user wants to use a XSSFWorkbook (Excel 2007, .xlsx) instead? It would be nice if a user could provide this value but if they were not obligated to. With default parameter value we can enable this behavior:
<script src="https://gist.github.com/ciferkey/e24267a779c53de742a4dda51da45341.js"></script>

__Extension Methods__

So after all that work we now have a nice way to create and modify Workbooks. What about Sheets, Rows and Cells though? For Workbooks we were able to create a default Workbook or use a provided Workbook. Will that work for the other parts? With Apache POI you need a parent object to create a child object:
<script src="https://gist.github.com/ciferkey/1f4bc092baf73b87845b5c30b7398d58.js"></script>

This is the example from the beginning we would like to create:
<script src="https://gist.github.com/ciferkey/a0b1421b853b6c9ccaa4da308fe3fe58.js"></script>

You will notice that inside the lambda block passed to the "workbook" method we just created we want to have a similar looking "sheet" statement. If you think back to the explanation given earlier that  *statement* would actually be a call to a method named "statement" on the implicit Workbook object. However Workbook is a class from Apache POI that we do not have control over and it does not have any method like that! So what we need is a clean mechanism to enhance the existing Java classes. If we were working in Java some options to achieve this are:

* Use inheritance to extend the classes in the library to add new functionality. In this case it would generally be frowned upon. Item 18 in Effective Java (Favor composition over inheritance) makes the important distinction between using inheritance within your own projects vs across project boundaries:
> It is safe to use inheritance within a package, where the subclass and the superclass implementations are under the control of the same programmers. It is also safe to use inheritance when extending classes specifically designed and documented for extension (Item 19). Inheriting from ordinary concrete classes across package boundaries,however, is dangerous. As a reminder, this book uses the word “inheritance” to mean implementation inheritance (when one class extends another). The problems discussed in this item do not apply to interface inheritance (when a class implements an interface or when one interface extends another). Unlike method invocation, inheritance violates encapsulation [Snyder86]. In other words, a subclass depends on the implementation details of its superclass for its proper function. The superclass’s implementation may change from release to release, and if it does, the subclass may break, even though its code has not been touched. As a consequence, a subclass must evolve in tandem with its superclass, unless the superclass’s authors have designed and documented it specifically for the purpose of being extended.
* Use composition to create a new object that wraps the original object and adds new functionality. Its perfectly acceptable to create the wrapper object but it can add noise to the code since you will need to either reach into the wrapper to get the inner class (littering the code with "wrapper.innerObject" references). Alternatively you could write the wrapper so that it has methods that cover every use case for the inner object but that would tightly couple the wrapper to the inner object and break encapsulation.
* Use a static utility class to contain the new functionality. An example of this idea would be [Apache Commons](https://commons.apache.org/) which provides many utility classes. For example the StringUtils class contains static methods like [containsIgnoreCase](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html#containsIgnoreCase-java.lang.CharSequence-java.lang.CharSequence-) which evaluates whether a given string contains a specific string while ignoring case. Static utility classes provides a clean separation of the original class (String) and the new functionality (the static methods) but adds noise to the syntax as you must now write something like StringUtils.containsIgnoreCase(bookText, searchPhrase) (or StringUtils.containsIgnoreCase(bookText, searchPhrase) with a static import). This unfortunately is not the clean call to just "sheet" that we want.

It would be nice if there was a way to enhance the original class with new functionality similar to a static utility class (to avoid breaking encapsulation) but be able to retain the calling conventions we would get if we extended the class. Born from this want Kotlin provides [extension methods](https://kotlinlang.org/docs/reference/extensions.html). Under the hood extensions methods basically generate a static utility class for you while allowing you too call the methods under the original class as if they were a true method belonging to the class.

For example, in Apache POI you can merge cells in a sheet by defining the region that bounds the cells to merge and then saying you want to merge that region:
<script src="https://gist.github.com/ciferkey/1811ad07d27f3780a085efe099ca00bb.js"></script>

In Kotline we can create an extension method called "merge" which directly takes the bounded region and handles creating the CellRangeAddress for you:
<script src="https://gist.github.com/ciferkey/8a58388f9cb97360efbe328525229c5d.js"></script>

Here you will noticed we defined the method as *Sheet.merge*. This is the special syntax for an extension method. It says we are defining the "merge" method on the existing "Sheet" type from Apache POI.

Extension methods are a powerful basic language feature that we need to use to build a DSL for an existing library. Going back to the previous question of how to implement the "sheet" method we can do something like this:
<script src="https://gist.github.com/ciferkey/c10b422d1a464619a09d5a822e6ce2e2.js"></script>

This leverages extensions methods as well as all the techniques from the previous sections. I even threw in some other Kotlin features *on top of that* (sorry not sorry). Breaking it down:

> Workbook.sheet(

* This is an extension method adding "sheet" to the Workbook class

> name: String? = null

* You can optionally pass a first parameter "name"
* Why is the parameters type "String?" and not "String"? This is a very important Kotlin feature that I'm glossing over for now. Basically the type "String" can never be null. "String?" is a "String" that can also be null. This means null safety can be *checked and enforced at compile time* forcing authors to think about how to handle nulls.

> block: Sheet.() -> Unit = {}

* you can optionally pass a parameter "block". If you don't supply it then a lambda that does nothing is used.

> return if (name != null) {
        createSheet(name)
    } else {
        createSheet()
    }

* In Kotlin if statements with else branches are actually *expressions*. This means the statement will be the final value of the branch that is evaluated (very similar to the ternary operator "?:" in Java). In this example if name is not null it will create a sheet with that name. Otherwise it will create a sheet using the default name. Kotlin actually has a lot of very cool operators to handle [null safety](https://kotlinlang.org/docs/reference/null-safety.html) but I opted not to use them here for clarity. 

> .apply(block)

* In Java we have Object that all objects inherit from. It gives us sad methods likes the default equals and default toString. In Kotlin we have Any which gives us a bunch of [actually useful methods](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-any/index.html) like apply. Apply will call a supplied method on an object and then return the object. By using apply hear we avoid needing to make the block parameter return its input. This in turn means you don't have to worry about returning anything when you write the block to pass in!

Putting that all together, the sheet method essentially adds a new method to Workbooks which takes a block that modifies a the work and then returns the Workbook so that we can chain methods as needed.

Whew... wait we still need row and cell. They leverage the same design more or less:
<script src="https://gist.github.com/ciferkey/393fec4083f9c68a8233257ad34c7a18.js"></script>

Some more Kotlin features here that could use more complete explainations but I'm glossing over since this is about DSLs and not a full Kotlin language tutorial:

* What are the "rowNumber ?: physicalNumberOfRows" and "column ?: physicalNumberOfCells" doing? Well remember those null operators I mentioned? "?:" is the "elvis operator". If the first value is null then the operator evaluates to second value instead.
* What is that "when" part? That is a "When Expression". Think of it like a better and safer switch statement. There are actually a couple different ways to use it. In this scenario the cell method is optionally taking value parameter which can be any of the different types that Apache POI supports for cell values. Cell has overloaded the setCellValue with a different implementation for each of the types it supports. This breaks unfortunately the automatic property access syntax so we use a When Expression to handle it. The branches in the When Expression matches the different types for "value" using the "is" keyword (similar to instanceOf in Java). Then we can call "cell.setCellValue(value)". Note that we do not need to explicitly cast the type of value to the type we matched. This is because Kotlin has [smart casts](https://kotlinlang.org/docs/reference/typecasts.html#smart-casts). Since the branch already checks the type the compiler knows its correct and does not require us to specify it. Features like this are how you know that Kotlin was written by people who have spent many days writing Java and wishing for something basically the same but less soul crushing.

With that we now can use the example that was given in the beginning! Along with the extension methods above I have also implemented some additional extension methods for common scenarios. They can all be viewed in the [repository](https://github.com/ciferkey/kpoi/tree/master/src/main/kotlin/com/matthewbrunelle)

### Testing Realistic Use Cases for kpoi

While the documentation for Apache POI can sometimes be sparse there does exist the great [Busy Developers' Guide to HSSF and XSSF Features](https://poi.apache.org/components/spreadsheet/quick-guide.html). It contains a series of code examples for common use cases and problems. To build out the tests for the DSL I took the example Java code and use it to generate workbooks. Then I create the equivalent workbooks using the Kotlin DSL and compare the two. This helps provide some robust test cases that are modeled after real world usage.

An unfortunate aspect of Apache POI is that none of the classes implement Equals() due to the authors deciding there was not a clear concept for "equivalent workbooks". Because of this I had to resort to using reflection based comparison for the workbooks. My personal preference would be to use AssertJ but there is currently [ongoing work](https://github.com/joel-costigliola/assertj-core/issues/1002) around the recursive comparison API so instead I opted to use Unitil's [assertReflectionEquals](http://www.unitils.org/apidocs/org/unitils/reflectionassert/ReflectionAssert.html#assertReflectionEquals(java.lang.Object,%20java.lang.Object,%20org.unitils.reflectionassert.ReflectionComparatorMode...)) which recursively uses reflection to compare objects field by field.


### Potential future work
Note that this DSL is specifically designed to cover declaratively creating workbooks. It has nothing to do with reading in notebooks and manipulating which is another use case for Apache POI. That is because I have primarily used it to generate workbooks. I still there is interesting work that could be done around read side as well to make it more pleasant.
