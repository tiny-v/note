> Tips:
>
>1. 原版英文文档地址： [https://github.com/EsotericSoftware/kryo/blob/master/README.md#recent-releases](https://github.com/EsotericSoftware/kryo/blob/master/README.md#recent-releases) 
>
>2. 文章为作者自行翻译，不当之处请予以指正，转载请注明出处。

![kryo 图标](https://img-blog.csdnimg.cn/20200704170524714.jpg)

Kryo 是一个快速高效的Java二进制对象图(binary object graph)序列化框架。Kryo 以快速的序列化/反序列化，较小的序列化结果和方便易用的API为目标。 无论对象是需要持久化到文件、写入数据库、或是通过网络来传输， Kryo都非常的有用。

同时 Kryo 可以自动对对象进行深浅 拷贝/克隆。这是个从对象直接到对象的过程， 中间不需要将 object 先转成 bytes，再从 bytes 转成 object 来过渡。

<h4>联系方式：</h4>

如果有任何问题，交流，和对 Kryo 的支持，请通过 [Kryo mailing list](https://groups.google.com/forum/#!forum/kryo-users)  联系我们。对于Kryo 问题跟踪器的使用，仅限于提出 Kryo 的 bug 和优化，而非上述的问题，交流，和对Kryo 的支持。

&nbsp;
&nbsp;

@[TOC]

&nbsp;

##  1. Recent releases

---

[5.0.0-RC2](https://github.com/EsotericSoftware/kryo/releases/tag/kryo-parent-5.0.0-RC2) 基于反馈，对RC1进行改进且第二次发布。 参见 [Migration to v5](https://github.com/EsotericSoftware/kryo/wiki/Migration-to-v5).
[5.0.0-RC1](https://github.com/EsotericSoftware/kryo/releases/tag/kryo-parent-5.0.0-RC1) 修复了许多问题，并做出了许多期待已久的改进。
[4.0.2](https://github.com/EsotericSoftware/kryo/releases/tag/kryo-parent-4.0.2) 做了一些修复和改进。

&nbsp;
## 2. Installation

---
在 [releases page](https://github.com/EsotericSoftware/kryo/releases) 和 [Maven Central](https://search.maven.org/#search%7Cgav%7C1%7Cg%3Acom.esotericsoftware%20a%3Akryo) 可以找到Kryo Jar；最新的Kryo snapshots 版本, 包括对主版本的 snapshot builds, 都在 [Sonatype Repository](https://oss.sonatype.org/content/repositories/snapshots/com/esotericsoftware/kryo/).

### 2.1 with maven

使用最新的 Kryo 发布版本, 在pom.xml 引入下面的 dependency:
 ```
<dependency>
   <groupId>com.esotericsoftware</groupId>
   <artifactId>kryo</artifactId>
   <version>5.0.0-RC2</version>
</dependency>
```
想使用最新的snapshot版本，则引入:
```
<repository>
   <id>sonatype-snapshots</id>
   <name>sonatype snapshots repo</name>
   <url>https://oss.sonatype.org/content/repositories/snapshots</url>
</repository>

<dependency>
   <groupId>com.esotericsoftware</groupId>
   <artifactId>kryo</artifactId>
   <version>5.0.0-RC3-SNAPSHOT</version>
</dependency>
```

### 2.2 without Maven
 并非每人都是 maven 的粉丝，在不使用 Maven 的情况下使用 Kryo 需要将 Kryo jar 与lib中的依赖 jar 一起放在类路径下。

&nbsp;
## 3. Quickstart

---
提前展示一下如何使用 Kryo 类库
```
import com.esotericsoftware.kryo.Kryo;
import com.esotericsoftware.kryo.io.Input;
import com.esotericsoftware.kryo.io.Output;
import java.io.*;

public class HelloKryo {
   static public void main (String[] args) throws Exception {
      Kryo kryo = new Kryo();
      kryo.register(SomeClass.class);

      SomeClass object = new SomeClass();
      object.value = "Hello Kryo!";

      Output output = new Output(new FileOutputStream("file.bin"));
      kryo.writeObject(output, object);
      output.close();

      Input input = new Input(new FileInputStream("file.bin"));
      SomeClass object2 = kryo.readObject(input, SomeClass.class);
      input.close();   
   }
   static public class SomeClass {
      String value;
   }
}
```
Kryo 类自动进行序列化. Output 和 Input 类处理缓冲字节并视情况写入流中。
该文档其余部分描述 Kryo 具体如何工作，并介绍该类库的高级用法。

&nbsp;

## 4. IO

---
使用 Input 和 Output 类来操作数据进出Kryo，  这些类并不是线程安全的。

### 4.1 Output

Output 类是一个用来写入数据到字节数组缓冲区(byte array buffer)的输出流(OutputStream)。如果想得到字节数组，可以直接获取并使用这个缓冲区。在已经使用 OutputStream 初始化了Output对象的情况下， 若缓冲区满了，Output 会将缓冲区数据写入流中，否则会先将字节数据写入缓冲区。Output 类有许多方法可以高效的将基本类型(primitives)和字符串转成字节。Output 类中，提供了类似 DataOutputStream、 BufferedOutputStream、 FilterOutputStream 和 ByteArrayOutputStream 的方法。

>Tip: Output 类包含了 ByteArrayOutputStream 所有的功能. 所以基本不需要将Output 转成 ByteArrayOutputStream.

当要写入数据到一个 OutputStream 时， Output 会先将字节数据写入缓冲区，所以在写入动作结束时，必须要执行 **flush** 或 **close** 方法， 以确保缓冲区的数据都会被刷到OutputStream。如果没有用 OutputStream 初始化一个 Output，最后就没有必要执行 **flush** 或 **close**。和其它的很多流不同， 一个Output对象实例可以通过被重置位置 或 赋一个新的字节数组或输出流来实现复用。
 
>Tip: 如果 Output buffer已存在, 就没必要将其写入BufferedOutputStream 了。

调用 Output 类 的无参构造函数会得到一个未初始化的 Output 对象，在使用这个对象前，必须先调用 **setBuffer** 方法。

![译者图：output stream.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xNTkxOTAzMy1kMWMwYjRiODVlNWFkMjIwLnBuZw?x-oss-process=image/format,png)

### 4.2 Input

Input 类用来从一个字节数组缓冲区(byte array buffer)读取数据的输入流(InputStream)。 当想从一个字节数组中读取数据时，可直接使用这个缓冲区。如果已经用 InputStream 初始化了这个 Input 对象，当缓冲区的数据已经都被读取，这个流将会再次写满缓冲区。Input 类有许多方法可以高效的读取字节数据来得到基本类型(primitives)和字符串。Input类中，提供了类似 DataInputStream,  BufferedInputStream,  FilterInputStream 和 ByteArrayInputStream的方法。

>Tip: Input类包含了ByteArrayInputStream 所有的功能. 所以基本不需要通过ByteArrayInputStream 来获得Input.

如果调用了 Input 类中的 **close** 方法， 这个 Input 的 InputStream 就被关闭了(如果有的话)。而若是这个Input类不含有 InputStream，就没必要调用 **close** 方法。和其它流不一样的是，一个 Input 类的实例可以通过被重置位置或给一个新的字节数组或输入流来实现复用。调用 Input 类的无参构造函数会得到一个未初始化的 Input 对象， 在使用这个对象前， 必须要调用 **setBuffer** 方法。
![译者图：input stream.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xNTkxOTAzMy1mNWYwNDI1NDkwNGE5OTI0LnBuZw?x-oss-process=image/format,png)

### 4.3 ByteBuffers

对于 ByteBufferOutput 类 和 ByteBufferInput类，  除了使用的是字节缓冲区(ByteBuffer) ，而不是字节数组(byte array)， 其使用方法和 Output 类和 Input 类完全一样。

对于 UnsafeOutput、 UnsafeInput、 UnsafeByteBufferOutput 和 UnsafeByteBufferInput 类， 除了在许多情况下，它们为了更好的性能而使用sun.misc.Unsafe 包提供的方法外，其它的用法和与其对应的，不加Unsafe前缀的类的用法相同。而为了使用这些类，**Util.unsafe** 方法必须为 true。

  使用不安全缓冲区(unsafe buffers)的缺点是 系统本地的字节序列(endianness)和数值类型(numeric types)的表示方法会在序列化时影响到序列化的结果。比如，在X86上序列化写入数据，在SPARC(译者注:SUN和TI公司合作开发了RISC的微处理器)读取进行反序列化就会失败。同时， 如果数据写入时是使用的 unsafe buffer， 读取时也必须要使用 unsafe buffer。unsafe buffer的最大性能差异在于不使用变长编码(variable length encoding)的情况下使用较大的原始数组([large primitive arrays](https://raw.github.com/wiki/EsotericSoftware/kryo/images/benchmarks/array.png))。unsafe buffer可以禁用变长编码， 或在使用 FieldSerializer(kryo的一种序列化器) 时, 只对特定字段使用变长编码。

### 4.4 Unsafe buffers
### 4.5 Variable length encoding
IO 类提供了读写不定长的int(varint) 型 和 long(varlong) 型值的方法，它是通过使用每个字节的第 8 位来表示后面是否有更多字节来实现的。即一个 varint 型变量占用1 - 5个字节，一个varlong型变量占用1 - 9个字节。使用变长编码来序列化的过程代价更高，但生成的序列化结果会更小。
当写入未知长度数值时，无论该值是正是负，都可以被优化。比如，当选择对正值进行优化时，0 - 127 占用一个字节，128 - 16383 占用两个字节。但小负数是最坏的情况，占5个字节。当选择不对正值做优化时，这个范围就会减小一半，比如，-64 - 63 写入一个字节，64 - 8191 和 -65 - -8192 占用两个字节。
Input 和 Output 缓冲区提供了读写定长/不定长数值的方法，还提供了一些让缓冲区来决定是写入定长值或不定长值的方法，这就使得序列化代码可确保变长编码可用于非常常见的值，而如果使用定长编码，这些常见值可能已经从缓冲区溢出。同时仍然允许缓冲区去配置决定其它的值。
方法|描述
---|---
writeInt(int)|写入一个 4 个字节的 Int 值
writeVarInt(int, boolean)|写入一个 1-5 个字节的 Int 值
writeInt(int, boolean)|写入一个 4 个字节 或 1-5 个字节的 Int 值 （由缓冲区决定）
writeLong(long)|写入一个 8 个字节的 long 值
writeVarLong(long, boolean)|写入一个 1-9 个字节的 long 值
writeLong(long, boolean)|写入一个 8 个字节 或 1-9 个字节的 long 值 （由缓冲区决定）
若想对所有值都禁用变长编码，需要重写 **writeVarInt**、**writeVarLong**、**readVarInt** 和 **readVarLong** 方法。

### 4.6 Chunked encoding(分块编码)

先记录一些数据的长度，再写入数据是很有用的。 当数据长度未知时，只有所有的数据都写入了缓冲区才能得到再记录数据长度。这样就要用到一个足够大的缓冲区，以此防止缓冲区满了数据先进入流，但使用如此大的缓冲区并不合理也不明智。

分块编码(Chunked encoding) 仅使用一个小的缓冲区就能解决这个问题。当缓冲区满了就先记录数据长度，再将缓冲区数据写入一个chunk; 清除这个缓冲区后，继续写入数据到该缓冲区再将缓冲区数据写入下一个chunk，如此循环没有更多的数据需要写入。当一个 chunk 的长度为 0， 就代表这组chunk的结束。
![译者图：chunk encoding.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xNTkxOTAzMy03MDg3YmM1MmZiYjk0OTgxLnBuZw?x-oss-process=image/format,png)

Kryo提供了进行分块编码的类。
OutputChunked 类用来写入数据到chunk，它继承 Output 类，有全部的用来写入数据的方便方法。当 OutputChunked 的缓冲区满了时，就会将数据从 chunk 刷到另一个 OutputStream 中。**endChunks** 方法用来标识一组 chunk 的结束。
```
OutputStream outputStream = new FileOutputStream("file.bin");
OutputChunked output = new OutputChunked(outputStream, 1024);
// Write data to output...
output.endChunks();
// Write more data to output...
output.endChunks();
// Write even more data to output...
output.close();
```
InputChunked 类用来从 chunk 中读取数据，它继承 Input 类，有全部的用来读取数据的方便方法。当读取数时，InputChunked 会直接到达这组 chunk 的最后一个 chunk 的结尾。即使当前这组 chunk 的数据还没被读完，调用 **nextChunks** 方法会提前读下一组 chunk 的数据。
```
InputStream outputStream = new FileInputStream("file.bin");
InputChunked input = new InputChunked(inputStream, 1024);
// Read data from first set of chunks...
input.nextChunks();
// Read data from second set of chunks...
input.nextChunks();
// Read data from third set of chunks...
input.close();
```

### 4.7 Buffer performance

总的来说，Output 和 Input 性能不错；但若不考虑平台兼容性，Unsafe buffers 的性能甚至更好，尤其是对基本数组来说。ByteBufferOutput 和 ByteBufferInput 的性能稍微差一点，如果最终目标是字节缓冲区(ByteBuffer)，那性能也能可以接受。
![image](https://imgconvert.csdnimg.cn/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzE1OTE5MDMzLTg5NDMxMzNkNWVkOTAzZmE?x-oss-process=image/format,png)

![image](https://imgconvert.csdnimg.cn/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzE1OTE5MDMzLTBhMTk2MDY2YjJiNDkwNWE?x-oss-process=image/format,png)

变长编码 (variable-length encoding) 的速度比定长要慢，尤其当大量的数据都使用变长编码。
![image](https://imgconvert.csdnimg.cn/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzE1OTE5MDMzLWU2ZWQ3NGVjZjg5NGQxNmU?x-oss-process=image/format,png)

分块编码 (chunked encoding) 使用了一块中间缓冲区，所以会额外复制一份数据。这也许能接受，但若使用可重入序列化器(reentrant serializer) 时，就必须为每个对象都创建一个OutputChunked 或 InputChunked，执行序列化时，缓冲区的内存分配和GC就会影响性能。
![image](https://imgconvert.csdnimg.cn/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzE1OTE5MDMzLTdkN2FhNDZjOWU5MGNkMjY?x-oss-process=image/format,png)

&nbsp;

## 5. Reading and writing objects

---
Kryo有三组方法用来读写对象。

如果 对象类型未知 或 对象为 null 时：
```
kryo.writeClassAndObject(output, object);

Object object = kryo.readClassAndObject(input);
if (object instanceof SomeClass) {
   // ...
}
```

当 类型已知，对象可能为null 时：
```
kryo.writeObjectOrNull(output, object);

SomeClass object = kryo.readObjectOrNull(input, SomeClass.class);
```

当 类型已知 且 对象不可能为null 时：
```
kryo.writeObject(output, object);

SomeClass object = kryo.readObject(input, SomeClass.class);
```

所有这些方法都要先找到合适的序列化器(serializer)，然后用序列化器来序列化和反序列化对象。序列化器能够调用这些方法来实现递归的序列化。对同一个对象的多次引用和循环引用会由 Kryo 自动解决。
除了用来读写对象的方法外，Kryo 类提供了一种方法来注册序列化器，高效的读写类标识符，为不能接受空值的序列化器处理空对象，以及处理读写对象引用(如果启用的话)。让序列化器能专注于它们的序列化任务。

### 5.1 Round trip
当测试和研究 Kryo API 时， 把一个对象转成字节数组， 再从字节数组读回这个对象，是很用帮助的。
```
Kryo kryo = new Kryo();

// Register all classes to be serialized.
kryo.register(SomeClass.class);

SomeClass object1 = new SomeClass();

Output output = new Output(1024, -1);
kryo.writeObject(output, object1);

Input input = new Input(output.getBuffer(), 0, output.position());
SomeClass object2 = kryo.readObject(input, SomeClass.class);
```
在这个例子里， Output 有一个 1024 字节的缓冲区， 如果有更多数据要写入 Output，这个缓冲区的大小可以无限制的增长。而因为这个 Output 没有 OutputStream，所以最后不需要执行 **close** 方法。随后，Input 直接从 Output 的字节数组缓冲区里读取数据。 

### 5.2 Deep and shallow copies(深拷贝和浅拷贝)

Kryo 通过从一个对象直接赋值给另一个对象来实现深拷贝和浅拷贝。 这比把对象先序列化成字节，再从字节转成对象的方式要高效的多。 
```
Kryo kryo = new Kryo();
SomeClass object = ...
SomeClass copy1 = kryo.copy(object);
SomeClass copy2 = kryo.copyShallow(object);
```

所有被使用的序列化器都应该支持复制([copying](https://github.com/EsotericSoftware/kryo/blob/master/README.md#serializer-copying))， 由 Kryo 提供的序列化器都是支持复制的。与序列化类似，在复制时，如果启用了引用，Kryo 会自动处理对同一对象的多次引用和循环引用。如果仅将 Kryo 用于复制，则可以安全地禁用注册。复制对象图之后，可以使用 Kryo 的 **getOriginalToCopyMap** 方法来获得从旧对象到新对象的映射。该映射会自动被 Kryo 重置清除，所以只有当Kryo的 **setAutoReset** 为 false 时才有用。

### 5.3 References

默认情况下，引用是未开启的。 意味着如果一个对象在对象图(object graph)中多次出现， 该对象会被写多次并且会被反序列成多个不同的对象。 当引用未开启， 循环引用会造成序列化失败。 Kryo 通过序列化功能的 **setReferences** 方法和复制功能的 **setCopyReferences** 方法开启或禁用引用。

当引用开启时，在一个对象图中， 每个对象第一次出现时会对应写入一个的变量，若该对象再次出现在这个对象图中，也只会使用这个变量替代。 反序列化后，包括任何循环引用在内的对象引用都将被恢复。 使用序列化器必须通过在序列化器 **read** 中调用 Kryo reference 来支持引用([support references](https://github.com/EsotericSoftware/kryo/blob/master/README.md#serializer-references))。

因为每个被读写对象都要求能被追踪到， 所以开启引用会对性能有影响。

![image](https://imgconvert.csdnimg.cn/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzE1OTE5MDMzLWNjYzVmMTI5NTkzZjhjYjM?x-oss-process=image/format,png)


#### 5.3.1 ReferenceResolver

在后台， ReferenceResolver 负责跟踪已被读写的对象，并返回 int 型的Reference ID。

ReferenceResolver 有多个实现:

1\. MapReferenceResolver: 如果没有特别指定,会默认使用MapReferenceResolver。它使用Kryo的 IdentityObjectIntMap (一种布谷鸟哈希 [cuckoo hashmap](https://en.wikipedia.org/wiki/Cuckoo_hashing)) 来跟踪被写入的对象。 布谷鸟散列的查询(get)速度非常快但不会为插入(put)分配内存，插入极大量的对象会非常的慢。

2\. HashMapReferenceResolver 使用哈希表(HashMap)跟踪被写入的对象。哈希表会为 put 分配内存， 当一个对象图中有大量的对象时，它的表现会更好。

3\. ListReferenceResolver 使用一个数组列表(ArrayList) 去跟踪被写入的对象。当对象图中只有数量相对较少的对象时， 它比使用map要快(一些测试里，大概快 15%)。 因为它是通过线性查找已被写入的对象， 所以不建议在对象图中有大量对象时使用。

ReferenceResolver 的 **useReferences(Class)** 方法可以被重写，它返回一个boolean 来表示一个类是否开启引用。 如果一个类未开启引用， 这个类出现在对象图时，就不会被对应的 reference ID 变量替换。如果一个类不需要引用并且这个类在对象图中多次出现， 那么可以通过禁用引用来极大的减小序列化结果的大小。默认的引用解析器(reference resolver)为所有原始包装器和枚举返回 false。根据序列化的对象图，字符串和其他类通常也返回 false。
```
public boolean useReferences (Class type) {
   return !Util.isWrapperClass(type) && !Util.isEnum(type) && type != String.class;
}
```

#### 5.3.2 Reference limits

引用解析器决定单个对象图中最多可以有多少个引用， java 数组下标最大为 **Interger.MAX_VALUE**, 所以如果引用解析器使用的数据结构是基于数组，会在序列化超过大概 20 亿个对象时，报  **java.lang.negativeArraySizeException**。Kryo 使用的是 Int 类的 ID， 所以在单个对象图中，最多支持从负值到正值整个范围，大概 40 亿 个引用。

### 5.4 Context

Kryo 的 **getContext** 方法返回一个用于存储用户数据的map。Kryo 实例对所有序列化器都可用，因此这些数据对所有序列化器来说都很容易获得。

Kryo 的 **getGraphContext** 方法类似， 但会在每个对象图被序列化或反序列化后清除。 这使得它很容易管理只和当前对象图有关的状态。比如，这可以用于在对象图中第一次遇到类时编写一些模式数据。有关示例，参考 CompatibleFieldSerializer。

### 5.5 Reset

默认在单个对象图全部被序列化后， Kryo 的 **reset** 方法会被执行。 这将重置在类解析器([class resolver](https://github.com/EsotericSoftware/kryo/blob/master/README.md#classresolver))中未注册的类名、和已经在引用解析器( [reference resolver](https://github.com/EsotericSoftware/kryo/blob/master/README.md#referenceresolver)) 中被序列化和反序列化的对象引用，并会清除该对象图的上下文。Kryo 的 **setAutoReset(false)** 方法可用来禁止自动执行 **reset** ，从而允许状态跨越多个对象图。


&nbsp;

## 6. Serializer framework

---
Kryo 是一个能让序列化更简单的框架。 这个框架本身没有固定的模式， 也不关心数据是如何被读写的。 而是让插件式的序列化器来决定读写什么。 许多直接可用的序列化器通过各种方式读写数据。  虽然提供的序列化器可以读写大多数对象，但仍可以轻松地用您自己的序列化器部分或全部替换它们。

当Kryo准备写入一个对象的实例时， 首先它需要写入一些东西来标识这个对象的类。 默认情况下， 所有将被 Kryo 读写的类必须提前被注册。注册机制提供一个int型的class ID、用于该类的序列化器 和 用于创建该类实例的对象实例化器([object instantiator](https://github.com/EsotericSoftware/kryo/blob/master/README.md#object-creation))。
```
Kryo kryo = new Kryo();
kryo.register(SomeClass.class);
Output output = ...
SomeClass object = ...
kryo.writeObject(output, object);
```
在反序列化期间，已注册的类必须具有与序列化期间完全相同的 ID。 注册时，将为类分配下一个可用的最小整数ID，这意味着类的注册顺序很重要。可以选择性地显式为类指定 ID，使这个顺序变的不重要。

```
Kryo kryo = new Kryo();
kryo.register(SomeClass.class, 9);
kryo.register(AnotherClass.class, 10);
kryo.register(YetAnotherClass.class, 11);
```
Class IDs : -1 和 -2 被保留了， 0-8 默认给基本类型和 String 使用， 但也可以另作它用。这些 ID 是通过使用正值优化的方法被写入的， 所以当它们是一些较小的整形数据时，效率很高；负的ID被序列化的效率就差了。

### 6.1 Registration

#### 6.1.1 ClassResolver

在底层， 实际是 类解析器(ClassResolver) 通过读写字节来表示一个类。 在多数情况下，其默认的实现已经够用了。 但在一个类被注册时， 或序列化时有未注册的类，以及想通过读写其它东西来表示一个类时，都可以用自己的类解析器来替换原有的。

#### 6.1.2 Optional registration

Kryo可以配置为不用预先注册类，就允许序列化。
```
Kryo kryo = new Kryo();
kryo.setRegistrationRequired(false);
Output output = ...
SomeClass object = ...
kryo.writeObject(output, object);
```

被注册的类和未被注册的类可以混合使用。 但未被注册的类主要有两个缺点：
   1.  存在安全问题，因为它允许通过反序列化来创建任何类的实例。在类的构造或终结期间，具有副作用的类可能被用于恶意目的。

  2.  与 varint 的 Class ID(通常为1-2字节)不同，在对象图中首次出现未注册的类时，将写入类的全路径。在同一个对象图中，再次出现该类则使用可变类型(varint)来写入。可以考虑使用较短的包名来减少序列化结果的大小。

如果只是将 Kryo 用于复制， 就可以安全的禁止注册功能了。

当不需要预注册时，若是遇到未被注册的类，使用 Kryo 的  **setWarnUnregisteredClasses** 方法可以打印出日志， 从而很容易拿到一个包含所有未注册类的列表。 开发人员可以通过重写 Kryo 的 **unregisteredClassMessage** 方法来自定义日志的内容或做别的事。

### 6.2 Default serializers

注册类时， 可以为其指定序列化器。 但在反序列化期间， 该注册类必须使用和在序列化时一样的序列化器，且序列化器的配置也要相同。
```
Kryo kryo = new Kryo();
kryo.register(SomeClass.class, new SomeSerializer());
kryo.register(AnotherClass.class, new AnotherSerializer());
```

当没有指定序列化器或遇到一个未被注册的类时， 就会从默认序列化器列表中，自动选择一个来和类做映射。 有多个默认的序列化器不会影响序列化的性能，因此， 默认情况下，Kryo 有50多个适用于各种JRE 类的默认序列化器 ([50+ default serializers](https://github.com/EsotericSoftware/kryo/blob/master/src/com/esotericsoftware/kryo/Kryo.java#L179))。也可以向其中再添加别的序列化器。

```
Kryo kryo = new Kryo();
kryo.setRegistrationRequired(false);
kryo.addDefaultSerializer(SomeClass.class, SomeSerializer.class);

Output output = ...
SomeClass object = ...
kryo.writeObject(output, object);
```

同时，若是一个类继承或实现某个已被注册的类时， 会创建一个该注册类对应的序列化器的实例。

默认序列化器是被排序了的，会首先匹配更契合的类， 若没有契合的类， 它们会按照被在列表中的顺序去和类做匹配。它们的顺序可以通过一些接口来改动。

如果默认序列化器中，没有能和类匹配上， 就会使用全局序列化器。 全局序列化器默认情况下是 [FieldSerializer](https://github.com/EsotericSoftware/kryo/blob/master/README.md#fieldserializer) ， 但也可以被替换。通常， 全局序列化器能处理很多种不同的类型。
```
Kryo kryo = new Kryo();
kryo.setDefaultSerializer(TaggedFieldSerializer.class);
kryo.register(SomeClass.class);
```
上面代码中， 若没有指定具体序列化器给 SomeClass ，就会使用TaggedFieldSerializer。

也可以使用注解的方式指定序列化器:
```
@DefaultSerializer(SomeClassSerializer.class)
public class SomeClass {
   // ...
}
```
为了获得最大的灵活性， 可以重写 Kryo 的 **getDefaultSerializer** 方法来实现特定的逻辑并实例化序列化器。

#### 6.2.1 Serializer factories

**addDefaultSerializer(Class, Class)** 方法不允许改动序列化器的配置。可以使用序列化器工厂(serializer factory)来代替一个具体的序列化类， 对每个序列化器来说， 对应的序列化器工厂允许创建并配置其实例。一般通用的序列化器都有对应的序列化器工厂， 通常调用 **getConfig** 方法来配置已被创建的序列化器。
```
Kryo kryo = new Kryo();
 
TaggedFieldSerializerFactory defaultFactory = new TaggedFieldSerializerFactory();
defaultFactory.getConfig().setReadUnknownTagData(true);
kryo.setDefaultSerializer(defaultFactory);

FieldSerializerFactory someClassFactory = new FieldSerializerFactory();
someClassFactory.getConfig().setFieldsCanBeNull(false);
kryo.register(SomeClass.class, someClassFactory);
```

序列化器工厂有一个 **isSupported(Class)** 方法，该方法允许它拒绝处理一个类，即使它在能和该类匹配上, 这允许工厂检查多个接口或实现其他逻辑。

### 6.3 Object creation

虽然有些序列化器只用于特定的类，但其它的序列化器可以序列化许多不同的类。 序列化器可以使用 Kryo 的 **newInstance(Class)** 方法创建任何类的实例。这是通过查找类的注册，然后使用注册的对象实例化器（ObjectInstantiator）来完成的。 实例化器可以在注册时被指定。
```
Registration registration = kryo.register(SomeClass.class);
registration.setInstantiator(new ObjectInstantiator<SomeClass>() {
   public SomeClass newInstance () {
      return new SomeClass("some constructor arguments", 1234);
   }
});
```
如果注册中没有实例化器， 调用 **newInstantiator**  方法能够创建一个。  也可以通过重写 **newInstantiator** 方法， 或通过 **InstantiatorStrategy** 提供自定义的创建对象的方式。



#### 6.3.1 InstantiatorStrategy 

Kryo 提供的默认实例化策略(DefaultInstantiatorStrategy)是使用 ReflectASM 调用一个无参构造函数来创建对象的。 如果无法创建成功， 就会使用反射调用无参构造函数。 若仍失败， 就抛出异常或是回退实例化策略。 反射使用 **setAccessible** 方法， 因此，私有无参构造函数是允许 Kryo 在不影响公共API的情况下创建类实例的好方法。

Kryo 推荐使用默认的实例化策略(DefaultInstantiatorStrategy) 来创建对象。 它像 Java 代码一样运行构造函数。另外，还可以使用语言外机制来创建对象。[Objenesis](http://objenesis.org/) 标准实例化策略(StdInstantiatorStrategy) 使用  JVM 特定的 API 来创建一个类的实例，而根本不需要调用任何构造函数。 但这样做是很危险的，因为大多数的类还是希望使用其构造函数来创建实例。绕过构造函数创建对象可能会使对象处于未初始化或无效的状态，而且类必须设计成以这种方式创建。 

通过配置，Kryo 可以先尝试使用 **DefaultInstantiatorStrategy**， 不行再使用 **StdInstantiatorStrategy**。
```
kryo.setInstantiatorStrategy(new DefaultInstantiatorStrategy(new StdInstantiatorStrategy()));
```
另外还有一个选择是 **SerializingInstantiatorStrategy**， 它使用 Java 内置序列化原理来创建一个实例。 想使用这个方法，必须实现 **java.io.Serializable** 接口，调用超类中的第一个无参构造函数。这个方法同样也是绕过了构造函数， 出于和 **StdInstantiatorStrategy** 相同的原因，这样做也是很危险的。
```
kryo.setInstantiatorStrategy(new DefaultInstantiatorStrategy(new SerializingInstantiatorStrategy()));
```


#### 6.3.2 Overriding create

另外，一些泛型序列化器提供了可重写的方法，用于为特定类型去自定义创建对象，而不是调用 Kryo 的 **newInstance**  方法。
```
kryo.register(SomeClass.class, new FieldSerializer(kryo, SomeClass.class) {
   protected T create (Kryo kryo, Input input, Class<? extends T> type) {
      return new SomeClass("some constructor arguments", 1234);
   }
});
```
一些序列化器提供了 **writeHeader** 方法，可以通过重写该方法，以在正确的时间写入 **create**方法中需要的数据。
```
static public class TreeMapSerializer extends MapSerializer<TreeMap> {
   protected void writeHeader (Kryo kryo, Output output, TreeMap map) {
      kryo.writeClassAndObject(output, map.comparator());
   }

   protected TreeMap create (Kryo kryo, Input input, Class<? extends TreeMap> type, int size) {
      return new TreeMap((Comparator)kryo.readClassAndObject(input));
   }
}
```
如果一个序列化器没有提供 **writeHeader** 方法， 那可以在 **write** 方法中写入 **create** 方法需要的数据。
```
static public class SomeClassSerializer extends FieldSerializer<SomeClass> {
   public SomeClassSerializer (Kryo kryo) {
      super(kryo, SomeClass.class);
   }
   public void write (Kryo kryo, Output output, SomeClass object) {
      output.writeInt(object.value);
   }
   protected SomeClass create (Kryo kryo, Input input, Class<? extends SomeClass> type) {
      return new SomeClass(input.readInt());
   }
}
```

### 6.4 Final classes

对于序列化器来说， 即使它知道某个值具体对应哪个类(比如知道一个字段是属于哪个类的)， 但如果这个类不是最终类，它仍会先写入这个类的ID (class ID)， 再写入值。 最终类是非多态的， 所以在被序列化时效率更高。 

Kryo 的 **isFinal**方法用于确定类是否是final。即使对于非final类型，也可以重写此方法以返回 true。例如，如果一个应用程序广泛使用数组列表(ArrayList)，但从不使用 ArrayList 子类，那么将ArrayList 作为 final 可以允许 FieldSerializer 为每个 ArrayList 字段节省1-2字节。

### 6.5 Closures(闭包)

Kryo 可以序列化实现 Java .io 的 Java 8+ 闭包。虽然可序列化，但有一些警告信息。在一个 JVM 上序列化的闭包可能无法在另一个 JVM 上反序列化。

Kryo 的 **isClosure** 方法用于确定类是否是闭包。如果是，那么就使用闭包序列化器(ClosureSerializer)。闭包用于查找类注册，而不是闭包的类。要序列化闭包，必须注册以下类: ClosureSerializer.Closure, SerializedLambda, Object[] 和 类。 此外，闭包的捕获类必须被注册。

```
kryo.register(Object[].class);
kryo.register(Class.class);
kryo.register(SerializedLambda.class);
kryo.register(ClosureSerializer.Closure.class, new ClosureSerializer());
kryo.register(CapturingClass.class);

Callable<Integer> closure1 = (Callable<Integer> & java.io.Serializable)( () -> 72363 );

Output output = new Output(1024, -1);
kryo.writeObject(output, closure1);

Input input = new Input(output.getBuffer(), 0, output.position());
Callable<Integer> closure2 = (Callable<Integer>)kryo.readObject(input, ClosureSerializer.Closure.class);
```


有一些方法([with some effort](https://ruediste.github.io/java,/kryo/2017/05/07/serializing-non-serializable-lambdas.html))，可以序列化没实现Serializable的闭包。

### 6.6 Compression and encryption (加密和解密)

 Kryo支持流，所以对所有序列化的字节使用压缩或加密很简单:

```
OutputStream outputStream = new DeflaterOutputStream(new FileOutputStream("file.bin"));
Output output = new Output(outputStream);
Kryo kryo = new Kryo();
kryo.writeObject(output, object);
output.close();
```
如果需要，序列化器可用于仅对对象图的字节子集压缩或加密字节。详例请参阅 DeflateSerializer 或 BlowfishSerializer。这些序列化器通过包装另一个序列化器来编码和解码字节。

&nbsp;

## 7. Implementing a serializer

---

Serializer 抽象类中定义了将对象转成字节数组 ，和把字节数组转成抽象类的方法。
```
public class ColorSerializer extends Serializer<Color> {
   public void write (Kryo kryo, Output output, Color color) {
      output.writeInt(color.getRGB());
   }

   public Color read (Kryo kryo, Input input, Class<? extends Color> type) {
      return new Color(input.readInt());
   }
}
```
Serializer 类中有两个必须被实现的方法。 **write** 方法用于将对象转成字节数组写入Output，**read** 方法用于创建一个新的对象实例，并从Input中读取数据填充实例。

### 7.1 Serializer references
当使用 Kryo 在序列化器 **read** 方法中读取嵌套对象时，如果嵌套对象可以引用父对象，则必须首先使用父对象调用 Kryo 的 **reference** 方法。如果嵌套对象不能引用父对象，或嵌套对象没有使用 Kryo，或没有使用引用，就没必要调用 **reference** 方法了。如果嵌套对象可以使用相同的序列化器，则序列化器必须是可重入的。
```
public SomeClass read (Kryo kryo, Input input, Class<? extends SomeClass> type) {
   SomeClass object = new SomeClass();
   kryo.reference(object);
   // Read objects that may reference the SomeClass instance.
   object.someField = kryo.readClassAndObject(input);
   return object;
}
```

#### 7.1.1 Nested serializers
序列化器不应该直接使用其它序列化器来赋值， 而应该用 Kryo 的 **read** 和 **write** 方法来代替。这样使得 Kryo 可以编排序列化来处理一些像引用, 空对象之类的情况。有时候，序列化器知道要为嵌套对象使用哪个序列化器。在这种情况下，它应该使用 Kryo 的读写方法，这些方法接受序列化器。

如果对象可能为 null:
```
Serializer serializer = ...
kryo.writeObjectOrNull(output, object, serializer);

SomeClass object = kryo.readObjectOrNull(input, SomeClass.class, serializer);
```
如果对象不可能为 null:
```
Serializer serializer = ...
kryo.writeObject(output, object, serializer);

SomeClass object = kryo.readObject(input, SomeClass.class, serializer);
```
在序列化期间，Kryo 的 **getDepth** 方法返回当前对象图的深度。

#### 7.1.2 KryoException
当序列化失败时， 可以抛出一个 KryoException， 其包含序列化追踪信息， 表明在对象图的哪个地方出现了异常。在使用嵌套序列化器时，可以捕获 KryoException 来添加序列化跟踪信息 。
```
Object object = ...
Field[] fields = ...
for (Field field : fields) {
   try {
      // Use other serializers to serialize each field.
   } catch (KryoException ex) {
      ex.addTrace(field.getName() + " (" + object.getClass().getName() + ")");
      throw ex;
   } catch (Throwable t) {
      KryoException ex = new KryoException(t);
      ex.addTrace(field.getName() + " (" + object.getClass().getName() + ")");
      throw ex;
   }
}
```
####  7.1.3 Stack size

Kryo 提供的序列化器在序列化嵌套对象时会调用栈。Kryo 最小化的调用栈， 但若对象图深度非常大， 仍会发生栈溢出。对于大多数的序列化库，包括 Java  内置的序列化机制，这都是一个很常见的问题。 可以使用 **-Xss** 来提高栈的大小， 但这个改变会应用到所有的线程上。 在 JVM 所有的线程上都使用较大的栈会占用大量的内存。
Kryo 的 **setMaxDepth** 方法可以用来限制对象图的最大深度，防止恶意数据造成栈溢出。

### 7.2 Accepting null

默认情况下，序列化器永远不会接收到 null，相反，Kryo 将根据需要编写一个字节来表示 null 或 not null。 如果一个序列化器想自己能通过处理 null 来提高效率， 可以调用 Serializer类 的 **setAcceptsNull(true)** 方法。当知道序列化器将处理的所有实例永远不会为空时，还可以使用这种方法来避免编写表示字节的 null。

### 7.3 Generics

Kryo 的 **getGenerics** 方法提供泛型类型信息，因此序列化器可以更有效。这通常用于避免在类型参数类为 final 时编写类。
如果类只有一个类型参数，则 **nextGenericClass** 方法返回类型参数类，若没有，则返回 null。在读取或写入任何嵌套对象之后必须调用 **popGenericType** 方法。有关示例，请参见CollectionSerializer。

```
public class SomeClass<T> {
   public T value;
}
public class SomeClassSerializer extends Serializer<SomeClass> {
   public void write (Kryo kryo, Output output, SomeClass object) {
      Class valueClass = kryo.getGenerics().nextGenericClass();

      if (valueClass != null && kryo.isFinal(valueClass)) {
         Serializer serializer = kryo.getSerializer(valueClass);
         kryo.writeObjectOrNull(output, object.value, serializer);
      } else
         kryo.writeClassAndObject(output, object.value);

      kryo.getGenerics().popGenericType();
   }

   public SomeClass read (Kryo kryo, Input input, Class<? extends SomeClass> type) {
      Class valueClass = kryo.getGenerics().nextGenericClass();

      SomeClass object = new SomeClass();
      kryo.reference(object);

      if (valueClass != null && kryo.isFinal(valueClass)) {
         Serializer serializer = kryo.getSerializer(valueClass);
         object.value = kryo.readObjectOrNull(input, valueClass, serializer);
      } else
         object.value = kryo.readClassAndObject(input);

      kryo.getGenerics().popGenericType();
      return object;
   }
}
```

对于具有多个类型参数的类，**nextGenericTypes** 方法返回一个泛型类型实例数组，并使用 **resolve** 方法获取每个泛型类型的类。在读取或写入任何嵌套对象之后，必须调用 **popGenericType** 方法。有关示例，请参见MapSerializer。
```
public class SomeClass<K, V> {
   public K key;
   public V value;
}
public class SomeClassSerializer extends Serializer<SomeClass> {
   public void write (Kryo kryo, Output output, SomeClass object) {
      Class keyClass = null, valueClass = null;
      GenericType[] genericTypes = kryo.getGenerics().nextGenericTypes();
      if (genericTypes != null) {
         keyClass = genericTypes[0].resolve(kryo.getGenerics());
         valueClass = genericTypes[1].resolve(kryo.getGenerics());
      }

      if (keyClass != null && kryo.isFinal(keyClass)) {
         Serializer serializer = kryo.getSerializer(keyClass);
         kryo.writeObjectOrNull(output, object.key, serializer);
      } else
         kryo.writeClassAndObject(output, object.key);

      if (valueClass != null && kryo.isFinal(valueClass)) {
         Serializer serializer = kryo.getSerializer(valueClass);
         kryo.writeObjectOrNull(output, object.value, serializer);
      } else
         kryo.writeClassAndObject(output, object.value);

      kryo.getGenerics().popGenericType();
   }

   public SomeClass read (Kryo kryo, Input input, Class<? extends SomeClass> type) {
      Class keyClass = null, valueClass = null;
      GenericType[] genericTypes = kryo.getGenerics().nextGenericTypes();
      if (genericTypes != null) {
         keyClass = genericTypes[0].resolve(kryo.getGenerics());
         valueClass = genericTypes[1].resolve(kryo.getGenerics());
      }

      SomeClass object = new SomeClass();
      kryo.reference(object);

      if (keyClass != null && kryo.isFinal(keyClass)) {
         Serializer serializer = kryo.getSerializer(keyClass);
         object.key = kryo.readObjectOrNull(input, keyClass, serializer);
      } else
         object.key = kryo.readClassAndObject(input);

      if (valueClass != null && kryo.isFinal(valueClass)) {
         Serializer serializer = kryo.getSerializer(valueClass);
         object.value = kryo.readObjectOrNull(input, valueClass, serializer);
      } else
         object.value = kryo.readClassAndObject(input);

      kryo.getGenerics().popGenericType();
      return object;
   }
}
```

对于在对象图中为嵌套对象传递类型参数信息的序列化器(一些高级的用法)，第一个泛型层次结构用于存储类的类型参数。在序列化期间，Generics 的 **pushTypeVariables** 方法在解析泛型类型(如果有的话)之前被调用。如果返回>0，后面必须跟着 Generics 的 **popTypeVariables** 方法。有关示例，请参见 FieldSerializer。
```
public class SomeClass<T> {
   T value;
   List<T> list;
}
public class SomeClassSerializer extends Serializer<SomeClass> {
   private final GenericsHierarchy genericsHierarchy;

   public SomeClassSerializer () {
      genericsHierarchy = new GenericsHierarchy(SomeClass.class);
   }

   public void write (Kryo kryo, Output output, SomeClass object) {
      Class valueClass = null;
      Generics generics = kryo.getGenerics();
      int pop = 0;
      GenericType[] genericTypes = generics.nextGenericTypes();
      if (genericTypes != null) {
         pop = generics.pushTypeVariables(genericsHierarchy, genericTypes);
         valueClass = genericTypes[0].resolve(generics);
      }

      if (valueClass != null && kryo.isFinal(valueClass)) {
         Serializer serializer = kryo.getSerializer(valueClass);
         kryo.writeObjectOrNull(output, object.value, serializer);
      } else
         kryo.writeClassAndObject(output, object.value);

      kryo.writeClassAndObject(output, object.list);

      if (pop > 0) generics.popTypeVariables(pop);
      generics.popGenericType();
   }

   public SomeClass read (Kryo kryo, Input input, Class<? extends SomeClass> type) {
      Class valueClass = null;
      Generics generics = kryo.getGenerics();
      int pop = 0;
      GenericType[] genericTypes = generics.nextGenericTypes();
      if (genericTypes != null) {
         pop = generics.pushTypeVariables(genericsHierarchy, genericTypes);
         valueClass = genericTypes[0].resolve(generics);
      }

      SomeClass object = new SomeClass();
      kryo.reference(object);

      if (valueClass != null && kryo.isFinal(valueClass)) {
         Serializer serializer = kryo.getSerializer(valueClass);
         object.value = kryo.readObjectOrNull(input, valueClass, serializer);
      } else
         object.value = kryo.readClassAndObject(input);

      object.list = (List)kryo.readClassAndObject(input);

      if (pop > 0) generics.popTypeVariables(pop);
      generics.popGenericType();
      return object;
   }
}
```

### 7.4 KryoSerializable

一个类可以通过实现 KryoSerializable 接口(类似于java.io.Externalizable) 来实现自己的序列化，来替代使用序列化器。
```
public class SomeClass implements KryoSerializable {
   private int value;
   public void write (Kryo kryo, Output output) {
      output.writeInt(value, false);
   }
   public void read (Kryo kryo, Input input) {
      value = input.readInt(false);
   }
}
```

显然，在调用 **read** 方法之前必须已经创建了实例，因此类无法控制自己的创建。实现了 KryoSerializable 接口的类将使用缺省序列化器KryoSerializableSerializer，它使用 Kryo 的 **newInstance** 创建一个新实例。这样就能很简单的编写自己的序列化程序来定制流程、在序列化之前或之后调用方法，等等。

### 7.5 Serializer copying

序列化器仅在 **copy** 方法被重写时支持复制。与 Serializer 类的 **read** 方法类似，此方法包含创建和配置副本的逻辑。与 **read** 方法一样，如果任何子对象都可以引用父对象，那么在使用 Kryo 复制子对象之前必须调用 Kryo 的 **reference** 方法。
```
class SomeClassSerializer extends Serializer<SomeClass> {
   public SomeClass copy (Kryo kryo, SomeClass original) {
      SomeClass copy = new SomeClass();
      kryo.reference(copy);
      copy.intValue = original.intValue;
      copy.object = kryo.copy(original.object);
      return copy;
   }
}
```

#### 7.5.1 KryoCopyable

一个类可以通过实现 KryoCopyable  接口来实现自己的复制， 替代使用序列化器实现。
```
public class SomeClass implements KryoCopyable<SomeClass> {
   public SomeClass copy (Kryo kryo) {
      SomeClass copy = new SomeClass();
      kryo.reference(copy);
      copy.intValue = intValue;
      copy.object = kryo.copy(object);
      return copy;
   }
}
```

#### 7.5.2 Immutable serializers

Serializer 类的 **setimmutable(true)** 方法可以在类型不可变时使用。在这种情况下，不需要实现 Serializer 类的 **copy** 方法——默认的 copy 方法实现将返回原始对象。

&nbsp;
## 8. Kryo versioning and upgrading

---
Kryo 的版本号遵循下列规则：

1.  当序列化的兼容性被破坏时， 会增加主版本。 即使用旧版本序列化得到的数据，可能无法被新版本反序列化。
2.  如果文档化的公共API的二进制或源代码兼容性被破坏，则会增加次要版本。为了避免在只有很用户受到影响时增加版本，如果在很少使用或不打算用于一般用途的公共类中发生了一些较小的破坏，则这些破坏是被允许的。

升级任何依赖项都是一个重要事件，但是序列化库比大多数依赖项更容易崩溃。升级Kryo时，请检查版本差异，并在您自己的应用程序中彻底测试新版本。我们试着让它尽可能的安全简单。

  在开发时，将测试不同二进制格式和默认序列化器的序列化兼容性。

  在开发时，使用 [clirr](http://www.mojohaus.org/clirr-maven-plugin/)  跟踪二进制和源代码兼容性。

  对于每个版本，都会提供一个变更日志，其中还包含一个部分，报告序列化、二进制和源代码兼容性。

  为了报告二进制和源代码兼容性，使用了 [japi-compliance-checker](https://github.com/lvc/japi-compliance-checker/)  。

&nbsp;
## 9. Interoperability

----
默认情况下提供的Kryo序列化器假定Java将用于反序列化，因此它们不会显式定义所编写的格式。序列化器可以使用更容易被其他语言读取的标准化格式编写，但默认情况下不提供这种格式。

&nbsp;
## 10. Compatibility

---
对于某些需要，例如序列化字节的长期存储，序列化如何处理对类的更改可能非常重要。这被称为向前兼容性(读取由新类序列化的字节)和向后兼容性(读取由旧类序列化的字节)。Kryo提供了一些通用的序列化器，它们采用不同的方法来处理兼容性。可以很容易地为向前和向后兼容性开发其他序列化器，比如使用外部手写模式的序列化器。

&nbsp;
## 11. Serializers

---
Kryo提供了许多具有各种配置选项和兼容性级别的序列化器。除了下列介绍的系列化器，你可以在本项目的姐妹项目[kryo-serializers](https://github.com/magro/kryo-serializers) 中， 找到更多的序列化器， 但这些序列化器提供的是私有的API， 或者其并不是在所有类型的JVM上， 都是安全的。 除此以外， 你还可以在[links section](https://github.com/EsotericSoftware/kryo/blob/master/README.md#links).这个链接上， 找到更多的序列化器。

### 11.1 FieldSerializer

FieldSerializer 对每个非transient 字段都可以进行序列化 (译者注：transient为Java关键字，它只能用来修饰变量，不能用来修饰类和变量；被transient修饰的变量不能被序列化)。 无需任何配置的情况下，就对POJO和其它许多类进行序列化。默认情况下，所有的非public字段都会被读写，所以对每个要被序列化的类进行进行评估就显得很重要。 如果字段是public的话， 序列化的速度就会快很多。

#### 11.1.1 CachedField settings
#### 11.1.2 FieldSerializer annotations


### 11.2 VersionFieldSerializer

### 11.3 TaggedFieldSerializer

### 11.4 CompatibleFieldSerializer
### 11.5 BeanSerializer

### 11.6 CollectionSerializer

### 11.7 MapSerializer

### 11.8 JavaSerializer and ExternalizableSerializer


&nbsp;

## 12. Logging

---


&nbsp;
## 13. Thread safety

---


### 13.1 Pooling

&nbsp;

## 14. Benchmarks

---

&nbsp;

## 15. Links

---

### 15.1 Projects using Kryo

### 15.2 Scala

### 15.3 Clojure

### 15.4 Objective-C


&nbsp;
&nbsp;





















.... 未完，持续更新中....








   






