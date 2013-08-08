---
layout: blog
title: JAVA序列化
category: tech
excerpt: JAVA序列化基础知识记录
---


当我们需要序列化一个JAVA对象时需要实现Serializable接口。这个接口仅仅是一个tag接口，
并不需要你真正实现一些方法，因为这个接口没有方法。他作用仅仅是告诉默认JAVA序列化工具，这个对象是可以序列化的。

####1.serialVersionUID的作用

当我们的类实现了Serializable接口后，会有一个警告，告诉你需要生成一个serialVersionUID属性。
这个serialVersionUID是做什么用的呢？其实这是JAVA序列化的版本控制功能。当序列化对象时会把这个属性写入，
当反序列化时则会把这个属性取出，然后与JAVA类中的serialVersionUID属性值对比，如果一致，则认为是同一个版本，
正常反序列化，如果不一致则认为版本不同，抛出InvalidClassException异常。

很多时候我们忽略这个警告，并不写这个serialVersionUID属性，但仍然可以正常序列化。
那是因为如果没有这个属性，JVM将会根据这个类的属性和方法，计算出一个值作为serialVersionUID的值。
这种做法会带来潜在的风险。不同的JVM产生serialVersionUID的算法可能会不一致，如果在不同的环境下产生的serialVersionUID不一致，将导致反序列化失败！

当一个类的结构发生变化，需要改变serialVersionUID，以体现序列化机制此类发生了变化，不兼容原来的版本了！其实并不是只有类结构变化，
就必须更改serialVersionUID，JAVA的序列化机制提供了部分变化的兼容机制，有如下几种：

#####添加新的属性 反序列时发现没有此属性，则会赋予该属性默认的类型值

#####添加 writeObject/readObject 方法 因为此方法是用于自定义序列化，不影响序列化

#####删除 writeObject/readObject 方法 同上原因

#####改变属性的访问权限(private protected package public)

#####将一个属性从static变为nonstatic 或者 transient 或者 nontransient

如果你的类改动属于以上范围，默认的JAVA序列化机制可以保证兼容性，也就是说你不需要改动serialVersionUID值。

更多情况参见[url]http://java.sun.com/j2se/1.4/pdf/serial-spec.pdf[/url] 5.6.2 Compatible Changes

其实大部分情况下我们不需要深究哪些改动会影响兼容性。如果我们的序列化仅仅是用作缓存的话，我们可以简单处理，只要改动了类结构，即修改serialVersionUID值，抛弃原先的序列化结果，重新生成！

####JAVA序列化过程

JAVA默认的序列化类是ObjectOutputStream,反序列化类是ObjectInputStream。

```java
	public static void main(String[] args) throws Exception {

		Object o=new Object();
		
			try {
				//序列化
				FileOutputStream ostream = new FileOutputStream("t.txt");
				ObjectOutputStream p = new ObjectOutputStream(ostream);
				p.writeObject(o); // 序列化对象，在内部通过调用defaultWriteFields(Object obj, ObjectStreamClass desc)来序列化对象的所有属性。
				p.flush();
				ostream.close();
				//反序列化
				FileInputStream fis=new FileInputStream("t.txt");
				ObjectInputStream istream=new ObjectInputStream(fis);
				Object s=(ObjectSerializable) istream.readObject();//反序列化对象，内部调用defaultReadFields(Object obj, ObjectStreamClass desc)来反序列化所有属性
				System.out.println(s);
				istream.close();
				fis.close();
			} catch (IOException ioe) {
				ioe.printStackTrace();
			}
	}
```
 
####JAVA自定义序列化

自定义序列化根据定制程度的不同，有多种定制方案。

#####1.Externalizable定制

Externalizable接口继承了Serializable，其中有2个方法：

```java
void writeExternal(ObjectOutput out) throws IOException;
void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
```

通过实现Externalizable这个接口，我们可以定制需要序列化的属性。

```java
/*
*ObjectOutputStream在调用writeObject()方法时，会判断需要序列化的类是否继承了
*Externalizable接口，如果是，则会调用writeExternal(ObjectOutput out)执行我们
*自己写的序列化代码
*/
p.writeObject(o); 
/*
*同理，ObjectInputStream在调用readObject()方法时，会判断需要反序列化的类是否继承
*了Externalizable接口，如果是，则会调用readExternal(ObjectInput in)执行我们自己
*写的反序列化代码
*/
Object s=(ObjectSerializable) istream.readObject()
```

#####2.重写ObjectOutputStream/ObjectInputStream

实现Externalizable的方式自定义序列化非常方便，只需要在序列化类内部添加2个方法即可，不需要外部的任何要求。
但是如果我们需要更加深度的定制这还是不够的。Externalizable无法定制序列化对象本身的描述，只能定制对象内部属性的描述。
此时我们需要新建一个自己的序列化类来实现。

```java
/**
 * 自定义序列化类
 */
public class CustomObjectOutputStream extends ObjectOutputStream {

	private OutputStream cusOut;
	
	public CustomObjectOutputStream(OutputStream out) throws IOException{
		/*
		 * 通过调用super()可以将父类的enableOverride设置为true
		 * 当调用父类的writeObject(obj)时，因为enableOverride=true，会调用writeObjectOverride(Object obj)方法
		 * 因此我们需要覆写writeObjectOverride(Object obj)如下所示
		 */
		super();
		cusOut=out;
	}
	
	@Override
	protected void writeObjectOverride(Object obj) throws IOException {
		//自定义序列化方案
		//cusOut.write(b)...
	}

}
```
```java
/**
 * 自定义反序列化类
 */
public class CustomObjectInputStream extends ObjectInputStream{
	
	private InputStream cusIn;

	public CustomObjectInputStream(InputStream in)throws IOException {
		/*
		 * 通过调用super()可以将父类的enableOverride设置为true
		 * 当调用父类的readObject()时，因为enableOverride=true，会调用readObjectOverride()方法
		 * 因此我们需要覆写readObjectOverride()如下所示
		 */
		super();
		cusIn=in;
	}
	
	@Override
	protected Object readObjectOverride() throws IOException,
			ClassNotFoundException {
		//自定义反序列化方案
		//cusIn.read() ...
		return null;
	}

}
```
以上2段代码继承了ObjectOutputStream和ObjectInputStream，完全自定义了序列化方法。

既然是完全自定义序列化方法，其实完全没有必要去继承ObjectOutputStream和ObjectInputStream。

上面的代码仅仅适用于做序列化的适配器。其他序列化机制比如hessian,protobuf等，需要适配JAVA默认的序列化机制，则可采用以上的方法适配
