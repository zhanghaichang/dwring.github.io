---
layout: post
title: Java自定义Annotation,通过反射解析Annotation
category: java
tags: [java]
---

## 创建一个自定义的Annotation

```java
import java.lang.annotation.*;
import java.lang.reflect.Method;

@Documented
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MethodInfo {
    String author() default "hupeng";
    String version() default "1.0";
    String date();
    String comment();

}
```

*   Annotation methods can’t have parameters.
*   Annotation methods return types are limited to primitives, String, Enums, Annotation or array of these.
*   Annotation methods can have default values.

### @Target：

@Target说明了Annotation所修饰的对象范围：Annotation可被用于 packages、types（类、接口、枚举、Annotation类型）、类型成员（方法、构造方法、成员变量、枚举值）、方法参数和本地变量（如循环变量、catch参数）。在Annotation类型的声明中使用了target可更加明晰其修饰的目标。

作用：用于描述注解的使用范围（即：被描述的注解可以用在什么地方）

取值(ElementType)有：

1.CONSTRUCTOR:用于描述构造器
2.FIELD:用于描述域
3.LOCAL_VARIABLE:用于描述局部变量
4.METHOD:用于描述方法
5.PACKAGE:用于描述包
6.PARAMETER:用于描述参数
7.TYPE:用于描述类、接口(包括注解类型) 或enum声明

### @Retention：

@Retention定义了该Annotation被保留的时间长短：某些Annotation仅出现在源代码中，而被编译器丢弃；而另一些却被编译在class文件中；编译在class文件中的Annotation可能会被虚拟机忽略，而另一些在class被装载时将被读取（请注意并不影响class的执行，因为Annotation与class在使用上是被分离的）。使用这个meta-Annotation可以对 Annotation的“生命周期”限制。

作用：表示需要在什么级别保存该注释信息，用于描述注解的生命周期（即：被描述的注解在什么范围内有效）

取值（RetentionPoicy）有：

1.SOURCE:在源文件中有效（即源文件保留）
2.CLASS:在class文件中有效（即class保留）
3.RUNTIME:在运行时有效（即运行时保留）

### @Documented:

@Documented用于描述其它类型的annotation应该被作为被标注的程序成员的公共API，因此可以被例如javadoc此类的工具文档化。Documented是一个标记注解，没有成员。

### @Inherited：

@Inherited 元注解是一个标记注解，@Inherited阐述了某个被标注的类型是被继承的。，则这个annotation将被用于该class的子类。

注意：@Inherited annotation类型是被标注过的class的子类所继承。类并不从它所实现的接口继承annotation，方法并不从它所重载的方法继承annotation。

## Java 内置的Annotation

从java5版本开始，自带了三种标准annontation类型,

*   Override，java.lang.Override 是一个marker annotation类型，它被用作标注方法。它说明了被标注的方法重载了父类的方法，起到了断言的作用。如果我们使用了这种annotation在一个没有覆盖父类方法的方法时，java编译器将以一个编译错误来警示。这个annotaton常常在我们试图覆盖父类方法而确又写错了方法名时加一个保障性的校验过程。
*   Deprecated，Deprecated也是一种marker annotation。，编译器将不鼓励使用这个被标注的程序元素。所以使用这种修饰具有一定的 “延续性”：如果我们在代码中通过继承或者覆盖的方式使用了这个过时的类型或者成员，虽然继承或者覆盖后的类型或者成员并不是被声明为 @Deprecated，但编译器仍然要报警。注意：@Deprecated这个annotation类型和javadoc中的 @deprecated这个tag是有区别的：前者是java编译器识别的，而后者是被javadoc工具所识别用来生成文档（包含程序成员为什么已经过时、它应当如何被禁止或者替代的描述）。
*   SuppressWarnings，此注解能告诉Java编译器关闭对类、方法及成员变量的警告。有时编译时会提出一些警告，对于这些警告有的隐藏着Bug，有的是无法避免的，对于某些不想看到的警告信息，可以通过这个注解来屏蔽。SuppressWarning不是一个marker annotation。它有一个类型为String[]的成员，这个成员的值为被禁止的警告名。对于javac编译器来讲，，同时编译器忽略掉无法识别的警告名。

下面我们来使用Java内置的Annotation 和 自定义的Annotation

```java
public class AnnotationExample {
    @Override
    @MethodInfo(author = "xxx",version = "1.0",date = "2015/03/26",comment = "override toString()")
    public String toString() {
        return "AnnotationExample{}";
    }

    @Deprecated
    @MethodInfo(comment = "deprecated method", date = "2015/03/26")
    public static void oldMethod() {
        System.out.println("old method, don't use it.");
    }

    @SuppressWarnings({ "unchecked", "deprecation" })
    @MethodInfo(author = "Pankaj", comment = "Main method", date = "Nov 17 2012", version = "1.0")
    public static void genericsTest() {
        oldMethod();
    }
}
```

## 使用反射来解析Annotation

注意我们的Annotation的Retention Policy 必须是RUNTIME，否则我们无法在运行时从他里面获得任何数据。

```java
import java.lang.annotation.Annotation;
import java.lang.reflect.Method;

/**
 * Created by Administrator on 2015/3/26.
 */
public class AnnotationParsing {

    public static void main(String[] args) {
        for (Method method: AnnotationExample.class.getMethods()) {
            if (method.isAnnotationPresent(MethodInfo.class)) {
                for (Annotation annotation:method.getAnnotations()) {
                    System.out.println(annotation + " in method:"+ method);
                }

                MethodInfo methodInfo = method.getAnnotation(MethodInfo.class);

                if ("1.0".equals(methodInfo.version())) {
                    System.out.println("Method with revision no 1.0 = "
                            + method);
                }
            }
        }
    }
}
```
