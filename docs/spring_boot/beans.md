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

## @Primary annotation
When multiple beans of the same type exist in the spring context you can designate one as the default using @Primary annotation

The Primary bean is the one spring automatically selects when multiple candidates are available and no specific bean name is provided

```java
@Bean
@Primary
Parrot parrot2(){
	var p=new Parrot();
	p.setName("Miki");
	return p;
}
```

## Using the @Import annotation
@Import allows one @Configuration class to include bean definitions from another configuration class. It helps organise and reuse configurations easily. ConfigB automatically imports all beans from ConfigA

```java
@Configuration
public class ConfigA{
	@Bean
	public A a(){
		return new A();
	}
}
```

```java
@Configuration
@Import(ConfigA.class)
public class ConfigB{
	@Bean
	public B b(){
		return new B();
	}
}
```
It is also possible to import multiple config classes at a time
ex: @Import({ConfigA.class,ConfigB.class})

```java
public static void main(String[]args){
	var ctx=new AnnotationConfigApplicationContext(ConfigB.class);
	A a =ctx.getBean(A.class);
	B b=ctx.getBean(B.class);
}
```

## Using @Component annotation
@Component is one of the most commonly used stereotype annotations in spring. It simplifies process of creating and registering beans in spring context with minimal code compared to @Bean method. By applying @Component to a class spring automaticaly detects and manages it as a spring bean

To ensure Spring scans and registers these annotated classes you can use @ComponentScan annotation over a configuration class

```java
package com.example.beans;
@Component
public class Vehicle{
	//other code
}
```

```java
@Configuration
@ComponentScan(basePackages="com.example.beans")
public class ProjectConfig{
	//other code
}
```

## @PostConstruct annotation

@PostConstruct is a Spring annotation used to run method automatically after spring bean is created and its dependencies are injected

### When is @PostConstruct used?
- To perform setup logic when a bean is created
- To load data,setup default values and execute startup logic
- To initialise resource after dependency injection

### When does @PostConstruct run?
- It runs after the constructor after all dependencies are injected
- It runs only once in bean's lifecycle

```java
@Component
public class Vehicle{
	private String name;
	@PostConstruct
	public void initialize(){
		this.name="Honda"
	}
}
```

- Spring creates vehicle bean when application starts
- Before using the bean it calls initalize method automatically
- This is useful for setting up initial data or configurations

## @PreDestroy annotation

@PreDestroy is a annotation used to run a method automatically before spring bean is destroyed

### When is @PreDestroy used?
- To perform cleanup logic before bean is destroyed
- To close resources such as database connections,file handles or threads
- To release memory or stop background tasks
- To log or notify that bean is being shutdown

### When does @PreDestroy run?
- It runs just before the Spring container destroys the bean
- It is called after the application context is closed
- It runs only once in bean's lifecycle

```java
@Component
public class Vehicle{

@PreDestroy
public void destroy(){
	System.out.println("Vehicle destroyed");
}
}
```

## Difference between @Bean and @Component

| Bean                                                                                                   | Component                                                                                         |
| ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------- |
| One or more instances of a class can be added to spring context                                        | Only one instance of a class can be added to spring context                                       |
| We can create an object instance of any type of class including present inside library like String etc | We can create an object instance for application class only which are created by dev team         |
| Usually we need to write more code like separate methods to create beans instances                     | Bean instances can be created with very less code like using @Component on top of the class       |
| Developer will have full control in creating and configuring the bean                                  | Developer will not have any control in creating and configuring the bean                          |
| Spring framework creates the bean based on the instructions and values provided by the developer       | Spring Framework takes charge of creating the bean and post that developer will have access to it |


## Layers in backend application

### Controller/Presentation layer
- Receives requests from the client (e.g. frotnend,mobiile app,postman)
- Calls the service layer for business logic
- Returns responses (usually JSON or XML)

### Service/Business logic layer
- Implements business rules,validations and transformations
- Calls the repositories to fetch or store data
- Handles transaction management if required

### Repository/Data Access layer
- Handles data persistence (CRUD operations)
- Uses Spring data JPA or JDBC to interact with databases
- Performs queries using JPQL or native SQL

## Spring stereotype annotations
Special annotations that automatically registers spring beans in the application context. These annotations help simplify bean creation and management

@Component-> Generic bean (used for any class)

@Service -> Business logic layer

@Repository -> Data Access layer

@Controller -> handles incoming web requests in spring MVC

@RestController -> Simplified @Controller,returns JSON responses to build REST APIs

Use @ComponentScan to enable automatic scanning of these annotations

## Bean wiring inside spring
In java web applications objects often delegates task to other objects creating dependency between them. Similarly in Spring when we define multiple beans,some beans may rely on others to function correctly. Bean wiring is the process of connecting these dependent beans within spring IoC container

### No wiring scenario inside spring
```java
public class Person{
	private String name;
	private String Vehicle;
}
```

```java
public class Vehicle{
	private String name;
}
```

```java
@Bean
public Vehicle vehicle(){
	Vehicle vehicle=new Vehicle();
	vehicle.setName("Toyota");
	return vehicle;
}

@Bean
public Person person(){
	Person person=new Person();
	person.setName("Lucy");
	return person;
}
```

Vehicle doesn't belong to any person. The person and vehicle bean present in context but no relation established

### Wiring using method call
```java
@Bean
public Vehicle vehicle(){
	Vehicle vehicle=new Vehicle();
	vehicle.setName("Toyota");
	return vehicle;
}

@Bean
public Person person(){
	Person person=new Person();
	person.setName("Lucy");
	vehicle.setName(vehicle());
	return person;
}
```

### Wiring beans using method params

```java
@Bean
public Vehicle vehicle(){
	Vehicle vehicle=new Vehicle();
	vehicle.setName("Toyota");
	return vehicle;
}

@Bean
public Person person(Vehicle veh){
	Person person=new Person();
	person.setName("Lucy");
	vehicle.setVehicle(veh);
	return person;
}
```

### What is AutoWiring?
Autowiring is a feature in Spring that automatically injects dependencies
- It eliminates need for manual object creation using new keyword
- Spring finds the right dependency and injects it automatically
#### Ways to Autowire in Spring
- By field injection (@AutoWired on field)
- By Setter injection (@AutoWired on setter method)
- By constructor injection (@AutoWired on constructor)

#### Autowiring using field injection (@AutoWired on field)

Spring looks for matching bean type and injects it automatically
```java
@Component
class Engine{}

@Component
class Car{
	@AutoWired
	private Engine engine;
}
```

- Hard to unit test
	- You cannot easily create objects and set dependencies yourself
	- Because the field is private and has no setter you must rely on spring
	- In tests it becomes difficult to mock replace dependencies

- No way to make the field final
	- If Spring injects via field you cannot write
		- `private final AccountService accountService;`
	- So,you cannot gurantee dependency is always available.It may be accidentally changed later
- Can create NullPointerExceptions
	- If spring fails to inject,the object can still be created,and you might get nullpointerexception at runtime

@Autowired(required=false) will help to avoid the NoSuchBeanDefinitionException if the bean is not available during autowiring process

#### Autowiring using the setter injection

Spring injects dependency using the setter method. Useful when dependency is optional
```java
@Component
class Engine{}

@Component
class Car{
	private Engine engine;
	
	@AutoWired
	public Car(Engine engine){
		this.engine=engine;
	}
}
```

- Has same drawbacks as field injection
- Less commonly used compared to constructor injection

#### Autowiring using constructor injection
- Preferred approach in modern spring applications
- Works well with immutability and unit testing
```java
@Component
class Engine{}

@Component
class Car{
	private final Engine engine;
	@AutoWired
	public void setEngine(Engine engine){
		this.engine=engine;
	}
}
```

#### Circular dependencies
A circular dependency occurs when two or more beans depend on each other directly or indirectly. Spring cannot resolve dependencies and throws an UnSatisfiedDependencyException.

```java

@Component
class Car{
	private final Engine eng;
	
	@AutoWired
	public Car(Engine engine){
		this.engine=engine;
	}
}

@Component
class Engine{
	private final Car car;
	
	@AutoWired
	public Engine(Car car){
		this.car=car;
	}
}
```

#### @Primary annotation

When multiple beans match for autowiring by type @Primary gives one bean higher preference so it's chosen automatically as the default

```java
@Component("petrolEngine")
class PetrolEngine implements Engine{}

@Component("dieselEngine")
@Primary
class DieselEngine implements Engine{}

@Component
class Car{

	@AutoWired
	private Engine engine;
}
```

#### @Qualifier annotation
If multiple beans of the same type exist,we use @Qualifier to specify the exact bean. 

```java
@Component("petrolEngine")
class PetrolEngine implements Engine{}

@Component("dieselEngine")
@Primary
class DieselEngine implements Engine{}

@Component
class Car{

	@AutoWired
	@Qualifier("dieselEngine")
	private Engine engine;
}
```

#### @Primary vs @Qualifier

| Annotation | When to Use?                                                    | How it works                                                                   |
| ---------- | --------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| @Primary   | When you have default bean that should be used most of the time | Spring automatically selects Primary bean if no specific qualifier is provided |
| @Qualifier | When you need to explicitly specify which bean to use           | Overrides @Primary and allows selecting a specific bean when multiple exist    |
