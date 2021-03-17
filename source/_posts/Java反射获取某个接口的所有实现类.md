---
title: Java反射获取某个接口的所有实现类
date: 2020-12-28
updated: 2020-12-28
tags: 反射
categories: 反射
---

我们知道通过java反射可以获取到某个类的class对象，从而获取这个类中的所有重要信息，如获取到该类实现的所有接口和继承的父类，但是对于获取到该类的子类等信息则获取不到了，这就像我们可以知道古人，古人却不知道我们，向上可以获取，向下不行！那么通过什么手段可以向下获取呢？

<!--more-->

### 实现原理：

```java
/**
 * 获取一个接口的所有实现类
 *
 * @param target
 * @return
 */
public static ArrayList<Class<?>> getInterfaceImpls(Class<?> target) {
    ArrayList<Class<?>> subclassaes = Lists.newArrayList();
    try {
        // 判断class对象是否是一个接口
        if (target.isInterface()) {
            @NotNull
            String basePackage = target.getClassLoader().getResource("").getPath();
            File[] files = new File(basePackage).listFiles();
            // 存放class路径的list
            ArrayList<String> classpaths = Lists.newArrayList();
            for (File file : files) {
                // 扫描项目编译后的所有类
                if (file.isDirectory()) {
                    listPackages(file.getName(), classpaths);
                }
            }
            // 获取所有类,然后判断是否是 target 接口的实现类
            for (String classpath : classpaths) {
                Class<?> classObject = Class.forName(classpath);
                if (classObject.getSuperclass() == null) continue; // 判断该对象的父类是否为null
                Set<Class<?>> interfaces = new HashSet<>(Arrays.asList(classObject.getInterfaces()));
                if (interfaces.contains(target)) {
                    subclasses.add(Class.forName(classObject.getName()));
                }
            }
        } else {
            throw new ParamException("Class对象不是一个interface");
        }
    } catch (Throwable e) {
        e.printStackTrace();
    }
    return subclasses;
}

/**
 * 获取项目编译后的所有的.class的字节码文件
 * 这么做的目的是为了让 Class.forName() 可以加载类
 *
 * @param basePackage 默认包名
 * @param classes     存放字节码文件路径的集合
 * @return
 */
public static void listPackages(String basePackage, List<String> classes) {
    URL url = SophonUtils.class.getClassLoader()
            .getResource("./" + basePackage.replaceAll("\\.", "/"));
    File directory = new File(url.getFile());
    for (File file : directory.listFiles()) {
        // 如果是一个目录就继续往下读取(递归调用)
        if (file.isDirectory()) {
            listPackages(basePackage + "." + file.getName(), classes);
        } else {
            // 如果不是一个目录,判断是不是以.class结尾的文件,如果不是则不作处理
            String classpath = file.getName();
            if (".class".equals(classpath.substring(classpath.length() - ".class".length()))) {
                classes.add(basePackage + "." + classpath.replaceAll(".class", ""));
            }
        }
    }
}
```

**代码演示:**

> 使用方法非常简单,你只需要调用getInterfaceImpls()方法即可,listPackages()方法是个辅助。

```java
//
// getInstanceImpls()返回一个Class<?>对象数组
// 这个数组中包含的数据就是SophonInit接口的子类
//
ArrayList<Class<?>> subclass = getInterfaceImpls(SophonInit.class);
```

**ps:值的注意的地方是,这个方法只能获取项目中自己定义的接口,不能获取到JDK或者是其他Jar包中的接口,因为这个工具的原理就是扫描编译后的classes目录**