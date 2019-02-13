IOC是什么？	
IoC is aimed at loosening the coupling of application components. The key concepts are: 
•	Components do not know each other directly. 
•	Components specify external dependencies using some kind of a key. 
•	Some "superior instance" (the IoC container, for example) resolves the dependencies once for each component and hereby "wires" the components together. 
这里说了三个关键点，组件与组件互相不认识（也就是经常听到解耦的概念），组件使用特殊的key来标识外部的依赖，一些高级的实例（比如IoC容器）解决组件之间相互依赖的问题并且将组件与组件之间连接起来。
对于我这种Java初学者来说，这几个概念看起来是一脸懵逼，概念不明白，那么就先来一个个解答心中的小疑问吧。
什么叫解耦？
	解耦这个概念很简单，但是需要了解一些背景，在一个对象A中引用另外一个对象B，传统的实现方式是在A中实例化一个B来使用：
	Public Class A(){
		B = new B();
}
这种情况下，就可以称作A是依赖于B的，因为如果B的实例化有任何变动，在A中都需要修改实例化的代码，在生产项目中对象之间将存在大量并且是多层的依赖关系，一个对象做修改之后可能会影响到几十个依赖此对象的其他对象；另外还有一点，在进行单元测试的时候，测试方法无法Mock，因为在测试对象当中存在对于其他对象的实例化；
解耦方法非常简单，在A中已参数形式引入一个B实例就可以了，A中不需要自己来对B进行实例化，这种方法也叫作依赖注入（Dependency Injection）
Public Class A(){
	Public setB(B b){
		This.b = b;
}
}
什么是IoC容器？容器如何实现控制反转的？
	在Spring当中的IoC容器可以用一张简单的图说明：
 
	之前我们说依赖注入就是直接在对象当中引用已经实例化了的其他对象，那么这个实例化就是IoC容器帮我们实现的，IoC容器抽取配置数据以及项目中的各种对象构建为可以直接使用的各个实例；
由此可见，项目中的所有对象只需要和IoC容器建立关系就好了，而传统的实现方式需要对象自己去实例化其他对象，对于程序的控制权一直在自己手上，有了IoC容器之后，在需要使用实例的时候将控制权交给IoC容器，由主动的控制变为被动控制，控制权反转了过来。
在Spring中如何实现依赖注入？
	作为一个Spring初学者刚刚开始深入了解依赖注入实现的时候，Google一搜，各种概念、示例铺天盖地看得我头都大了，Spring从最早的版本就已经支持依赖注入实现了，而现在已经演进到Spring Boot了，具体依赖注入的实现方法可以总结为三种类型：
①	基于XML配置文件的依赖注入
	②	基于XML+Java代码的依赖注入
	③	基于Java代码（注解）的依赖注入
	其实抛开实现方式的细节，从上面的IoC容器原理图我们可以看到，IoC容器要抽取配置数据以及项目中的各种对象，所以要配合IoC容器工作，需要给IoC容器提供配置数据以及将需要注入的各种对象注册到容器，使容器能够识别这些对象并且在实例化的时候能够获取到对象的声明。
什么是Bean？
	个人理解，Spring中的Bean就是能够被IoC容器用来实例化的对象。在最新的Spring Boot中大多已经使用注解的方式来配置、注入Bean
	这里简单记录一下几种常用注解的含义：
@Springbootapplication： 此注解包含@configuration、@enableautoconfiguration、@componentscan三个注解，一般会在main函数处进行注解，对于Spring Web APP来说就是在根目录的application.java的main函数里面用；
@configuration：表示这个class为多个beans的来源，一般会搭配@bean注解使用，在该class里面会有一系列工厂方法创建实例化对象（所谓工厂方法就是return各种实例对象）
@enableautoconfiguration：简单理解就是Spring框架自动化实现的功能，它能从当前Spring项目的各个依赖包里面扫描Bean，注册到IOC容器里面以备使用
@componentscan：表示Spring从指定的路径中扫描Bean来注册，这个注解里面是可以指定路径参数的，如果未指定则默认为根路径，比如在Main函数这里，就会扫描项目根目录下各个子目录中的Bean进行注册，各种Bean包括@Service、@component、@Repository、@controller、@Bean这些注解配置的Bean
@Service：相当于@component，但是表示该Bean位于Service层，同样的还有@controller表示bean为controller层，@repository表示bean在数据处理层（数据库交互）
@bean：@configuration下面的工厂方法的注解
@autowired：此注解表示自动注入一个实例

