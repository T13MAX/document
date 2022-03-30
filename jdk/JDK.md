# JDK

[TOC ]



## java.lang

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
    toString()->getChars(int i, int index, char[] buf) final static char [] DigitOnes 
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
       
    
ClassLoader
    
Compiler
    
String
    
StringBuilder
    
Enum
    
Iterable
    
Throwable
    
Error
    
Exception
    
Process
    
Package
    
System
    
ThreadLocal
    
Runtime
    
Thread
    
Runnable
    
Shutdown
    
Void
    
很多Exception
    
很多Error
````



## java.util

## java.io

## java.nio

## java.net

## java.sql

## java.time

## java.math

## java.text



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