## Beans
A spring bean is normal java class that is instantiated and managed by the Spring IoC container.

### How beans are created
Beans are instantiated by the container based on the configuration metadata developer provides which can be using
- XML configuration(Legacy approach)
- Annotations (@Component, @Service)
- Java based configurations (via @Configuration and @Bean methods)

### Bean lifecycle management
The spring IoC container manages entire lifecycle of bean including its creation,initialization,scope and destruction

It also ensures any required dependencies are injected into bean as needed

## Beanfactory
The IoC container is represented by the beanfactory interface which provides the basic functionalities for managing beans.
Suitable for lightweight applications where performance is critical and fewer features are needed

## SpringContext (application context)
- Extends beanfactory
- Adds enterprise features
	- i18n support
	- Application events: publish/listen to events inside spring
	- Resource loading: Read resources from classpath,filesystem,URL
	- Environment and profiles
	- Autoscan and annotations
	- AOP support

```java
@Configuration
public class ProjectConfig{

@Bean
Parrot parrot(){
	var p=new Parrot();
	p.setName("koko");
	return p;
}

@Bean
String hello(){
	return "hello";
}

@Bean
Integer ten(){
	return 10;
}
}
```

```java
public class Main{
	public static void main(String[]args){
		var context=new AnnotationConfigApplicationContext(ProjectConfig.class);
		Parrot p=context.getBean(Parrot.class);
		System.out.println(p.getName());
		String s=context.getBean(String.class);
		System.out.println(s);
		Integer n=context.getBean(Integer.class);
		System.out.println(n);
	}
}
```

Running the app now, the values of the three beans will be printed in the console, as shown in the next code snippet.

```bash
koko
hello
10
```

When we create an java object with new() operator directly as shown below then your Spring Context/IoC container will not have any clue of the object

```java
Vehicle v=new Vehicle();
```

@Configuration is a Spring annotation used to mark class as a source of bean definitions
@Bean annotation lets Spring know that it needs to call this method when it initialises its context  and adds returned object/value to the spring IoC container

Java best practice is to put verbs in method names because method generally represent actions but for methods we use to add beans in the Spring Context we don't follow this convention

## What is NoUniqueBeanDefinition exception?
When multiple beans of the same type are defined in the spring context,and we attempt to retrieve bean by type,Spring cannot determine which instance to provide. This ambiguity results in NoUniqueBeanDefinition exception

```java
@Bean
Vehicle vehicle1(){
	var veh=new Vehicle();
	veh.setName("Audi");
	return veh;
}

@Bean
Vehicle vehicle2(){
	var veh=new Vehicle();
	veh.setName("Honda");
	return veh;
}

@Bean
Vehicle vehicle3(){
	var veh=new Vehicle();
	veh.setName("Ferrari");
	return veh;
}
```

To solve this ambiguity problem you need to refer precisely to one of the instance by using Bean's name

```java
public class Main{
	public static void main(String[] args){
		var context=new AnnotationConfigApplicationContext(ProjectConfig.class);
		Vehicle p=context.getBean("vehicle2",Vehicle.class);
		System.out.println(p.getName());
	}
}
```

## Different ways to name a bean
By default spring will consider method name as bean name. But if we have custom requirement to define a separate bean name,then we can use any of the below approaches

```java
@Bean(name="audiVehicle")
Vehicle vehicle1(){
	var veh=new Vehicle();
	veh.setName("Audi");
	return veh;
}

@Bean(value="hondaVehicle")
Vehicle vehicle2(){
	var veh=new Vehicle();
	veh.setName("Honda");
	return veh;
}

@Bean("ferrariVehicle")
Vehicle vehicle3(){
	var veh=new Vehicle();
	veh.setName("Ferrari");
	return veh;
}
```

```java
public class Main{
	public static void main(String[] args){
		var context=new AnnotationConfigApplicationContext(ProjectConfig.class);
		Vehicle veh1=context.getBean("audiVehicle",Vehicle.class);
		System.out.println(veh1.getName());
		Vehicle veh2=context.getBean("hondaVehicle",Vehicle.class);
		System.out.println(veh2.getName());
		Vehicle veh3=context.getBean("ferrariVehicle",Vehicle.class);
		System.out.println(veh3.getName());
	}
}
```

Output:
```bash
Audi
Honda
Ferrari
```

## Giving multiple names to a bean(bean aliasing)
Sometimes you may want single bean to be known by multiple names. This is called bean aliasing. You can define aliases by passing multiple names in the @Bean annotation using a string array. Now the same bean can be accessed by any of these names - audivehicle,audi,myfavvehicle

```java
@Bean({"audivehicle","audi","myfavvehicle"})
Vehicle vehicle(){
	var veh=new Vehicle();
	veh.setName("audi");
	return veh;
}
```

Adding a description to a bean
Sometimes it is useful to add short description for a bean

```java
@Bean
@Description
Vehicle vehicle(){
	var veh=new Vehicle();
	veh.setName("Audi");
	return veh;
}
```
