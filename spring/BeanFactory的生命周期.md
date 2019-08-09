# Spring Bean的生命周期和IOC容器初始化过程

####初始化过程

| **1.BeanFacotry加载Bean配置文件，将读到的Bean配置封装到BeanDefinition** |
| :----------------------------------------------------------- |
| **2.将封装好的BeanDefinition对象注册到BeanDefinition容器中** |
| **3.注册BeanPostProcessor 相关实现类到BeanPostProcessor容器中** |
| **4.BeanFactory进入就绪状态**                                |
| **5.外部调用BeanFacotry的getBean(String name)方法，BeanFactory着手实例化相关的bean** |
| **6.重复3 4 ，直到程序退出，BeanFactory被销毁**              |

#### 具体步骤：

1⃣️：Bean容器找到.xml配置文件中的Spring bean 的定义,利用反射创建一个Bean的实例。

2⃣️：如果涉及到一些属性值，利用Set方法设置一些属性值。

3⃣️：如果实现了BeanNameAware接口，调用setBeanName方法，传入Bean的名字，如果实现了	  BeanClassLoaderAware接口，调用setBeanClassLoader()方法，传入ClassLoader对象的实例。

4⃣️：如果实现了其他的*Aware接口，就调用相应的方法。

5⃣️：如果有和加载这个Bean的Spring容器相关的BeanPostProcessor对象，执行postProcessBeforeInitialization（）方法。

6⃣️：如果Bean实现了InitializingBean接口，执行afterPropertiesSet()方法。

7⃣️：执行配置文件中定义的Init-method属性的方法。

8⃣️：执行BeanPostProcessor.postProcessAfterInitalizaiton()方法。

9⃣️：当要销毁时，如果Bean实现了DisposableBean接口，调用destory()方法。

🔟：再执行配置文件中定义中指定的destroy-method方法。

![img](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-9-17/48376272.jpg)

中文配图：

![img](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-9-17/5496407.jpg)

#### 参考链接：

> **1. https://www.cnblogs.com/zrtqsk/p/3735273.html**
>
> **2.http://www.tianxiaobo.com/2018/01/18/自己动手实现的-Spring-IOC-和-AOP-下篇/**
>
> **3.《spring源码深度解析》**

