---
layout: blog
title: JAVA单例模式
category: tech
excerpt: JAVA单例模式的几种实现方案
---


在单例模式中有一种延时实例化方法，
当调用get方法获取单例时，判断是否已经实例化，如果没有，则创建该实例并返回。如果有直接返回。
在这个过程中如果考虑多线程并发问题，我们需要用双重锁定来保证该实例的单一性。
```java
public class Singleton  {

	private Singleton (){
		
	}
	private static Singleton  main;
	/**
	 * 双重锁定
	 * @return
	 */
	public static Singleton  get(){
		if(main==null){
			synchronized (Singleton.class) {
				if(main==null){
					main=new Singleton();
				}
			}
		}
		return main;
	}
	
	public static void main(String[] args) throws Exception{
		System.out.println(Singleton.get().toString());
	}
 
	@Override
	public String toString() {
		return "I am Singleton ";
	}
}

```

单例模式的另一种实例化方法是在静态代码块中直接创建实例，这样就不需要考虑多线程的问题了。

```java
public class Singleton  {

	private Singleton (){
		
	}
	private static Singleton  main=new Singleton();
}
```

今天读到一篇创新型的单例设计,记录一下
```java
public class Singleton {   
  
  static class SingletonHolder {   
    static Singleton instance = new Singleton();   
  }   
  
  public static Singleton getInstance() {   
    return SingletonHolder.instance;   
  }   
  
}  
```