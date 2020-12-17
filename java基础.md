# Java基础

## 面向对象三大特性

封装：隐藏对象的属性和实现细节，仅公开对外接口。

继承：子类继承父类的属性和方法，提高代码复用率。

多态：父对象引用可以根据当前赋给它的子对象的类型有不同的表现形式。

## 四大域关键字

|           | 同一个类 | **同一个包** | **不同包的子类** | **不同包的非子类** |
| --------- | -------- | ------------ | ---------------- | ------------------ |
| Private   | √        |              |                  |                    |
| Default   | √        | √            |                  |                    |
| Protected | √        | √            | √                |                    |
| Public    | √        | √            | √                | √                  |

## 四大其它关键字

### final

- 类不能被继承，方法也被隐式指定final
- 方法不能被重写
- 成员变量：基本类型不可变；引用类型引用对象不可变

### static

- 成员变量和方法将被所有对象共享
- 静态代码块：静态代码块—>非静态代码块—>构造方法；无论创建多少对象只执行一次
- 静态内部类：创建不依赖外围类；不能访问外围类的非static成员或方法

### this super

## 基本数据类型

| 基本类型 | 位数 | 字节 | 默认值  |
| -------- | ---- | ---- | ------- |
| byte     | 8    | 1    | 0       |
| short    | 16   | 2    | 0       |
| int      | 32   | 4    | 0       |
| long     | 64   | 8    | 0L      |
| char     | 16   | 2    | 'u0000' |
| float    | 32   | 4    | 0f      |
| double   | 64   | 8    | 0d      |
| boolean  | 1    |      | false   |

这八种基本类型都有对应的包装类分别为：Byte、Short、Integer、Long、Float、Double、Character、Boolean

### 自动拆箱与装箱

- **装箱**：将基本类型用它们对应的引用类型包装起来；
- **拆箱**：将包装类型转换为基本数据类型；

### 常量池

Java 基本类型的包装类的大部分都实现了常量池技术，即 Byte,Short,Integer,Long,Character,Boolean；前面 4 种包装类默认创建了数值[-128，127] 的相应类型的缓存数据，Character创建了数值在[0,127]范围的缓存数据，Boolean 直接返回True Or False。如果超出对应范围仍然会去创建新的对象。

```java
Integer i1 = 33;
Integer i2 = 33;
System.out.println(i1 == i2);// 输出 true
Integer i11 = 333;
Integer i22 = 333;
System.out.println(i11 == i22);// 输出 false
Double i3 = 1.2;
Double i4 = 1.2;
System.out.println(i3 == i4);// 输出 false
```

new直接创建新的对象。范围内数据的=赋值会直接引用现有对象

此外，表达式的比较会触发拆箱，装箱

## ==  equals

- 前者：基础数据类型比较值，引用类型表示指向的地址
- 后者：未重写也为地址，重写后为内容

### 重写equals

自反性，传递性，对称性，一致性，非空性

```java
@Override
public boolean equals(Object o) {
    if (o == this)
    	return true;
    if (!(o instanceof Person))
    	return false;
    if (o == null)
    	return false;

		Person p = (Person) o;

		return p.height == this.height && p.weight == this.weight;
}
```

## hashcode equals

- object中的hashcode返回地址的整数值
- equals相等则hashcode一定相等，反之不一定

## 序列化

- 将对象的字节序列持久化保存在文件中
- 可以在网络上传输对象字节序列

## String StringBuffer StringBuilder

### String

不可变对象，每次操作产生新的字符串

### StringBuffer StringBuilder

StringBuffer 和 StringBuilder  类的对象能够被多次的修改，并且不产生新的未使用对象。后者速度快但线程不安全

### String对象不可变

- 不可变原因：value数组是final的，指向地址不变；内部没有修改value的方法
- 不可变好处
	- 字符串池，提高字符串重用的效率
	- 提高使用安全与线程安全
	- 缓存了hashCode，适合作为键值使用

## 泛型

Java 泛型（generics）是 JDK 5 中引入的一个新特性, 泛型的本质是参数化类型，也就是说所操作的数据类型被指定为一个参数。

```java
//泛型类
public class Generic<T> {
  private T key;
  
  public Generic(T key) {
    this.key = key;
  }
  
  public T getKey() {
    return this.key;
  }
}
//泛型接口
public interface Generator<T> {
    public T method();
}
//泛型方法
public static < E > void printArray( E[] inputArray )
{         
    for ( E element : inputArray ){        
      System.out.printf( "%s ", element );
    }	
    System.out.println();
}
//类型限定
public static <E extends Comparable> E printArray(E[] inputArray) {
  	return inputArray[0];
}
class GeneratorImpl<T> implements Generator<String>{
    @Override
    public String method() {
        return "hello";
    }
}

```

### 泛型擦除

在编译期间，泛型类型信息会被擦掉替换为限定类型（没有限定类型的变量用Object）。

### 与多态的冲突

```java
class Pair<T> {

    private T value;

    public T getValue() {
        return value;
    }

    public void setValue(T value) {
        this.value = value;
    }
}
class DateInter extends Pair<Date> {

    @Override
    public void setValue(Date value) {
        super.setValue(value);
    }

    @Override
    public Date getValue() {
        return super.getValue();
    }

    public static void main(String[] args) {
        DateInter dateInter = new DateInter();
        dateInter.setValue(new Date());
        dateInter.setValue(new Object());	//出错
    }
}
```

由于泛型擦除，Pair中类仍为object，为解决会生成桥方法。

## 注解

见[链接](https://www.runoob.com/w3cnote/java-annotation.html)











