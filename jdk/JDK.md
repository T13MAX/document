# JDK

[TOC]

## java.lang----------------------------------------------------------------------------------------------------------------

2022-3-30  ~ 2022-4-

### java.lang.annotation

```java
Annotation
	使用@interface自定义注解时，自动继承了java.lang.annotation.Annotation接口
	annotationType()获取注解本身 getClass()会返回代理类
    hashCode() equals() toString() 抽象方法
    
@Documented
    生成java doc文档
    
@Inherited
    声明在类上 被子类继承 方法字段则无效
    
@Native
    表示这个变量可以被本地代码引用，常常被代码生成工具使用
    
@Repeatable
    重复用 写多个 最后会把多个变成一个父类型
    
@Retention
    何时生效 RetentionPolicy枚举
    
@Target
	作用目标 ElementType枚举
    
ElementType
    类接口枚举 方法 字段 构造方法 仅接口 包 等等
    
RetentionPolicy
    SOURCE 被编译器丢弃 也就是只在源码阶段生效 
    CLASS 字节码文件有 但是JVM不管 
    RUNTIME 运行时
    
```



### java.lang.invoke

```java
```



### java.lang.reflect

```java
Array
	数组类 一些静态方法 一些本地方法(set类型 get类型) 分配数组空间  
    
AccessibleObject
	Field Method Constructor的父类 提供访问控制的权限检查 ReflectPermission
    
Executable
	Method和Constructor的通用功能父类
    
Member
	字段方法的标识信息接口
    
Modifier
    工具类 private public 等等 包括volatile abstract final synchronize
    
Constructor
	private Class<T>            clazz;	//方法所属类
    private int                 slot;
    private Class<?>[]          parameterTypes;	//参数类型列表
    private Class<?>[]          exceptionTypes;	//异常列表
    private int                 modifiers;		//修饰符
    private transient String    signature;    // Generics and annotations support
    private transient ConstructorRepository genericInfo;    // generic info repository; lazily initialized
    private byte[]              annotations;	//方法上的注解
    private byte[]              parameterAnnotations;	//参数上的注解 Annotation[][] getParameterAnnotations() 返回二维数组
	public T newInstance(Object ... initargs)
        
Field
    private Class<?>            type;	//字段类型
	private FieldAccessor fieldAccessor;	//没被重写的 字段访问器
	private FieldAccessor overrideFieldAccessor;	//重写的 字段访问器
	private Field               root;
	set类型 get类型方法
        
Method
    同Constructor
    private String              name;	//方法名
    private Class<?>            returnType;	//返回类型
	private byte[]              annotationDefault;	//默认注解
	private volatile MethodAccessor methodAccessor;	//invoke用的 方法访问器
	private Method              root;	//

Proxy
    提供动态代理的静态方法
    public static Class<?> getProxyClass(ClassLoader loader,Class<?>... interfaces)
    public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces,InvocationHandler h)
    public static boolean isProxyClass(Class<?> cl) 
    public static InvocationHandler getInvocationHandler(Object proxy)
    
```



### java.lang.ref

```java
Finalizer
    虚拟机用来存储重写了finalize方法的实例
    
ReferenceQueue
    引用队列，在检测到适当的可到达性更改后，垃圾回收器将已注册的引用对象添加到该队列中
    
Reference
    各种引用类的抽象父类 引用对象与垃圾收集器密切合作
    
SoftReference
    static private long clock;	//由垃圾回收器更新
	private long timestamp;	//调用get方法更新

WeakReference
    
PhantomReference
	比Java终结机制更灵活的方式调度事前清理操作
    
```



### java.lang.management

```java

```



### java.lang.instrument

```java

```



### java.lang

````java
Object
	private static native void registerNatives();	//注册的方法就是除了registerNatives()方法以外的所有本地方法 hashCode clone
    static {
        registerNatives();
    }
	public final native Class<?> getClass();
	public native int hashCode();
	public boolean equals(Object obj)
	protected native Object clone()
	public String toString()
	public final native void notify()
    public final native void notifyAll()
    public final void wait()	//两个重载
    protected void finalize()
        
AutoCloseable
    实现AutoCloseable接口 重写close方法 try(){}

Math
	工具类 各种静态方法 各种函数 sin log 等 max min
    
Number
    所有数字类的抽象父类 类型的Value intValue longValue等
    实现了序列化接口 所以数字类就只实现Comparable Boolean还要自己实现序列化接口
    
Boolean 
    implements Serializable,Comparable<Boolean>
    private final boolean value;
	public static final Boolean TRUE = new Boolean(true);
	public static final Boolean FALSE = new Boolean(false);
	public static final Class<Boolean> TYPE = (Class<Boolean>) Class.getPrimitiveClass("boolean");
    valueOf(String s)调用parseBoolean(String s)调用String.equalsIgnoreCase(String anotherString)
        
Byte
    public static final byte   MIN_VALUE = -128;
	public static final byte   MAX_VALUE = 127;
	ByteCache	static final Byte cache[] = new Byte[-(-128) + 127 + 1];
	public static final int SIZE = 8;	//占多少位
	public static final int BYTES = SIZE / Byte.SIZE;	//占多少字节

Double
    public static final double POSITIVE_INFINITY = 1.0 / 0.0;
	public static final double NEGATIVE_INFINITY = -1.0 / 0.0;
	public static final double NaN = 0.0d / 0.0;
	public static final double MAX_VALUE = 0x1.fffffffffffffP+1023; // 1.7976931348623157e+308
	public static final double MIN_NORMAL = 0x1.0p-1022; // 2.2250738585072014E-308
	public static final double MIN_VALUE = 0x0.0000000000001P-1022; // 4.9e-324
	public static final int MAX_EXPONENT = 1023;
	public static final int MIN_EXPONENT = -1022;
	public static final int SIZE = 64;
	public static String toHexString(double d)	//16进制字符串
    public static boolean isNaN(double v) {
        return (v != v);
    }
	min() max() sum()
        
Float
    public static final float MAX_VALUE = 0x1.fffffeP+127f; // 3.4028235e+38f
	public static final float MIN_NORMAL = 0x1.0p-126f; // 1.17549435E-38f
	public static final float MIN_VALUE = 0x0.000002P-126f; // 1.4e-45f
	public static final int MAX_EXPONENT = 127;
	public static final int MIN_EXPONENT = -126;
	public static final int SIZE = 32;

Short
    public static short reverseBytes(short i) {
        return (short) (((i & 0xFF00) >> 8) | (i << 8));	//补码
    }
	public static int toUnsignedInt(short x) {
        return ((int) x) & 0xffff;	//无符号int值
    }
	public static long toUnsignedLong(short x) {
        return ((long) x) & 0xffffL;	//无符号long值
    }

Integer
    final static char[] digits={26英文字母+0-9}
	public static String toString(int i, int radix) //数字,进制 转换为某进制的字符串 最大36进制 看digits
    toString()->getChars(int i, int index, char[] buf) final static char [] DigitOnes 字符串也是字符拼的
    private static String toUnsignedString0(int val, int shift)	//转成某进制的字符串 1->二进制 2->四进制 3->八进制
    
    待续...
        
Long
    略
        
Character
    public static final int MIN_RADIX = 2; //最小二进制
	public static final int MAX_RADIX = 36;	//最大36进制
	public static final char MIN_VALUE = '\u0000';
	public static final char MAX_VALUE = '\uFFFF';  
	Unicode UTF-16相关的东东
    待续... 
        
Class
    implements java.io.Serializable,GenericDeclaration,Type,AnnotatedElement
    private static final int ANNOTATION= 0x00002000;	//注解
    private static final int ENUM      = 0x00004000;	//枚举
    private static final int SYNTHETIC = 0x00001000;	//
    private Class() {}	//只有JVM可以创建Class对象
	this.name = name = getName0();
	public String toGenericString()	//包括有关修饰符和类型参数的信息
    forName() 重载 是否初始化 传类加载器
    newInstance()	//校验条件 cachedConstructor构造器 newInstanceCallerCache
    对象是类的实例 是否为数组 是否为接口 是否基本类型 获取父类 返回数组类型 getModifiers()修饰符 Signers的getset 的本地方法
    是否注解 是否合成类(由编译器引入的字段、方法、类或其他结构，主要用于JVM内部使用，为了遵循某些规范而作的一些小技巧从而绕过这些规范)
    getClassLoader()获取类加载器 getTypeParameters()获取泛型 getGenericSuperclass() 获取父类 getPackage()获取包 
    getInterfaces()实现的接口的数组 getGenericInterfaces() 带泛型 
    getEnclosingMethod()如果是匿名内部类则返回所在方法 getEnclosingConstructor()所在方法构造方法 
    getDeclaringClass()如果是内部类则返回外部类 getEnclosingClass()
    getSimpleName() getTypeName() getCanonicalName() 
    方法 构造方法 字段 所有 所有公共 很久字段名或参数获取单个
    public InputStream getResourceAsStream(String name) public java.net.URL getResource(String name)
    private static class Atomic 
    private static class ReflectionData<T> 
    isEnum() getEnumConstants()
    public T cast(Object obj)
    public <U> Class<? extends U> asSubclass(Class<U> clazz)
        
ClassLoader
    
Compiler
    java->本地代码的编译
        
String
    implements java.io.Serializable, Comparable<String>, CharSequence
    private final char value[];    //final的数组 不可变 
	private int hash; 	//延迟初始化  h = 31 * h + val[i];
    private static final ObjectStreamField[] serialPersistentFields = new ObjectStreamField[0];   
	空参构造器 拷贝构造器 字符数组构造器 字符数组节选 int数组节选 byte数组节选 ascii数组 带编码的
    indexOf(int ch, int fromIndex)		//要遍历  重载
    public String concat(String str) 	//连接
    public native String intern();

StringCoding
        
Charset  
        
AbstractStringBuilder
    
StringBuilder
    
Enum
    implements Comparable<E>, Serializable   
    private final int ordinal;
    public static <T extends Enum<T>> T valueOf(Class<T> enumType,String name)
    
Iterable
    Iterator<T> iterator();		//返回迭代器
	default void forEach(Consumer<? super T> action) for (T t : this) {action.accept(t);}	//stream的foreach吧
	default Spliterator<T> spliterator() {return Spliterators.spliteratorUnknownSize(iterator(), 0);}

Throwable
      
Error 
    extends Throwable
    几个构造方法
    
Exception
    extends Throwable
    几个构造方法
  
Process
    
Package
    
System
    public final static InputStream in = null;		//标准输入流 Scanner那块用的那个	setIn(InputStream in)
	public final static PrintStream out = null;		//sout那个			setOut(PrintStream out)//静态java方法 调用本地方法
	public final static PrintStream err = null;		//0.0				setErr(PrintStream err)
	private static volatile SecurityManager security = null;	//系统安全管理器
	private static volatile Console cons = null;	 public static Console console()	//
    public static native long currentTimeMillis()
    public static native long nanoTime()	//纳秒
    public static native void arraycopy(Object src,  int  srcPos,Object dest, int destPos,int length);	//拷贝数组
	public static native int identityHashCode(Object x)		//默认hash
    private static Properties props;//System properties	get() set() clear()
    private static native Properties initProperties(Properties props);	
	public static Properties getProperties() 
    public static String lineSeparator() {return lineSeparator;}		//  UNIX systems "\n"  	Windows systems "\r\n"
	public static String getenv(String name) 	//获取环境变量的值 Map<String,String> getenv() 
        
ThreadLocal
    private final int threadLocalHashCode = nextHashCode()->nextHashCode.getAndAdd(HASH_INCREMENT) ;		//
	private static AtomicInteger nextHashCode = new AtomicInteger();
	private static final int HASH_INCREMENT = 0x61c88647;
	get() set() remove() createMap() 还有一些初始值相关的方法
    ThreadLocalMap getMap(Thread t) {return t.threadLocals;}

ThreadLocalMap
    static class Entry extends WeakReference<ThreadLocal<?>>
    private Entry[] table;	//Entry数组 key是ThreadLocal的hash值
	int i = key.threadLocalHashCode & (table.length - 1);   Entry e = table[i];
	Map相关的一些方法
        
Runtime
    //饿汉式单例
    public void exit(int status) -> Shutdown.exit(status); //终止
	public void addShutdownHook(Thread hook) removeShutdownHook(Thread hook)	//虚拟机关闭挂钩
    public void halt(int status) -> Shutdown.halt(status);	//强行终止
   	runFinalizersOnExit(boolean value) -> Shutdown.setRunFinalizersOnExit(value);	//启用禁用 退出之前执行终结方法
	public Process exec(String command)		//执行命令 重载
    public native int availableProcessors()		//可用处理器数量
    public native long freeMemory()		//可用内存
    public native long totalMemory()	//总内存 应该是当前占用的
    public native long maxMemory()		//最大内存
    public void load(String filename)	//加载本地库
    public void loadLibrary(String libname)	//加载本地库
    public InputStream getLocalizedInputStream(InputStream in) {return in;}
    public OutputStream getLocalizedOutputStream(OutputStream out) {return out;}

Thread
    implements Runnable
    registerNatives()
    private volatile char name[] 
    private int priority优先级 
    private daemon = false;	//守护线程 
    private Runnable target	//要执行的东东 
    private ThreadGroup group线程组
    private ClassLoader contextClassLoader类加载器 
    ThreadLocal.ThreadLocalMap threadLocals = null;	
	ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;	//子线程创建时，需要自动继承父线程的ThreadLocal变量
	private static int threadInitNumber;	//自动编号匿名线程
	private static long threadSeqNumber	//生成线程id	nextThreadID() {return ++threadSeqNumber;}
    private long stackSize;	//栈大小
	private long tid;	//Thread ID
	private volatile int threadStatus = 0;	//线程是否启动
	private volatile Interruptible blocker;	//可中断IO中的被阻塞对象 锁Object blockerLock
	volatile Object parkBlocker;
	public static native Thread currentThread();	//当前线程静态方法
	public static native void yield();	//放弃CPU的本地方法
	public static native void sleep(long millis) throws InterruptedException;	//抱着锁睡觉 重载
	public Thread() {init(null, null, "Thread-" + nextThreadNum(), 0);}	//重载 可传Runnable 
	public synchronized void start()	//调用本地start0()方法
    public final void stop()	//强制停止 可能出问题
    public void interrupt() 	//中断
    public boolean isInterrupted()	//是否已被中断
    public final native boolean isAlive()	//线程是否存在
    public final void suspend()	//暂停
    public final void resume()	//恢复挂起
    public final synchronized void join(long millis)	//等待终止
    public static void dumpStack() {new Exception("Stack trace").printStackTrace();}	//将当前线程的堆栈跟踪打印
	public StackTraceElement[] getStackTrace()
    public static Map<Thread, StackTraceElement[]> getAllStackTraces()
        
Runnable
	@FunctionalInterface	//函数式接口标记在有且仅有一个抽象方法的接口上
    public abstract void run();    

Shutdown
    //控制虚拟机关闭顺序的数据结构和逻辑
    
Void
    public static final Class<Void> TYPE = (Class<Void>) Class.getPrimitiveClass("void");
    private Void() {}

很多Exception
    
很多Error
````



## java.util-----------------------------------------------------------------------------------------------------------------

### java.util.concurrent

```java
```



### java.util.function

```java
	
```



### java.util.jar

```java
```



### java.util.logging

```java
```



### java.util.prefs

```java
```



### java.util.spi

```java
```



### java.util.stream

```java
```



### java.util.zip

```java
```



## java.io--------------------------------------------------------------------------------------------------------------------

```java
```



## java.nio------------------------------------------------------------------------------------------------------------------

### java.nio.channels

```java
```

### java.nio.charset

```java
```



### java.nio.file

```java

```



## java.net-----------------------------------------------------------------------------------------------------------------

```java
```



## java.sql------------------------------------------------------------------------------------------------------------------

```java
```



## java.time---------------------------------------------------------------------------------------------------------------

```java
略
```



## java.math--------------------------------------------------------------------------------------------------------------

```java
略
```



## java.text----------------------------------------------------------------------------------------------------------------

```java
略
```





## 集合

### ArrayList

RandomAccess接口 表明支持快速随机访问

Arrays.copyOf->System.arraycopy()

MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8 但是最后好像可以到 Integer.MAX_VALUE

indexOf 遍历一遍

transient Object[] elementData readObject和writeObject手动读写

### HashMap

(key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16)

### HashSet

### Iterator(ListIterator..)



## 锁

### AbstractQueuedSynchronizer

### ReentrantLock

### ReentrantReadWriteLock



## Util

### Random

### ThreadLocalRandom

### String

### Collections

### Arrays

## Date

### Date

### Calendar

### SimpleDateFormat

### LocalDate

### LocalTime

### LocalDateTime



public static boolean isNaN(float v) {
    return (v != v);
}