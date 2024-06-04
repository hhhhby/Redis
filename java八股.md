## java

### java面向对象
 - HashMap
 - **JVM**：运行JAVA字节码的虚拟机
   - 不同的系统，其实现不同，目的是为了对于相同的字节码，不同系统的JVM实现会给出相同的结果
   - 
 - **JDK（Java Development Kit）包含JRE，JVM**：它是功能齐全的 Java SDK，是提供给开发者使用，能够创建和编译 Java 程序的**开发套件**.
 - **JRE（Java Runtime Environment）包含JVM**：Java 运行时**环境**。仅包含 Java 应用程序的运行时**环境**和**必要的类库**。
   <img width="629" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/ee2ed250-bdd5-4d08-b7d2-e9405b0e96a3">
 - **字节码**：JVM能够理解的代码就叫字节码（.class文件）。
   - 它不面向任何特定的处理器，只面向虚拟机。
   - Java 语言通过字节码的方式，在一定程度上**解决了传统解释型语言执行效率低的问题**，同时又保留了**解释型语言可移植**的特点。
   - Java 程序**无须重新编译**便可在多种不同操作系统的计算机上运行
 - Java程序转变为机器代码的过程
   - 1.创建.java文件
   - 2.经过javac编译为.class文件（字节码）
   - 3.JVM加载字节码文件，通过解释器逐行解释执行，此时会判断当前代码是否是热点代码（经常被调用的代码）
   - 4.1 如果是热点代码，则通过JIT（Just in Time Compilation）编译器编译，保存热点代码对应的机器码（仅执行一次，下次直接使用）
   - 4.2 如果不是，则正常通过解释器来解释。
   - 5.转换为机器码。
<img width="754" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/2ab21abb-bcbf-4ca9-a1cd-447ac753bd21">

 - 编译型语言：通过编译器将源代码**一次性**翻译成可被该平台执行的机器码
   - 执行速度快，开发效率低
 - 解释型语言：通过解释器一句一句将代码解释为机器代码后再执行
   - 执行速度慢，开发效率高

 - Java语言“编译与解释并存”的原因：
   - Java语句同时具备编译器与解释器的特征
   - Java 程序要经过先编译，后解释两个步骤，由 Java 编写的程序需要先经过编译步骤，生成字节码（.class 文件），这种字节码必须由 Java 解释器来解释执行。

 - **Java与C++相同点与不同点**
   - 相同点：都是面向对象语言，都支持封装、继承和多态
   - 不同点：
     - Java**不提供指针**来访问内存，程序内存更加安全
     - Java**类是单继承，接口是多继承**，C++都是多继承
     - Java有**自动内存管理垃圾回收机制**，不需要程序员手动释放无用内存
     - **C++同时支持方法重载和操作符重载**，但是**Java只支持方法重载**

 - 注释：**代码说明书**（单行注释//、文档注释/* *** */）
   - 代码的注释不是越详细越好。实际上好的代码**本身就是注释**，我们要尽量规范和美化自己的代码来减少不必要的注释。若编程语言足够有表达力，就不需要注释，尽量通过代码来阐述。

 - 标识符和关键字
   -

 - 数据类型
   - 基本类型：byte(1字节) boolean（） short（2） int（4） long（8） char（2） float（4） double（8）
   - 包装类型：Byte      Boolean      Short     Integer   Long     Character  Float     Double
 - 基本类型和包装类型
   - **用途**：基本类型用来定义一些常量和局部变量，包装类型用来在方法参数、对象属性中定义变量
   - **存储方式**：基本类型的局部变量存放在Java虚拟机**栈**中的局部变量表中，基本数据类型的成员变量存放在JVM的**堆**中。包装类型属于对象类型，都存在**堆中**。
   - **占用空间**：基本类型占用的空间非常小
   - **默认值**：包装类型不赋值就是null，基本类型有默认值且不是null
   - **比较方式**：基本类型 “==” 比较的是值，包装类型“==” 比较的是对象的内存地址，**所有整型包装类对象之间值的比较，全部使用“equals()”方法**
 - 包装类型的缓存机制
   - Java 基本数据类型的包装类型的大部分都用到了缓存机制来提升性能
   - Byte,Short,Integer,Long 这 4 种包装类默认创建了数值 **[-128，127]** 的相应类型的缓存数据，Character 创建了数值在 **[0,127]** 范围的缓存数据，Boolean 直接返回 **True** or **False**
   - 如果超出对应范围仍然会去创建新的对象，缓存的范围区间的大小只是在**性能和资源**之间的权衡。
   - 两种浮点数类型的包装类 Float,Double 并没有实现缓存机制。
 - 自动装箱与拆箱
   - 装箱：将基本类型用它们对应的引用类型包装起来；
   - 拆箱：将包装类型转换为基本数据类型；
   ```
   Integer i = 10; ==>Integer i = Integer.valueof(10);   //装箱
   int n = i;     ==> int n = i.intValue();        //拆箱
   ```




 - 面向对象和面向过程的区别
   - 面向过程：把解决问题的过程拆成一个个方法，通过一个个方法的执行解决问题
   - 面向对象：先抽象出对象，然后用对象执行方法的方式解决问题。面向对象开发的程序一般更易维护、易复用、易扩展
  
 - 面向对象三大特征
   - **封装**。把一个对象的状态信息（也就是属性）隐藏在对象内部，不允许外部对象直接访问对象的内部信息。但是可以提供一些可以被外界访问的方法来操作属性。
   - **继承**。使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。
     - 子类拥有父类对象所有的属性和方法（包括私有属性和私有方法），但是父类中的私有属性和方法子类是无法访问，**只是拥有**。
     - 子类可以拥有自己属性和方法，即子类可以对父类进行扩展
     - 子类可以用自己的方式实现父类的方法
   - **多态**。一个对象具有多种的状态，具体表现为父类的引用指向子类的实例
     - 对象类型和引用类型之间具有继承（类）/实现（接口）的关系
     - 引用类型变量发出的方法调用的到底是哪个类中的方法，必须在程序运行期间才能确定
     - 多态不能调用“只在子类存在但在父类不存在”的方法
     - 如果子类重写了父类的方法，真正执行的是子类重写的方法，如果子类没有重写父类的方法，执行的是父类的方法

### 接口与抽象类
   - 共同点
     - 都不能被**实例化**
     - 都可以包含**抽象方法**
     - 都可以有**默认实现的方法**
   - 不同点
     - 接口主要用于**对类的行为进行约束**，你实现了某个接口就具有了对应的行为。抽象类主要用于**代码复用**，强调的是**所属关系**
     - 一个类只能**继承一个抽象类**，但可以**实现多个接口**
     - 接口内的成员变量只能是 public static final 类型的，不能被修改且必须有初始值，而抽象类的成员变量默认default，可以在子类中被重新定义，也可以被重新赋值。
### 深拷贝与浅拷贝、引用拷贝
   - 深拷贝：直接复制整个对象，包括对象内部的对象
   - 浅拷贝：在堆上新建一个对象，如果原对象内部的属性是引用类型的话，浅拷贝会直接复制内部对象的引用地址（拷贝对象与原对象共用一个内部对象）
   - 引用拷贝：两个不同的引用同时指向同一个对象
### String、StringBuffer、StringBuilder
   - String：对象不可变、线程安全（常量），对其进行修改时都会创建一个新的对象，并改变对象引用
     - 不可变原因↓
     - 1.保存字符串的数组被 final 修饰且为私有的，并且String 类没有提供/暴露修改这个字符串的方法（**只能访问，没有提供对其进行修改的方法**）。
     - 2.String 类被 final 修饰导致其不能被继承，进而避免了子类破坏 String 不可变
   - StringBuffer：可变、线程安全（加了同步锁），对对象本身进行修改
   - StringBulider：可变、非线程安全，对对象本身进行修改
   - 在 **Java 9** 之后，String、StringBuilder 与 StringBuffer 的实现改用 **byte** 数组存储字符串（之前使用**char**，这样更加节省空间）。

 - 字符串常量池：JVM为了提升性能和减少内存消耗针对字符串（String类）专门开辟的一块区域，主要目的是为了避免字符串的重复创建
 - String.intern()：将指定的字符串对象的引用保存在字符串常量池

### 异常
   <img width="767" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/ac9b27b2-1e97-4bdd-b0a2-af1a5b980e67">
 - Exception与Error
   - Exception：程序本身可以处理，可以通过try-catch
   - Error：程序无法处理的异常

### 泛型
“泛型”意味着编写的代码可以被**不同类型**的对象所**重用**。
   - Java 泛型（Generics） 是 JDK 5 中引入的一个新特性。使用泛型参数，可以增强代码的可读性以及稳定性
   - 编译器可以对泛型参数进行检测，并且通过泛型参数可以指定传入的对象类型。

 - 泛型的使用方式
   - 泛型类
   ```
   public class Generic<T>{

       private T key;
   
       public Generic(T key){
           this.key = key;
       }

       public T getKey(){
           return key;
       }
   }
   ```
     - 实例化泛型类  
       ``Generic<Integer> genericInteger = new Generic<Integer>(123456);``
   - 泛型接口
   ```
   public interface Generator<T>{
       public T method();
   }
   ```
     - 实现泛型接口，不指定类型：
       ```
       class GeneratorImpl<T> implements Generator<T>{
           @override
           public T method(){
               return null;
           }
       }
       ```
     - 实现泛型接口，指定类型
       ```
       class GeneratorImpl<T> implements Generator<String>{
           @override
           public String method(){
               return "hello";
           }
       }
       ```
   - 泛型方法
     ```
     public static <E> void printArray(E[] inputArray){
         for( E element: inputArray){
             System.out.printf("%s",element);
         }
         System.out.println();
     }
     ```
     - 使用方法
       ```
       Integer[] intArray = {1,2,3};
       String[] stringArray = {"Hello","World"};
       printArray(intArray);
       printArray(stringArray);
       ```
   - 在 java 中泛型只是一个占位符，必须在传递类型后才能使用
### 反射
   - 反射赋予了我们在运行时分析类以及执行类中方法的能力
   - 通过反射，可以获得任意一个类的所有属性和方法，还可以调用这些方法和属性
   - 实现Java反射的类
     - 1.Class：表示正在运行的Java应用程序中的**类和接口**（所有获取对象的信息都需要Class类来实现）
     - 2.Field：提供有关类和接口的**属性**信息，以及对它的动态访问权限
     - 3.Constructor：提供关于类的单个**构造方法**的信息以及它的访问权限
     - 4.Method：提供类或接口中某个**方法的**信息
   - 优点
     - 1.能够在运行时动态获取类的实例，**提高灵活性**；
     - 2.与动态编译结合
   - 缺点
     - 1.使用反射**性能较低**，需要解析字节码，将内存中的对象进行解析
       - 解决方案1：通过**setAccessible(true)关闭JDK的安全检查**来提升反射速度
       - 解决方案2：多次创建一个类的实例时，有**缓存**会快很多
       - 解决方案3：**ReflectASM**工具类，通过**字节码生成的方式**加快反射速度。
     - 2.**相对不安全，破坏了封装性**（通过反射，类中的私有属性和方法都能调用）
### 代理模式
  - 代理模式
    - 使用代理对象来代替对真是对象的访问，这样都可以在不修改原目标对象的前提下，提供额外的功能操作扩展目标对象的功能。
    - 作用：**扩展目标对象的功能**，比如说在目标对象的某个方法执行前后你可以增加一些自定义的操作。
    - 优点
      - 1.代理模式能够协调调用者和被调用者，在一定程度上**降低了系统的耦合度**。
      - 2.可以灵活地**隐藏被代理对象的部分功能和服务**，也**增加了额外的功能和服务**。
    - 缺点
      - 1.性能没有直接调用高
      - 2.提高了代码的复杂度
  - 静态代理
    - 目标对象的每个方法的增强都是**手动完成**的，
    - 非常不灵活（接口一旦新增加方法，目标对象和代理对象都要进行修改）
    - 麻烦，需要对每个目标类都单独写一个代理类
  - 动态代理
    - 相对于静态代理来说，**动态更加灵活**，我们不需要针对每个目标类都单独创建一个代理类，并且也不需要我们必须实现接口，我们可以**直接代理实现类**
    - 从JVM角度来说，动态代理是**在运行时动态生成类字节码，并加载到JVM中**的
    - Spring AOP、RPC 框架的实现都依赖了动态代理

  - JDK动态代理机制
    - 核心：一个类（Proxy类）、一个接口（InvocationHandler接口）
    - Proxy类
      - Proxy类中使用频率最高的方法是：newProxyinstance()，这个方法主要用来**生成一个代理对象**
      ```
      public static Object newProxyInstance(ClassLoader loader,
                                            Class<?>[] interfaces,
                                            InvocationHandler h)
          throws IllegalArgumentException
      {
        ......
      }

      ```
      - 参数1：loader，类加载器，用于加载代理对象
      - 参数2：interfaces，被代理类实现的一些接口；
      - 参数3：h，实现了InvocationHandler接口的对象
    - InvocationHandler接口
      - 要实现动态代理，还需要实现InvocationHandler接口来自定义处理逻辑。
      ```
      public interface InvocationHandler {
      
          /**
           * 当你使用代理对象调用方法的时候实际会调用到这个方法
           */
          public Object invoke(Object proxy, Method method, Object[] args)
              throws Throwable;
      }

      ```
      - 参数1：proxy，动态生成的代理类
      - 参数2：method，与代理类对象调用的方法相对应
      - 参数3：args，当前method方法的参数
    - 缺点：**只能代理实现了接口的类**
 - CGLIB动态代理机制
   - 在运行时可以对**字节码进行修改和动态生成**
   - 通过**继承**方式实现代理
   - 一个接口（MethodInterceptor接口）和一个类（Enhancer类）
     - MethodInterceptor接口
       - 需要自定义MethodInterceptor 并重写intercept方法，intercept用于拦截增强被代理类的方法
       ```
       public interface MethodInterceptor
       extends Callback{
           // 拦截被代理类中的方法
           public Object intercept(Object obj, java.lang.reflect.Method method, Object[] args,MethodProxy proxy) throws Throwable;
       }
       ```
       - 参数1：obj，被代理的对象（需要增强的对象）
       - 参数2：method，被拦截的方法（需要增强的方法）
       - 参数3：args，方法入参
       - 参数4：proxy，用于调用原始方法
 - JDK动态代理与CGLIB动态代理对比
   - JDK动态代理只能代理**实现了接口的类**或者**直接代理接口**，而CGLIB可以代理**未实现任何接口的类**。另外， CGLIB 动态代理是通过生成一个被代理类的子类来拦截被代理类的方法调用，因此不能代理声明为 final 类型的类和方法。
   - 就二者的**效率**来说，**JDK动态代理更优秀**，随着JDK版本的升级，这个优势更加明显
 - 静态代理和动态代理的对比
   - **灵活性**：动态代理更加灵活，不需要必须实现接口，可以直接代理实现类，并且可以不需要针对每个目标类都创建一个代理类。另外，静态代理中，接口一旦新增加方法，目标对象和代理对象都要进行修改
   - **JVM层面**：静态代理**在编译时**就将接口、实现类、代理类这些都变成了一个个实际的class文件，而动态代理是**在运行时**动态生成类字节码，并加载到JVM中的。
### JAVA集合
  Java集合，也叫做容器，主要是由两大接口派生而来
  - Collection接口：主要用于存放单一元素
    - 子接口1：List，存储的元素都是有序的
      - ArrayList与Array(数组)的区别
        - 1.ArrayList可以动态扩容，Array在创建之后长度不能被改变
        - 2.ArrayList可以使用泛型来保证类型安全，Array不可以
        - 3.ArrayList只能存储对象（在存储基本类型时，需要使用其对应的包装类Integer、Double等），Array既可以存储基本类型数据，也可以存储对象
        - 4.ArrayList支持插入、删除、遍历等常见操作，并且提供了丰富的API操作方法，比如add()，remove（）等。Array只是一个固定长度的数组，只能按照其下标来访问元素，不具备动态添加、删除元素的能力。
        - 5.ArrayList创建时不需要指定大小，Array需要指定大小。
    - 子接口2：Set，存储的元素不重复
    - 子接口3：Queue，按特定的排队规则来确定先后顺序，存储的元素是有序的，可重复的
  - Map接口：主要存放键值对，key是无序的、不可重复的，每个键最多映射到一个值
    - HashMap：
      - JDK1.8之前由数组+链表组成，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）
      - 1.8之后再解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间
    - LinkedHashMap：
      - 继承自HashMap,由数组+链表或数组+红黑树组成
      - 在HashMap的基础上增加了一条双向链表，使得上面的结构可以保持键值对的插入顺序
      - 通过对链表进行相应的操作，实现了**访问顺序**相关逻辑
    - HashTable：数组+链表，数组是主体，链表则是为了解决哈希冲突而存在（与HashMap有啥区别？）。
      - 与HashMap区别
        - 1.两者父类不同：HashMap是继承自AbstractMap类，而Hashtable是继承自Dictionary类。不过它们都实现了同时实现了map、Cloneable(可复制)、Serializable(可序列化)这三个接口。
        - 2.对外提供的接口不同：Hashtable比HashMap多提供了elments() 和contains() 两个方法。 elments() 方法继承自 Hashtable的父类Dictionnary。elements() 方法用于返回此Hashtable中的value的枚举。contains()方法判断该Hashtable是否包含传入的value。它的作用与containsValue()一致。事实 上，contansValue() 就只是调用了一下contains() 方法。
        - 3.对null的支持不同：HashTable：key和value都不能为null，HashMap：key可以为null，但是这样的key只能有一个，因为必须保证key的唯一性;可以有多个 key值对应的value为null
        - 4.安全性不同：HashMap是非线程安全的，HashTable是线程安全的。
        - 5.初始容量大小和每次扩充容量大小不同
        - 6.计算hash值的方法不同

## Redis
 - 缓存穿透（一直访问缓存和数据库都不存在的数据）：
   
 - 缓存击穿（热点key突然失效）：
   
 - 缓存雪崩（同一时段大量key同时失效）：
   
 - 跳表
   - 原理
   - 应用
 
