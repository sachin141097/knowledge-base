## Why Core Java alone isn't enough for web applications

**Database connection management**

```java
Connection conn=null
try{
    conn=DriverManager.getConnection(url,username,password);
    PreparedStatement stmt = conn.preparedStatement("SELECT * FROM products");
    ResultSet rs = stmt.executeQuery();
    //process results
} catch(SQLException e){
    e.printStackTrace();
} finally{
    if(conn!=null)conn.close();
}
```

Every single database operation requires this boilerplate code. If you forgot to close the connection you would have memory leaks. You need to manage connection pools manually


**Object creation and Dependencies**
**Configuration Management**
		You need to manually load property files
		
**Creating REST APIs**
		With Plain Java you would use servelets. Then configure everything in web.xml,handle url mappings,manage servlet lifecycle

The core problems with Java alone
- Too much boilerplate code: You spend more time on plumbing than business logic
- Manual object management: You are responsible for creating and wiring everything
- Tight coupling: Hard to test,Hard to maintain
- Configuration nightmare: XML files everywhere
- Reinventing the wheel: writing the same patterns over and over

## Introduction to Spring

### What is Spring?
Spring is a popular java framework for building web applications. It helps developers write clean,modular and testable code

Provides built in features for handling databases,security,messaging and more

## How spring boot changed everything

Traditional spring application
- Configure dispatcher servlet
- Set up component scanning
- Configure data source
- Setup view resolvers
- Configure server settings

Spring Boot simplified all of that with
- Auto configuration: it detects what you have in your project and configures it automatically
- Embedded servers: No need to deploy Tomcat manually- Just run your app like any Java program


## Maven

Before tools like Maven existed:
- Developers had to manually search the internet for the jar files
- Download them one by one
- Place them inside folder like lib/
- Add them manually to classpath

And if one jar depend on 10 more jars?more downloading

Maven automates all of this

You simply tell Maven your project's requirements (dependencies) in one file called

pom.xml (POM- Project Object Model)

Inside pom.xml you write something like

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<version>x.x.x</version>
</dependency>
```

Every library has a unique address
- GroupId: Company/Organisation (like the city)
- ArtifactId: Project name (like the street)
- Version: Specific number (like the house number)

Maven will
- Search for the library
- Download it automatically
- Download all its required library (called transitive dependencies)
- Keep them in correct classpath
No manual effort needed

Maven downloads dependencies from the central place called the Maven Central Repository

Once downloaded these libraries are stored on your machine in a Maven folder called local repository

If another project needs the same Jar
- Maven simply reuses it locally
- No need to download again

![](/docs/spring_boot/maven_project_structure.png)

This standard structure ensures

- Any developer can understand Maven project instantly
- Tools and IDE know exactly where to find things

Maven is not just dependency manager. It is also build automation tool. It can
- Compile code
- Run tests
- Package jar/war
- Deploy artifacts
- Clean old builds

`mvn clean` Removes old compiled files
- This deletes target folder
- To avoid conflicts with older builds
- To ensure fresh start

`mvn validate` checks if your project is correctly configured. It checks
- is pom.xml valid?
- Are all required fields present
- Is the project structure correct?
- Are all necessary plugins available?

`mvn compile` Compiles our java source code
This reads files from `src/main/java` and generates .class files into `target/`

`mvn test` runs all unit tests

`mvn package` packages the project into jar or war. Output is generated in the target folder
(`target/myapp-0.0.1.SNAPSHOT.jar`)This is the file we can run or deploy

`mvn verify` It ensures build is ready for final installation or deployment
It builds your project,tests it and verifies everything is working correctly without actually deploying it

`mvn install` It compiles->tests->packages->verify then installs the final .jar into local maven repository. This allows other projects on your machine to reuse it

## Inversion of Control (IoC)
Inversion of control (IoC) is a software design principle that defines how objects are created and managed in a program. It doesn't create object itself but outlines way for their creation and dependency management

With IoC the control flow of the program is reversed,Instead of programmer managing the flow of application,a framework or service takes over this responsibility,ensuring objects and their dependencies are handled automatically

-  Normally in java we create objects manually using new keyword
- With IoC spring manages object creation and dependencies automatically
- It reduces manual coding and makes application more flexible

## Dependency Injection

It is a design pattern used to implement inversion of control. DI is a way of providing dependencies to a class instead of creating them inside a class. Helps in bringing loose coupling between objects. Spring automatically injects required objects

### Advantages of DI and IoC
- Loose coupling between components
- Minimizes the amount of code
- Makes unit testing easier with different mocks
- Increased system maintainability and module resuability
- Allow concurrent and independent development
- Replacing modules has no side effect on other modules

