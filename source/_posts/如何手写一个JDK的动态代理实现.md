---
title: 如何手写一个JDK的动态代理实现
date: 2020-12-18
updated: 2020-12-18
tags: 动态代理
categories: 动态代理
---

代理模式分为**静态代理**与**动态代理**两种模式，其中静态代理则必须在编译期就需要确认代理类与被代理类，灵活性不高，而动态代理则是在运行期来确认，代理类通过编译器在运行时动态生成，无需我们手动创建！

动态代理模式如果细分，又要分为**接口代理**与**子类代理**模式， 在我们常用的Spring AOP 中就用到了两种动态代理模式：**JDK动态代理**和**CGLIB动态代理**，两种动态代理形成互补。 

- 接口代理模式：需要代理类与被代理类实现共同接口，也就是说被代理类必须实现一个接口才能被代理。

- 子类代理模式：没有要求，因为代理类会做为被代理类的子类存在。（灵活性更高，效率更快）

<!--more-->

### 动态代理的条件

1. 两个角色： 代理对象，被代理对象
2. 代理对象需要完成被代理对象的需要完成的业务操作
3. 代理对象持有被代理对象的引用
4. JDK动态代理 被代理对象必须实现接口，CGLIB动态代理被代理类和方法不能用final修饰

### 实现代码

1. 被代理对象实现的目标接口

```java
package com.nqmysb.proxy.jdk;


/**
 * 目标接口
 * @author liaocan
 *
 */
public interface Subject {
    
    /*
     * 抽象业务方法
     */
    void businessMethod();

}
```

1. 被代理的目标对象

```java
package com.nqmysb.proxy.jdk.impl;

import com.nqmysb.proxy.jdk.Subject;

/**
 * 具体的目标对象，实现目标接口的方法
 * @author liaocan
 *
 */
public class RealSubject implements Subject {

    @Override
    public void businessMethod() {
        System.out.println("我在进行具体的业务操作。。。。。。");
    }

}
```

1. 被代理的目标对象

```java
package com.nqmysb.proxy.jdk;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.Arrays;

/**
 * 代理类，必须实现JDK中的InvocationHandler  这是JDK动态代理的标志
 * 可以理解：需要实现JDK的代理，必须要这个证书 
 * @author liaocan
 *
 */
public class DynamicProxy implements InvocationHandler {
    
    private Subject target; //被代理对象的引用  这里是代理对象的接口  说明 该代理类是可以代理该接口下的所有子类的
    
    //动态生成代理对象
    public Object getInstance(Subject target) throws Exception{
        //获取代理对象
        this.target = target;
        //代理的对象必须是 suject的实现类
        Class<? extends Subject> clazz = target.getClass();
        
        //被代理对象的class是:class com.nqmysb.proxy.jdk.impl.RealSubject
        System.out.println("被代理对象的class是:"+clazz);
        
        //被代理对象的类加载器:sun.misc.Launcher$AppClassLoader@20eb607d
        ClassLoader classLoader = clazz.getClassLoader();
        System.out.println("被代理对象的类加载器:"+classLoader);
        
        //被代理对象的class是:[Ljava.lang.Class;@602f892f  返回所有实现接口的数组 [interface com.nqmysb.proxy.jdk.Subject]
        Class<?>[] classs = clazz.getInterfaces();
        System.out.println("被代理对象实现的接口:"+Arrays.asList(classs));
        //proxy的静态方法 newProxyInstance
        // 参数说明：被代理对象的类加载器 clazz.getClassLoader  ,被代理对象实现的接口 clazz.getInterfaces() ,当前对象 this
        //this 参数传入 把代理类和被代理类产生关联 当要执行被代理对象的方法时 会自动调用代理对象invoke方法进行代理执行
        return Proxy.newProxyInstance(classLoader, classs, this);
    }
    

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        
        System.out.println("开始代理...");
        System.out.println("代理之前做的事情...");
        System.out.println("------------"); 
        
        //执行代理方法
        method.invoke(this.target, args);
        
        System.out.println("------------");
        System.out.println("代理之后做的事情...");
        System.out.println("结束代理...");
        return null;
    }
    

}
```

1. JDK动态代理测试类

```java
package com.nqmysb.proxy.jdk;

import com.nqmysb.proxy.jdk.impl.RealSubject;
/**
 * JDK动态代理测试类
 * 
 * @author liaocan
 *
 */
public class DynamicProxyTest {

    public static void main(String[] args) {
        
    try {
                    
        Subject subject = (Subject)new DynamicProxy().getInstance(new RealSubject());
        System.out.println(subject.getClass());
        subject.businessMethod();

        } catch (Exception e) {
            e.printStackTrace();
        }

    }

}
```

1. 运行结果如下

```
被代理对象的class是:class com.nqmysb.proxy.jdk.impl.RealSubject
被代理对象的类加载器:sun.misc.Launcher$AppClassLoader@73d16e93
被代理对象实现的接口:[interface com.nqmysb.proxy.jdk.Subject]
class com.sun.proxy.$Proxy0
开始代理...
代理之前做的事情...
------------
我在进行具体的业务操作。。。。。。
------------
代理之后做的事情...
结束代理...
```

## 手写JDK动态代理

### JDK做了哪些事？

> 在自己手写JDK动态代理之前我们分析一下JDK动态代理帮我们做了哪些事情？

1. 代理类必须实现JDK中的java.lang.reflect.InvocationHandler接口，这个接口中只有一个方法,如下：

```java
/**
 * @param   proxy the proxy instance that the method was invoked on
 * @param   method the {@code Method} instance corresponding to
 * the interface method invoked on the proxy instance.  
 * @param   args an array of objects containing the values of the
 * arguments passed in the method invocation on the proxy instance,
 */
 public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
```

代理类需要重写和实现该方法，在这个方法里面我们可以进行附加业务操作，再调用被代理对象的方法。从而对被代理对象的方法进行扩展。

1. 生成代理类对象,代理类中的getInstance(Subject target)方法产生代理对象。关键代码如下：

```java
/**
* 该方法传入三个参数：1.类加载，用于加载动态生成代理类。2.被代理类实现的所有接口 3.代理类对象本身引用，用于回调我们实现的invoke方法 
*/
Proxy.newProxyInstance(classLoader, classs, this)
```

从上面两点可以得出JDK动态代理帮我们生成了代理类，代理实现了被代理类的所有接口，并将代理类加载到JVM中，生成代理实例，并回调我们实现的invoke方法进行动态代理。
JDK给我们提供了 java.lang.reflect.InvocationHandler 接口和 java.lang.reflect.Proxy 代理类 其中JDK1.8版本中的Proxy类中有一个私有的静态内部类 private static final class ProxyClassFactory 该类用于生成代理类的class对象。源代码如下：

```java
 private static final class ProxyClassFactory
        implements BiFunction<ClassLoader, Class<?>[], Class<?>>
    {
        //$Proxy 作为代理类的标志
        private static final String proxyClassNamePrefix = "$Proxy";

        private static final AtomicLong nextUniqueNumber = new AtomicLong();

        @Override
        public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {

            Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
            for (Class<?> intf : interfaces) {
          
                Class<?> interfaceClass = null;
                try {
                    interfaceClass = Class.forName(intf.getName(), false, loader);
                } catch (ClassNotFoundException e) {
                }
                if (interfaceClass != intf) {
                    throw new IllegalArgumentException(
                        intf + " is not visible from class loader");
                }
          
                if (!interfaceClass.isInterface()) {
                    throw new IllegalArgumentException(
                        interfaceClass.getName() + " is not an interface");
                }
       
                if (interfaceSet.put(interfaceClass, Boolean.TRUE) != null) {
                    throw new IllegalArgumentException(
                        "repeated interface: " + interfaceClass.getName());
                }
            }

            String proxyPkg = null;    
            int accessFlags = Modifier.PUBLIC | Modifier.FINAL;

 
            for (Class<?> intf : interfaces) {
                int flags = intf.getModifiers();
                if (!Modifier.isPublic(flags)) {
                    accessFlags = Modifier.FINAL;
                    String name = intf.getName();
                    int n = name.lastIndexOf('.');
                    String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
                    if (proxyPkg == null) {
                        proxyPkg = pkg;
                    } else if (!pkg.equals(proxyPkg)) {
                        throw new IllegalArgumentException(
                            "non-public interfaces from different packages");
                    }
                }
            }

            if (proxyPkg == null) {
                proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";
            }

    
            long num = nextUniqueNumber.getAndIncrement();
            //动态生成代理类的全名
            String proxyName = proxyPkg + proxyClassNamePrefix + num;

            //动态生成代理类class对象
            byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
                proxyName, interfaces, accessFlags);
            try {
                return defineClass0(loader, proxyName,
                                    proxyClassFile, 0, proxyClassFile.length);
            } catch (ClassFormatError e) {
           
                throw new IllegalArgumentException(e.toString());
            }
        }
    }
```

### 手写JDK动态代理代码实现

> 前两个要素不变：目标接口和目标实现类。

1. 被代理对象实现的目标接口

```java
package com.nqmysb.proxy.jdk;


/**
 * 目标接口
 * @author liaocan
 *
 */
public interface Subject {
    
    /*
     * 抽象业务方法
     */
    void businessMethod();

}
```

1. 被代理的目标对象

```java
package com.nqmysb.proxy.jdk.impl;

import com.nqmysb.proxy.jdk.Subject;

/**
 * 具体的目标对象，实现目标接口的方法
 * @author liaocan
 *
 */
public class RealSubject implements Subject {

    @Override
    public void businessMethod() {
        System.out.println("我在进行具体的业务操作。。。。。。");
    }

}
```

1. 自定义类加载器

```java
package com.nqmysb.myproxy.proxytools;

import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.util.HashMap;


/**
 * 自定义类加载器
 * 
 * 作用：将我们动态生成的class文件生成class对象加载JVM内存中
 * @author liaocan
 *
 */
public class MyClassLoader extends ClassLoader{
    
    private File baseDir;
    
    public MyClassLoader(){
        
        Class<MyClassLoader> clazz = MyClassLoader.class;
        String basePath = clazz.getResource("").getPath();
//        String basePath = clazz.getResource("/").getPath() +"com/nqmysb/myproxy/proxytools";
        this.baseDir = new java.io.File(basePath);
    }
    /**
     * 自定义类加载方法 ，加载自定义类
     */
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        //获取需要加载的类的全名  com.nqmysb.myproxy.proxytools.$Proxy0
        String className = MyClassLoader.class.getPackage().getName() + "." + name;
        if(baseDir != null){
            //获取 class文件对象 E:\workspace\spring-source-analysis\spring-source-proxy\bin\com\nqmysb\myproxy\proxytools\$Proxy0.class
            File classFile = new File(baseDir,name.replaceAll("\\.", "/") + ".class");
            if(classFile.exists()){
                FileInputStream in = null;
                ByteArrayOutputStream out = null;
                try{
                    in = new FileInputStream(classFile);
                    out = new ByteArrayOutputStream();
                    byte [] buff = new byte[1024];
                    int len;
                    while ((len = in.read(buff)) != -1) {
                        out.write(buff, 0, len);
                    }
                    //根据class文件和类名生成class对象 并加载到JVM中
                    return defineClass(className, out.toByteArray(), 0,out.size());
                    
                }catch (Exception e) {
                    e.printStackTrace();
                }finally{
                    if(null != in){
                        try {
                            in.close();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                    if(null != out){
                        try {
                            out.close();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                    //删除Class文件
                    classFile.delete();
                }
                
            }
        }
        
        return null;
    }
    
    /**
     * 测试了解Java类加载机制
     * 1.类加载器的作用：类加载器负责将class文件读入JVM内存，并为之生成对应的java.lang.Class对象。
     * java类加载器分为三类：
     * （Bootstrap ClassLoader） 启动类加载器 ： 负责加载Java的核心类（如String、System等）, rt.jar
     *                             它比较特殊，因为它是由原生C++代码实现的，并不是java.lang.ClassLoader的子类
     * （Extension ClassLoader ）扩展类加载器：它负责加载JRE的扩展目录（%JAVA_HOME%/jre/lib/ext）中JAR包的类，java实现
     *                             我们可以通过把自己开发的类打包成JAR文件放入扩展目录来为Java扩展核心类以外的新功能。
     * （ApplicationClassLoader）应用程序类加载器：它负责在JVM启动时加载来自Java命令的-classpath选项、java.class.path系统属性，java实现
     *                             或CLASSPATH环境变量所指定的JAR包和类路径。程序可以通过ClassLoader的静态方法getSystemClassLoader来获取系统类加载器：
     * 
     * 类加载机制之双亲委派
     *        加载类时 会现递归交于父类加载器加载，如果所有父类没有加载才有自己加载，好处防止类的重复加载
     * 
     */
    
    public static void main(String[] args) {
        //（Bootstrap ClassLoader） 启动类加载器 ： 负责加载Java的核心类（如String、System等）rt.jar
        //它比较特殊，因为它是由原生C++代码实现的，并不是java.lang.ClassLoader的子类 所以输出结果为null
        //Returns the class loader for the class.  Some implementations may use
        //null to represent the bootstrap class loader. This method will return
        //null in such implementations if this class was loaded by the bootstrap
        //class loader.
        System.out.println(String.class.getClassLoader());
        System.out.println(HashMap.class.getClassLoader());
        
        //sun.misc.Launcher$AppClassLoader
        System.out.println(MyClassLoader.class.getClassLoader().getClass().getName());

        //sun.misc.Launcher$AppClassLoader
        System.out.println(ClassLoader.getSystemClassLoader().getClass().getName());

        
        
    }

}
```

1. 自定义 InvocationHandler 接口

```java
package com.nqmysb.myproxy.proxytools;

import java.lang.reflect.Method;

public interface MyInvocationHandler {

    Object invoke(Object proxy, Method method, Object[] args) throws Throwable;

}
```

1. 自定义 Proxy 代理类

```java
package com.nqmysb.myproxy.proxytools;

import java.io.File;
import java.io.FileWriter;
import java.lang.reflect.Constructor;
import java.lang.reflect.Method;

import javax.tools.JavaCompiler;
import javax.tools.JavaCompiler.CompilationTask;
import javax.tools.JavaFileObject;
import javax.tools.StandardJavaFileManager;
import javax.tools.ToolProvider;
/**
 * 自己实现的Proxy  对应JDK中的java.lang.reflect.Proxy类
 * 
 * @author liaocan
 *
 */
public class MyProxy {
    
    //换行符
    private static String ln = "\r\n";


    /**
     * 生成代理类对象  
     *  对应JDK中Proxy的newProxyInstance方法生成代理类对象
     *  JDK1.8中的实现是：通过Proxy 中的的静态内部类ProxyClassFactory =>(private static final class ProxyClassFactory)中的apply方法
     *  注意：JDK1.7中是通过getProxyClass()方法实现 有点小区别 但是核心代码大致相同
     *  核心代码如下：
     *      //所有代理类的前缀
     *      private static final String proxyClassNamePrefix = "$Proxy";
     *      //代理类的序号
     *      long num = nextUniqueNumber.getAndIncrement();
     *      //代理类的全名
     *      String proxyName = proxyPkg + proxyClassNamePrefix + num;
     *      //生成代理类class字节码文件 ,反编译生成的class文件可以发现代理类继承Proxy类实现了被代理类的implements的所有接口 =>(public final class $Proxy0 extends Proxy  implements Subject ) 
     *      //generateProxyClass()可以再看看具体实现
     *      byte[] proxyClassFile = ProxyGenerator.generateProxyClass(proxyName, interfaces, accessFlags);
     *      try {
     *      //根据class字节码生成实例对象
     *          return defineClass0(loader, proxyName,proxyClassFile, 0, proxyClassFile.length);
     *         } 
     *          
     * @param classLoader
     * @param interfaces
     * @param myInvocationHandler
     * @return
     */
    public static Object newProxyInstance(MyClassLoader classLoader,Class<?>[] interfaces,MyInvocationHandler myInvocationHandler){
        
        
        try{
            //1、动态生成代理类的源代码 .java文件
            String proxySrc = generateSrc(interfaces[0]);
            
            
            //2、将生成的源代码输出到磁盘，保存为.java文件
            String filePath = MyProxy.class.getResource("").getPath(); //打jar包之后抛空指针
//            String filePath = MyProxy.class.getResource("/").getPath() +"com/nqmysb/myproxy/proxytools/";
            File f = new File(filePath + "$Proxy0.java");
            FileWriter fw = new FileWriter(f);
            fw.write(proxySrc);
            fw.flush();
            fw.close();
        
            //3、编译源代码，并且生成.class文件
            JavaCompiler  compiler = ToolProvider.getSystemJavaCompiler();
            StandardJavaFileManager manager = compiler.getStandardFileManager(null, null, null);
            Iterable<? extends JavaFileObject> iterable = manager.getJavaFileObjects(f);
            
            CompilationTask task = compiler.getTask(null, manager, null, null, null, iterable);
            task.call();
            manager.close();
        
            //4.将class文件中的内容，动态加载到JVM中来
            
            //5.返回被代理后的代理类class对象
            Class<?> proxyClass = classLoader.findClass("$Proxy0");
            //通过class对象和参数获取指定构造对象 
            Constructor<?> c = proxyClass.getConstructor(MyInvocationHandler.class);
            //删除Java文件
            f.delete();
            
            return c.newInstance(myInvocationHandler);
            
        }catch (Exception e) {
            e.printStackTrace();
        }
        
        
        return null;
    }
    
    /**
     * 
     * 动态生成代理类的.java文件的代码内容
     * @param interfaces
     * @return
     */
    private static String generateSrc(Class<?> interfaces){
        StringBuffer src = new StringBuffer();
        //代理类包名
        src.append("package com.nqmysb.myproxy.proxytools;" + ln);
        //导入反射包method类  
        src.append("import java.lang.reflect.Method;" + ln);
        //声明类名 实现被代理类的接口这里简化处理只实现一个接口, JDK中生成的代理类会继承proxy类 实现被代理类的所有接口
        src.append("public class $Proxy0 implements " + interfaces.getName() + "{" + ln);
        //生成代理类中持有MyInvocationHandler引用
        //JDK中InvocationHandler是Proxy中的成员变量=>(protected InvocationHandler h;)
        //生成的代理类继承Proxy也继承了它的protected的成员变量,我们自己生成的代理类没有继承Proxy 所有需要自己提供
        src.append("MyInvocationHandler h;" + ln);
        //代理类中构造方法中传入MyInvocationHandler给InvocationHandler初始化
        src.append("public $Proxy0(MyInvocationHandler h) {" + ln);
        src.append("this.h = h;" + ln);
        src.append("}" + ln);
        //生成所有代理方法
        for (Method m : interfaces.getMethods()) {
            src.append("public " + m.getReturnType().getName() + " " + m.getName() + "(){" + ln);
            src.append("try{" + ln);
            //通过代理方法对象
            src.append("Method m = " + interfaces.getName() + ".class.getMethod(\"" +m.getName()+"\",new Class[]{});" + ln);
            //调用代理方法
            src.append("this.h.invoke(this,m,null);" + ln);
            src.append("}catch(Throwable e){e.printStackTrace();}" + ln);
            src.append("}" + ln);
        }
        
        src.append("}");
        System.out.println("动态生成的代理类$Proxy0:"+ln+src);
        /*
        生成代理类如下：
        package com.nqmysb.myproxy.proxytools;
        
        import java.lang.reflect.Method;
        
        public class $Proxy0 implements com.nqmysb.myproxy.Subject{
        
        MyInvocationHandler h;
        
        public $Proxy0(MyInvocationHandler h) {
            this.h = h;
        }
        
        public void businessMethod(){
        try{
            
                Method m = com.nqmysb.myproxy.Subject.class.getMethod("businessMethod",new Class[]{});
                //此处调用的其实就是传入的com.nqmysb.myproxy.MyDynamicProxy对象的invoke方法
                //然后再在invoke中对代理对象的方法前后进行业务方法操作
                this.h.invoke(this,m,null);
                
                }catch(Throwable e){
                e.printStackTrace();
                }
            }
        }
        */
        return src.toString();
    }

}
```

1. 使用自己手写的动态代理

```java
package com.nqmysb.myproxy;

import java.lang.reflect.Method;
import java.util.Arrays;

import com.nqmysb.myproxy.proxytools.MyClassLoader;
import com.nqmysb.myproxy.proxytools.MyInvocationHandler;
import com.nqmysb.myproxy.proxytools.MyProxy;

/**
 * 代理类，必须实现JDK中的InvocationHandler  这是JDK动态代理的标志
 * 可以理解：需要实现JDK的代理，必须要这个证书 
 * @author liaocan
 *
 */
public class MyDynamicProxy implements MyInvocationHandler {
    
    private Subject target; //被代理对象的引用  这里是代理对象的接口  说明 该代理类是可以代理该接口下的所有子类的
    
    //动态生成代理对象
    public Object getInstance(Subject target) throws Exception{
        //获取代理对象
        this.target = target;
        //代理的对象必须是 suject的实现类
        Class<? extends Subject> clazz = target.getClass();
        //获取被代理类实现的所有接口  从这里可以看出如果被代理类没有实现接口，是无法使用JDK动态代理的
        Class<?>[] classs = clazz.getInterfaces();
        System.out.println("被代理对象实现的接口:"+Arrays.asList(classs));
        return MyProxy.newProxyInstance(new MyClassLoader(), classs, this);
    }
    

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        
        System.out.println("开始代理...");
        System.out.println("代理之前做的事情...");
        System.out.println("------------"); 
        
        //执行代理方法
        method.invoke(this.target, args);
        
        System.out.println("------------");
        System.out.println("代理之后做的事情...");
        System.out.println("结束代理...");
        return null;
    }
    

}
```

1. 测试动态代理

```java
package com.nqmysb.myproxy;

import com.nqmysb.myproxy.impl.RealSubject;

/**
 * 手写JDK动态代理测试类
 * 
 * 
 * @author liaocan
 *
 */
public class MyDynamicProxyTest {

    public static void main(String[] args) {
        
    try {
            
            Subject subject = (Subject)new MyDynamicProxy().getInstance(new RealSubject());
            System.out.println(subject.getClass());
            subject.businessMethod();


        } catch (Exception e) {
            e.printStackTrace();
        }

    }

}
```

1. 运行结果

```java
被代理对象实现的接口:[interface com.nqmysb.myproxy.Subject]
动态生成的代理类$Proxy0:
package com.nqmysb.myproxy.proxytools;
import java.lang.reflect.Method;
public class $Proxy0 implements com.nqmysb.myproxy.Subject{
MyInvocationHandler h;
public $Proxy0(MyInvocationHandler h) {
this.h = h;
}
public void businessMethod(){
try{
Method m = com.nqmysb.myproxy.Subject.class.getMethod("businessMethod",new Class[]{});
this.h.invoke(this,m,null);
}catch(Throwable e){e.printStackTrace();}
}
}
class com.nqmysb.myproxy.proxytools.$Proxy0
开始代理...
代理之前做的事情...
------------
我在进行具体的业务操作。。。。。。
------------
代理之后做的事情...
结束代理...
```

### 总结

动态代理的底层实现的原理：

1. 反射机制：获取被代理类的方法，并实现动态字节码重组生成新的代理类
   值得注意：JDK动态代理和CGLIB动态代理使用的反射机制是不一样的，

分别是：java.lang.reflect.Method和net.sf.cglib.reflect.FastClass 类实现的

1. 函数回调机制：将代理类需要执行的附加方法传入新生成的代理类中，执行被代理类方法时会回调代理类的方法

### 代码工程

github地址：https://github.com/nqmysb/spring-source-analysis