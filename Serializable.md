## Serializable
(屁话)Serializable接口是启用其序列化功能的接口。实现java.io.Serializable 接口的类是可序列化的。**没有实现此接口的类将不能使它们的任意状态被序列化或逆序列化**。

==序列化的过程，就是一个“freeze”的过程，它将一个对象freeze（冷冻）住，然后进行存储，等到再次需要的时候，再将这个对象de-freeze就可以立即使用。==


我们以为的没有进行序列化，其实是在声明的各个不同变量的时候，由具体的数据类型帮助我们实现了序列化操作。

Serializable接口其实是个空接口，当我们让实体类实现Serializable接口时，**其实是在告诉JVM此类可被序列化，可被默认的序列化机制序列化。**

### 序列化的作用：

**序列化是将对象状态转换为可保持或传输的格式的过程。**与序列化相对的是**反序列化，将流转换为对象**。这两个过程结合起来，可以轻松地存储和传输数据。

**1，存储对象在存储介质中，以便在下次使用的时候，可以很快捷的重建一个副本。**也就是When the resulting series of bits is reread according to the serialization format, it can be used to create a semantically identical clone of the original object.

<font color=#008000 >问题：我没有实现序列化的时候，我一样可以存入到我的sqlserver或者MySQL、Oracle数据库中啊，为什么一定要序列化才能存储呢？？？？</font>

**2，便于数据传输，尤其是在远程调用的时候！**

实现java.io.Serializable这个接口是为序列化,serialVersionUID 用来表明实现序列化类的不同版本间的兼容性。如果你修改了此类, 要修改此值。否则以前用老版本的类序列化的类恢复时会出错。

实现后如果你用的是工具的话会出来警告,他会提示你,可以自动生成private static final long serialVersionUID = 1L; 为了在反序列化时，确保类版本的兼容性，最好在每个要序列化的类中加入private static final long serialVersionUID这个属性，具体数值自己定义.

关于serialVersionUID的解释

**serialVersionUID作用： 序列化时为了保持版本的兼容性，即在版本升级时反序列化仍保持对象的唯一性。**

它替你生成有两种生成方式：

一个是默认的1L，比如：private static final long serialVersionUID = 1L;

一个是根据类名、接口名、成员方法及属性等来生成一个64位的哈希字段，

比如：private static final long serialVersionUID = -8940196742313994740L;之类的。