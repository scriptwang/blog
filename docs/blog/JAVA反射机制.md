

#  Java动态加载机制
动态加载就是需要某个类的时候才对其进行加载，而不是对所有的类加载完成后才开始执行main方法。(需要看虚拟机详细的加载输入，需要对虚拟机输入 -verbose:class 参数)
```java
public class Test{
    public static void main(String[] args){
        new A();//看到A才加载A，此时B还没有被加载
        System.out.println("=======");//横线会被打印在加载A和加载B之间   
        new B();//看到B才加载B
 
        new C();
        new C();//new了两次C却只有一次输出
 
        new D();
        new D();//new两次D就两次输出
    }
}
 
 
class A{
}
 
class B{
}
 
class C{
    static{//在加载完成后被调用，且只调用一次
        System.out.println("静态语句块，不管new多少个对象只会执行一次");
    }
}
 
class D{
    {//动态语句块，和构造方法差不多，用得少
        System.out.println("没每new一个对象执行一次");
    }
}
```

# JDK中的ClassLoader
包含四个ClassLoader，bootstrap class loader是用本地语言写的，别的class loader用java写的。

bootstrap class loader加载别的class loader，然后别的class loader被加载完成之后，再去加载另外的class。

在java中，每个class都有一个Class对象，当编译完一个.class文件后，就会产生一个Class对象，用以描述这个类的类型信息
```java
package com.scriptwang;
public class Test{
    public static void main(String[] args) throws Exception{
        //三种方法可以拿到Test类的Class对象
        Class a = new com.scriptwang.Test().getClass();//getClass继承自Object
        Class b = com.scriptwang.Test.class;//个人理解相当于每个类都有一个Class class的成员变量，是public的，通过类名.属性名可以拿到该成员变量。
        Class c = Class.forName("com.scriptwang.Test");//通过Class类的forName("完整的类名")
 
 
        //拿到加载Test类的CLassLoader的名字
        /*
            Class c ＝ Test.class;//返回Test类的Class对象
            ClassLoader cl = c.getClassLoader();//返回ClassLoader实例
            Class d ＝ cl.getClass();//拿到ClassLoader类的Class对象
            (ClassLoader本身也是一个java类，getClass方法继承自Object类)
            String name ＝ d.getName();//拿到Class对象返回的类的名字
        */
        System.out.println(Test.class.getClassLoader().
                getClass().getName());
 
    }
}
```

# ClassLoader的层次关系（不是继承，继承是从类的角度说的，层次是对象和对象之间的关系）
application class loader对象里面有一个引用，叫做Parent，指向extension class loader，同理，extension class loader有一个Parent指向bootstrap class loader（可以这么认为，因为bootstrap class loader是用本地的语言写的，不能被指向）

通过getParent方法可以拿到上一个层次的class loader，因此可以用循环遍历class loader对象。不是类继承，不是类继承，不是类继承，重要的事情说三遍。

设置这样的层次关系的目的：当加载一个类的时候，会通过检查（一直检查到root class loader）上一层次的Class Loader是否加载，如果上一个层次加载过了，那么当前的Class Loader就不会加载这个类。比如：自己写的java.lang.String永远不会被加载！这样可以保证安全性。
```java
public class Test{
    public static void main(String[] args){
        //拿到加载Test类的CLassLoader
        ClassLoader c = Test.class.getClassLoader();
         
        //通过循环遍历ClassLoader
        while (c != null ){
            System.out.println(c.getClass().getName());
            c = c.getParent();//拿到上一层次的ClasLoader
        }
    }
}
```

# 反射机制
JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能称为java语言的反射机制。

JAVA反射（放射）机制："程序运行时，允许改变程序结构或变量类型，这种语言称为动态语言"。从这个观点看，Perl，Python，Ruby是动态语言，C++，Java，C#不是动态语言。但是JAVA有着一个非常突出的动态相关机制：Reflection，用在Java身上指的是我们可以于运行时加载、探知、使用编译期间完全未知的classes。换句话说，Java程序可以加载一个运行时才得知名称的class，获悉其完整构造（但不包括methods定义），并生成其对象实体、或对其fields设值、或唤起其methods
```java
import java.lang.reflect.Method;
 
public class Test{
    public static void main(String[] args) throws Exception{
        String className = "Test";
        Class c = Class.forName(className);//通过class名拿到Class对象
        Object instance = c.newInstance();//通过Class对象new这个类的对象
 
        Method[] methods = c.getMethods();//通过Class对象获得Method对象
        for (int i=0;i<methods.length;++i){
            if (methods[i].getName() == "m"){
                methods[i].invoke(instance,12);
            }
        }
    }    
    public void m(int i){
        System.out.println(i);
    }
}
 
/*
forName会抛出ClassNotFoundException
newInstance会抛出InstantiationException、IllegalAccessException
invoke会抛出IllegalAccessException、InvocationTargetException
*/
```
 比如这样一个例子，有一个接口叫做Animal，它有方法enjoy、eat、sleep，所有实现了这个接口的具体类（比如Cat、Dog）都拥有这些方法，因此，在通过反射机制new具体对象的时候，反射机制的代码并不用改变（因为具体的类的方法都是依据接口固定化了的）。
 ```java
public interface Animal {//定义一个接口
    public void enjoy(String thing);
    public void eat();
}
 
public class Dog implements Animal{//实现了Animal接口的Dog
    public void enjoy(String thing){
        System.out.println("Dog is enjoy " + thing);
    }
    public void eat(){
        System.out.println("Dog is eatting");
    }
}
 
public class Cat implements Animal {//实现了Animal借口的Cat
    public void enjoy(String thing){
        System.out.println("Cat is enjoy "+ thing);
    }
    public void eat(){
        System.out.println("Cat is eatting");
    }
}
 
import java.lang.reflect.Method;//引入相关的类
public class UserClass {
    public static void main(String[] args) throws Exception{
       //不管从配置文件获得的类名是Cat还是Dog都可以传进来，只要实现了Animal接口的具体类都可以传入，以下的代码照样适用，因为它们都实现了Animal接口（里面的方法申明都是一样的，只不过每个类具体的实现方式不一样而已，这样就实现了很好的扩展性）
        String className = "Cat";
        Class c =  Class.forName(className);
        Object o = c.newInstance();
        Method[] methods = c.getMethods();
        for (int i=0;i<methods.length;++i){
            if (methods[i].getName() == "enjoy"){
                methods[i].invoke(o,"playing");
            }
            if (methods[i].getName() == "eat"){
                methods[i].invoke(o);
            }
        }
 
    }
}
 ```
# 反射机制的一些相关应用（创建对象、访问Field、调用方法、重写toString）
```java
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
  
/**
 * Created by Script Wang on 2016/11/27.
 */
  
class Student {
    private String name;
    private int age;
    Student (){
        name = "";
        age = 0;
    }
    Student (String name,int age){
        this.name = name;
        this.age = age;
    }
  
    public void m(){
        System.out.println("I am invoked ===== !!!");
    }
  
  
    /**
     *
     * @return
     * 利用反射重写toString
     */
    @Override
    public String toString(){
        StringBuilder sb = new StringBuilder();
  
        Field[] fields = this.getClass().getDeclaredFields();//拿到所有成员变量
  
        //循环成员变量，将变量名值添加到sb中
        for (int i=0;i<fields.length;i++){
            sb.append(fields[i].getName());//拿到当前Field的名字
            sb.append(" = ");//添加等号
            try{
                sb.append(fields[i].get(this));//拿到当前Field的值
            }catch (IllegalAccessException e){
                e.printStackTrace();
            }
  
            //如果遇到最后一个变量，则末尾不添加分号
            if (i != fields.length - 1){
                sb.append(" ; ");
            }
        }
  
        //返回sb的toString
        return sb.toString();
    }
  
}
  
public class TestReflect {
    public static void main(String[] args){
        test1();//打印示例：name = SW ; age = 21
        test2();//打印示例：name = SW ; age = 22
        test3();//打印示例：I am invoked ===== !!!
    }
  
    /**
     * 用反射中的Constructor构建对象
     */
    public static void test1(){
        Class clazz = null;
        Constructor con = null;
        Student s = null;
        try{
            clazz = Class.forName("Student");//抛ClassNotFoundException
            con = clazz.getDeclaredConstructor(String.class,int.class);//抛NoSuchMethodException
            s = (Student) con.newInstance("SW",21);//抛剩下的三个Exception
        }catch (ClassNotFoundException e){
            e.printStackTrace();
        }catch (NoSuchMethodException e){
            e.printStackTrace();
        }catch (InstantiationException e){
            e.printStackTrace();
        }catch (IllegalAccessException e){
            e.printStackTrace();
        }catch (InvocationTargetException e){
            e.printStackTrace();
        }
  
        System.out.println(s);
    }
  
  
  
    /**
     * 用反射为成员变量赋值构建对象（需要被构建对象的类有空的构造方法）
     * 需要知道构造函数需要哪些类型的参数
     *
     */
    public static void test2(){
        Class clazz = null;
        Object obj = null;
        Field[] fields = null;
  
        try{
            clazz = Class.forName("Student");//抛ClassNotFoundException
            obj = clazz.newInstance();//抛剩下的两个Exception
        }catch (ClassNotFoundException e){
            e.printStackTrace();
        }catch (InstantiationException e){
            e.printStackTrace();
        }catch (IllegalAccessException e){
            e.printStackTrace();
        }
  
        //拿到所有Field并循环
        fields = clazz.getDeclaredFields();
        for (Field f : fields){
            f.setAccessible(true);//这句相当重要，可以访问private的变量
            try {
                //get方法会抛IllegalAccessException
                if (f.get(obj) instanceof String){
                    f.set(obj,"SW");//为当前obj 赋值
                } else if (f.get(obj) instanceof Integer){
                    f.set(obj,22);
                }
            }catch (IllegalAccessException e){
                e.printStackTrace();
            }
        }
  
        System.out.println( (Student)obj );
  
    }
  
  
    /**
     * 利用反射调用方法
     */
    public static void test3(){
        Class clazz = null;
        Object obj = null;
        Method[] methods = null;
  
        try{
            clazz = Class.forName("Student");//抛ClassNotFoundException
            obj = clazz.newInstance();//抛剩下的两个Exception
        }catch (ClassNotFoundException e){
            e.printStackTrace();
        }catch (InstantiationException e){
            e.printStackTrace();
        }catch (IllegalAccessException e){
            e.printStackTrace();
        }
  
        //拿到所有的Method对象并循环
        methods =clazz.getDeclaredMethods();
        for (Method m : methods){
            if (m.getName().equals("m")){
                m.setAccessible(true);
                try {
                    m.invoke(obj);//调用obj的方法，抛出以下两个Exception
                }catch (IllegalAccessException e){
                    e.printStackTrace();
                } catch (InvocationTargetException e){
                    e.printStackTrace();
                }
  
            }
        }
  
    }
  
  
}
```
