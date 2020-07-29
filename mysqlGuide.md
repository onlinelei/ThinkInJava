# 一、Spring





# 二、bean生命周期

Spring Bean 生命周期比较复杂，可以分为创建和销毁两个过程。

首先，创建 Bean 会经过一系列的步骤，主要包括：

实例化 Bean 对象。
设置 Bean 属性。
如果我们通过各种 Aware 接口声明了依赖关系，则会注入 Bean 对容器基础设施层面的依赖。具体包括 BeanNameAware、BeanFactoryAware 和 ApplicationContextAware，分别会注入 Bean ID、Bean Factory 或者 ApplicationContext。
调用 BeanPostProcessor 的前置初始化方法 postProcessBeforeInitialization。
如果实现了 InitializingBean 接口，则会调用 afterPropertiesSet 方法。
调用 Bean 自身定义的 init 方法。
调用 BeanPostProcessor 的后置初始化方法 postProcessAfterInitialization。
创建过程完毕。

<img src="https://gitee.com/suqianlei/Pic-Go-Repository/raw/master/img/20200727184747.png" style="zoom: 50%;" />


