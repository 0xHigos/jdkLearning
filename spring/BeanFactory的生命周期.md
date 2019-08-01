# BeanFactory的生命周期

| **1.BeanFacotry加载Bean配置文件，将读到的Bean配置封装到BeanDefinition** |
| :----------------------------------------------------------- |
| **2.将封装好的BeanDefinition对象注册到BeanDefinition容器中** |
| **3.注册BeanPostProcessor 相关实现类到BeanPostProcessor容器中** |
| **4.BeanFactory进入就绪状态**                                |
| **5.外部调用BeanFacotry的getBean(String name)方法，BeanFactory着手实例化相关的bean** |
| **6.重复3 4 ，直到程序退出，BeanFactory被销毁**              |

#### 具体示例：![img](https://images0.cnblogs.com/i/580631/201405/181453414212066.png)

![img](https://images0.cnblogs.com/i/580631/201405/181454040628981.png)

#### 参考链接：

> **1. https://www.cnblogs.com/zrtqsk/p/3735273.html**
>
> **2.http://www.tianxiaobo.com/2018/01/18/自己动手实现的-Spring-IOC-和-AOP-下篇/**
>
> **3.《spring源码深度解析》**

