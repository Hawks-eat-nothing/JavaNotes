# jvm方法调用

[TOC]

## 1. 解析

所有方法在Class文件都是一个常量池中的符号引用，类加载的解析阶段会将其转换成直接引用，这种解析的前提是：**要保证这个方法在运行期是不可变的。**这类方法的调用称为解析。

jvm提供了5条方法调用字节码指令：

- `[] invokestatic`：调用静态方法
- `[] invokespecial`：调用构造器方法、私有方法和父类方法
- `[] invokevirtual:`：调用所有的虚方法。
- `[] invokeinterface`：调用接口方法，会在运行时期再确定一个实现此接口的对象
- `[] invokedynamic`：现在运行时期动态解析出调用点限定符所引用的方法，然后再执行该方法，在此之前的4条指令，分派逻辑都是固化在虚拟机里面的，而invokedynamic指令的分派逻辑是由用户所设定的引导方法决定的。InvokeDynamic指令详细请点击[InvokeDynamic](https://blog.csdn.net/zxhoo/article/details/38387141?spm=a2c6h.12873639.0.0.58803093UhlRsB)

被`invokestatic`和`invokespecial`指令调用的方法，都能保证方法的不可变性，符合这个条件的有**静态方法、私有方法、实例构造器、父类方法**4类。这些方法称为非虚方法。

```java
public class Main {
    public static void main(String[] args) {
        //invokestatic调用静态方法
        Test.hello();
        //invokespecial调用构造器方法
        Test test = new Test();
    }
    static class Test{
        static void hello(){
            System.out.println("hello");
        }
    }
}
```

**解析调用一定是一个静态的过程**，在编译期间就可以完全确定，在类装载的解析阶段就会把涉及的符号引用全部转化为可确定的直接引用，不会延迟到运行期去完成。而分派调用可能是静态的也可能是动态的，根据分派一句的宗量数可分为单分派和多分派。因此分派可分为：**静态单分派、静态多分派、动态单分派、动态多分派。**

### 静态分派（方法重载）

>所有依赖静态类型来定位方法执行版本的分派动作称为静态分派。

```java
public class Test {
    static class Phone{}
    static class Mi extends Phone{}
    static class Iphone extends Phone{}

    public void show(Mi mi){
        System.out.println("phone is mi");
    }
    public void show(Iphone iphone){
        System.out.println("phone is iphone");
    }
    public void show(Phone phone){
        System.out.println("phone parent class be called");
    }

    public static void main(String[] args) {
        Phone mi = new Mi();
        Phone iphone = new Iphone();

        Test test = new Test();
        test.show(mi);
        test.show(iphone);
        test.show((Mi)mi);//将mi强转成Mi
    }
}
```

执行结果

> phone parent class be called
> phone parent class be called
> phone is mi

我们把上面代码中的Phone称为==变量的静态类型或者叫外观类型==，把Mi和Iphone称为==实际类型==，**静态类型仅仅在使用时发生变化**，编译可知；**实际类型在运行期才知道结果**，编译器在编译程序的时候并不知道一个对象的实际类型是什么。

所以，jvm重载时是通过参数的静态类型而不是实际类型作为判定依据。下图可以证明：

根据上面的代码也可以看出，我们可以使用强制类型转换来使静态类型发生改变。

### 动态分派（方法覆盖）

```java
public class Test2 {
    static abstract class Phone{
        abstract void show();
    }
    static class Mi extends Phone{
        @Override
        void show() {
            System.out.println("phone is mi");
        }
    }
    static class Iphone extends Phone{
        @Override
        void show() {
            System.out.println("phone is iphone");
        }
    }

    public static void main(String[] args) {
        Phone mi = new Mi();
        Phone iphone = new Iphone();
        mi.show();
        iphone.show();
        mi = new Iphone();
        mi.show();
    }
}
```

> phone is mi
> phone is iphone
> phone is iphone

`invokevirtual`指令的第一步就是在运行期确定接受者的实际类型。两次调用`invokevirtual`指令把常量池中的类方法符号引用解析到了不同的直接引用上。

**`invokevirtual`指令的运行时解析过程大致分为以下几个步骤:**

1. 找到操作数栈顶的第一个元素(对象引用)所指向的对象的实际类型，记作C；
2. 如果在类型C中找到与常量中的描述符和简单名称都相符的方法，则进行访问权限校验，如果通过则返回这个方法的直接引用，查找过程结束；如果不通过，则返回`java.lang.IllegalAccessError`。
3. 否则，按照继承关系从下往上依次对C的各个父类进行第2步的搜索和验证。
4. 如果始终没有找到合适的方法，则抛出`java.lang.AbstractMethodError`异常。

### 动态类型语言支持

动态语言的关键特征是它的类型检查的主体过程是在运行期间而不是编译期。相对的，在编译期间进行类型检查过程的语言（java、c++）就是静态类型语言。

运行时异常：代码只要不运行到这一行就不会报错。
连接时异常：类加载抛出异常。

静态类型语言在编译期确定类型，可以提供严谨的类型检查，有很多问题编码的时候就能及时发现，利于开发稳定的大规模项目。动态类型语言在运行期确定类型，有很大的灵活性，代码更简洁清晰，开发效率高。




