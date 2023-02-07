 git clone https://github.com/RedHatTraining/AD221-apps.git

 https://mirror.openshift.com/pub/openshift-v4/clients/camel-k/1.6.10/camel-k-client-1.6.10-linux-64bit.tar.gz

 sudo cp kamel /usr/local/bin/
sudo chmod +x /usr/local/bin/kamel
kamel version

# Chapter 2.  Creating Camel Routes
- Red Hat provides an Apache Camel extension pack for Visual Studio Code that includes multiple extensions related to Camel development. The pack enables code completion for both Java Domain-specific Language (DSL) and XML DSL. 
### Maven Configuration
- Fuse applications are typically built with Maven. To access artifacts that are in Red Hat Maven repositories, you must add those repositories to Maven's settings.xml file in the .m2 directory of your home directory. The system-level settings.xml file at M2_HOME/conf/settings.xml is used if a user specific file is not found. 

```
<?xml version="1.0"?>
<settings>

  <profiles>
    <profile>
      <id>extra-repos</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <repositories>
       <repository>
            <id>redhat-ga-repository</id>
            <url>https://maven.repository.redhat.com/ga</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
        <repository>
            <id>redhat-ea-repository</id>
           <url>https://maven.repository.redhat.com/earlyaccess/all</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
        <repository>
          <id>jboss-public</id>
          <name>JBoss Public Repository Group</name>
          <url>https://repository.jboss.org/nexus/content/groups/public/</url>
        </repository>
      </repositories>
      <pluginRepositories>
        <pluginRepository>
            <id>redhat-ga-repository</id>
            <url>https://maven.repository.redhat.com/ga</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </pluginRepository>
        <pluginRepository>
            <id>redhat-ea-repository</id>
            <url>https://maven.repository.redhat.com/earlyaccess/all</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </pluginRepository>
        <pluginRepository>
          <id>jboss-public</id>
          <name>JBoss Public Repository Group</name>
          <url>https://repository.jboss.org/nexus/content/groups/public</url>
        </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>
  <activeProfiles>
    <activeProfile>extra-repos</activeProfile>
  </activeProfiles>
</settings>
```
### Maven Support for Spring Boot
-  In order to build Spring Boot applications for Fuse, the Fuse Bill of Materials (BOM) is required. The BOM defines a curated set of Red Hat supported dependencies from the Red Hat Maven repository. The BOM exploits Maven's dependency management mechanism to define the appropriate versions of Maven dependencies. The BOM is provided by the following pom.xml configuration
```
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.jboss.redhat-fuse</groupId>
      <artifactId>fuse-springboot-bom</artifactId>
      <version>${fuse.version}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```
- In addition to the BOM, the Spring Boot Maven Plugin is also required. The Spring Boot Maven Plugin implements the build process for a Spring Boot application in Maven. This plugin is responsible for packaging your Spring Boot application as an executable Jar file. The Spring Boot maven plugin is provided by the following pom.xml configuration.

```
<plugin>
    <groupId>org.jboss.redhat-fuse</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <version>${fuse.version}</version>
    <executions>
	  <execution>
		<goals>
		  <goal>repackage</goal>
		</goals>
	  </execution>
    </executions>
</plugin>
```
### Spring Boot
- Red Hat Fuse includes a Spring Boot Starter module. With this module, you can use Camel in Spring Boot applications by using starters.
```
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-spring-boot-starter</artifactId>
    <version>${camel.version}</version> <!-- use the same version as your Camel core version -->
</dependency>
```
- With Spring Boot, you can add classes with your Camel routes, by annotating a RouteBuilder class with the org.springframework.stereotype.Component annotation. The annotation allows Spring Boot to find the class, register it as a Java Bean, and start a camel context that includes the route. 

```
package com.example;

import org.apache.camel.builder.RouteBuilder;
import org.springframework.stereotype.Component;

@Component
public class MyRoute extends RouteBuilder {

    @Override
    public void configure() throws Exception {
        from("timer:foo").to("log:bar");
    }
}
```
### Camel Routes
- In Camel, a route describes the path of a message from one endpoint (the origin) to another endpoint (the destination). The origin of a route is associated with the from method in the Java DSL and normally consumes messages from a source.
- The from method uses an integration component configured as a consumer endpoint. Likewise, the destination is associated with the to method in the Java DSL and produces, or send messages to a destination. The to method uses an integration component configured as a producer endpoint.
- Routes are a critical aspect of Camel because they define integration between endpoints. With the help of components, routes can move, transform, and split messages. Traditionally, integration implementation requires lots of complicated and unnecessary coding. With Camel, routes are defined in a few simple, human-readable lines of code in either Java DSL or XML DSL.
- A route starts with a consumer, which receives the data from a point of origin. With Camel, consider that the consumer is referring to where and how the initial message is being picked up. The origin determines which type of consumer endpoint Camel component is used, such as a location on the file system, a JMS queue, or even a tweet from Twitter. The route then directs the message to the producer, which sends data to a destination. By abstracting the integration code, developers can implement Enterprise Integration Patterns (EIPs) that manipulate or transform the data within the Camel route without requiring changes to either the origin or the destination.

### Components
- One of the most compelling reasons to use Camel is for the library of over 180 components. Each component typically has an exhaustive set of options that allow you to customize how the component interacts with the origin or destination.

1. File component: read from or write to a file system
2. JMS component: reads from or writes to a Java Messaging Service
3. FTP component: integrates with the File Transfer Protocol
4. Scheduler component: Generates messages on a given schedule

- The Direct component is a Camel core component that can be used to create consumer or producer endpoints for receiving and sending messages within the same CamelContext. External systems cannot send messages to direct component endpoints. In this example XML DSL route, the from element is using the direct component to receive messages from other routes running within the same CamelContext as this route. The log_body context provided in the uri attribute specifies the identity for this direct component. Another route can send a message to this direct component, within the same CamelContext by using a producing direct component with the same uri value.

```
<route id="XML DSL route">
  <from uri="direct:log_body"/>
  <log message="Message body: ${body}"/>
  <to uri="mock:next_service"/>
</route>

```
- The producer in the to element in this route, is using a mock component. The mock component is used to make testing routes easier, by simulating a real component. The mock component is often used when a real component is not available.

- A complete coverage of all components is out of scope for this course. It only takes a general understanding of how to use components, however, to be able to use any of the other 180 components. All of them conform to the same usage pattern. Refer to the Camel documentation for complete coverage on any component and all the options available for each component.

### Endpoints
- A Camel endpoint consists of a component and a URI. The URI defines how the component is used to consume new messages from an origin or produce exchange messages to a destination. The syntax of the URI endpoint consists of three parts: the scheme, the context path, and the options.
```
URI syntax:  scheme:context_path?options
```
example: 
```
ftp://services.lab.example.com?username=delete=true&include=order.*xml
```
- In this example URI, the scheme instructs Camel to use the ftp component. The context path of services.lab.example.com provides the address of the ftp service to use. After the ? two options are specified and separated by the & character to provide additional details for how the component is to be used.

- Each Camel route must have a consumer endpoint and can have multiple producer endpoints. The most powerful way of creating the routes is via Camel's Java Domain-specific Language (DSL)

### Java DSL Routes

- Java DSL routes in Camel are created by extending the org.apache.camel.builder.RouteBuilder class and overriding the configure method. A route is composed of two endpoints: a consumer and a producer. In Java DSL, this is represented by the from method for the consumer and the to method for the producer. Inside the overridden configure method, the from and to methods are used to define the route. The org.springframework.stereotype.Component annotation enables Spring Boot to recognize that this class is providing a Camel Route.

```
@Component
public class FileRouteBuilder extends RouteBuilder {

    @Override
    public void configure() throws Exception {

        from("file:orders/incoming")
        .to("file:orders/outgoing");
    }
}

```
- In this example, the consumer uses the file component to read all files from the orders/incoming path. Likewise, the producer also uses a file component to send the file to the orders/outgoing path.

- Inside a RouteBuilder class, each route can be uniquely identified by using a routeId method. Naming a route makes it easy to verify route execution in the logs, and also simplifies the process of creating unit tests. Add additional calls to the from method to create multiple routes in the configure method.

```
public void configure() throws Exception {

   from("file:orders/incoming")
   .routeId("route1")
   .to("file:orders/outgoing");

   from("file:orders/new")
   .routeId("routefinancial")
   .to("file:orders/financial");
}

```
- Each component can specify endpoint options to further configure how the component should function at that endpoint. The endpoint options are listed after the ? character as illustrated in the next example. Refer to the Camel documentation for component-specific attributes.

```
public void configure() throws Exception {

   from("file:orders/incoming?include=order.*xml") 1
   .to("file:orders/outgoing/?fileExist=Fail"); 2
}
```
1. The route consumes only XML files with a name starting with order.
2. The producer component throws an exception if a given file already exists.


- In addition to Java DSL, routes can be created via XML DSL files. Java DSL is a richer language to work with because you have the full power of the Java language at your fingertips. Often, messages require customized handling that is beyond the scope of Camel routes. Java provides an elegant solution as discussed later in this course. Also, some Java DSL features, such as value builders (for building expressions and predicates), are not available in the XML DSL.

On the other hand, using XML DSL routes gives a convenient alternative for externalizing route configurations.

### XML DSL Routes 

- Method names from the Java DSL map directly to XML elements in the Spring DSL in most cases. However, due to syntax differences between Java and XML, sometimes the name and the structure of elements are different in Spring DSL. Refer to Camel documentation for the correct structure of the route methods.

- To use the Spring DSL with Spring Boot, declare a routes element, using the custom Camel Spring namespace, inside an XML configuration file located in a camel folder on the Java classpath. Inside the routes element, declare one or more route elements starting with a from element and usually ending with a to element. These from and to elements are similar to the Java DSL from and to methods.

```
<routes xmlns="http://camel.apache.org/schema/spring">
  <route id="XML example">
    <from uri="file:orders/incoming"/>
    <to uri="file:orders/outgoing"/>
  </route>
</routes>

``
### REFERENCES

- https://camel.apache.org/camel-spring-boot/3.12.x/spring-boot.html

-----------------------------------

# Guided Exercise: Creating Routes Using the Java and XML DSL

- step1:  Open the project's POM file, and add the following dependencies:
```
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-spring-boot-starter</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
    <dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-test-spring</artifactId>
    <scope>test</scope>
</dependency>

```

- step2: Create the Java DSL route by updating the SchedulerRouteBuilder class.
1. Open the SchedulerRouteBuilder.java file.
2. Notice that the use of the org.springframework.stereotype.Component annotation is the marker that tells Spring Boot to pick up this class and use it to add a route to the CamelContext.
3. Enable the route by extending the RouteBuilder superclass.
```
***import org.apache.camel.builder.RouteBuilder;***
...
public class SchedulerRouteBuilder extends ***RouteBuilder***

```
4. Set up the Scheduler component.
- The scheduler component comes with Camel's core library. In this exercise we are using the scheduler to generate message exchanges every 2 seconds. The name, myScheduler, is an arbitrary name assigned to this instance of the component.

```
public void configure() throws Exception {
  ***from("scheduler:myScheduler?delay=2000")***

```
5. Set up a route ID.

- Each route can be uniquely identified by using a routeId method. Naming a route makes it easy to verify route execution in the logs, and also simplifies the process of creating unit tests.

```
public void configure() throws Exception {
    from("scheduler:myScheduler?delay=2000")
       ***.routeId("Java DSL route")`***
```
6. Add an exchange header.
- The scheduler component is generating exchange messages with empty bodies. In this step of the route, use the setBody method and a simple expression to add the message creation timestamp to the body of the message. The simple expression in this example copies the time value from a message header field.
```
public void configure() throws Exception {
    from("scheduler:myScheduler?delay=2000")
        .routeId("Java DSL route")
        ***.setBody().simple("Current time is ${header.CamelTimerFiredTime}")***
```                

7. Log a message to the console with the Log method.
- Add a log step to the route. The log helps with tracking the progression of the message through the routes.

```
public void configure() throws Exception {
        from("scheduler:myScheduler?delay=2000")
            .routeId("Java DSL route")
            .setBody().simple("Current time is ${header.CamelTimerFiredTime}")
         ***.log("Sending message to the body logging route")***
```            
8. Add a producer with the Direct component.

- For the final step of this route, use the direct component to create a producer. The message produced by the to method in this route is consumed by the matching direct endpoint, which is created in the XML route discussed next in this lab.

```
public void configure() throws Exception {
    from("scheduler:myScheduler?delay=2000")
        .routeId("Java DSL route")
        .setBody().simple("Current time is ${header.CamelTimerFiredTime}")
        .log("Sending message to the body logging route")
     ***.to("direct:log_body");***

```    

- step: 3 : Add an XML DSL route to receive the messages produced by the Java DSL route. Create the XML route by updating the camel-context.xml file.

note: Although the course focuses on the Java DSL, this step uses the XML DSL for illustrative purposes. You could create the same route by using the Java DSL.

1. Open the src/main/resources/camel/camel-context.xml file.

  - By default, Spring Boot inspects the classpath for a folder called camel. Any XML files with routes defined in that folder are automatically discovered and added to the CamelContext by Spring Boot.

2. Add the XML DSL route.

```
<route id="XML DSL route">
  <from uri="direct:log_body"/>
  <log message="Message body: ${body}"/>
  <to uri="mock:next_service"/>
</route>

```
3. Test the route.
 - Run the ./mvnw clean package spring-boot:run command to start the Spring Boot application.
 - Observe that the Java DSL route sends exchanges to the XML DSL route, and that the timestamps in the log messages differ by 2 seconds for every exchange.

 ```
 Java DSL route  : Sending message to the body logging route
XML DSL route   : Message body: Current time is Thu Dec 02 17:39:33 EST 2021
Java DSL route  : Sending message to the body logging route
XML DSL route   : Message body: Current time is Thu Dec 02 17:39:35 EST 2021

```
----------------------------------------

# Reading and Writing Files

### Introducing the File and FTP Components

- Use the File and FTP Camel components to integrate your application with file systems, by reading and writing files. In particular, use the File component to work with local file systems. The FTP component, on the other hand, interacts with files in remote servers via the file transfer protocol (FTP) and its secure variants.

- The following class is an example of a Camel route that uses the file and ftp components. The route downloads files from an FTP server to a local directory.

```
public class FileRouteBuilder extends RouteBuilder {

    @Override
    public void configure() throws Exception {
        from("ftp://localhost:21/documents")
        .to("file:downloads/docs");
    }

}

```
### The File Component
- Use the file component to read files from the file system or write files to the file system. To use the file component, you must specify the URI endpoint as follows:
```
file:directoryName
```
- The directoryName part is required. It specifies the base directory to use for the file endpoint.
- Additionally, you can specify endpoint options. For a complete list of endpoint options, refer to the File component documentation.


- For example, to copy files from one directory to another, you can use the following implementation:


```
public class FileRouteBuilder extends RouteBuilder {

    @Override
    public void configure() throws Exception {

        from("file:orders/incoming") 1
        .to("file:orders/outgoing"); 2

    }
}

```
1. The file component reads all the files from the orders/incoming directory.
2. The file component writes all the files to the orders/outgoing directory.

### Filtering Files

- The file component requires a directory as the only endpoint parameter. If you want to process a specific file, either as the origin or destination of a route, then you must use the fileName option, as the following example shows.

```
from("file:datasets?fileName=covid_cases.csv")
.to("file:tmp/data/");

```
- Likewise, you can configure the endpoint to include a subset of files, by using the filter option. The following example demonstrates a file component endpoint that only reads product json files.
```
from("file:warehouse/incoming?include=product.*json")
.to("file:warehouse/outgoing");

```
note: The include parameter supports regex patterns, both for the file and ftp components.


- Message Headers
    - Similar to other components, the file component provides producer and consumer headers in the message. For example, you can use the CamelFileLastModified header to inspect the last modified time of each consumed file.\

```
from("file:orders/")
.log("File: ${header.CamelFileLastModified}")

```
#  The FTP Component
- Use the ftp component to read files from and write files to an FTP server. The URI endpoint for this component is as follows:


```
ftp://[username@]hostname[:port]/directoryName

```
1. The component allows the use of ***ftp***, ***sftp***, and **ftps** protocols.
2. The*** hostname*** parameter is required, but the username and port are optional. You can also specify the username as an endpoint query option.
3. The ***directoryName*** parameter must be a relative directory path.
4. Similar to the ***file ***component, you can include further options in the endpoint. For example, you can use options to specify additional connection parameters, such as the authentication password.

```
from(
    "ftp://localhost:21/documents?" +
    "username=myuser&password=mypass"
)
.to("file:docs/");

```
- The preceding Camel route reads files from the ***documents*** directory in an FTP server and copies the files into the ***docs*** directory of the local file system. Likewise, you can use the **ftp **component as a producer, to write files to an FTP endpoint.

## Installing the FTP component
- The ftp component is not included in camel-core. To use this component, you must specify the camel-ftp artifact as a Maven dependency, as follows:


```
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-ftp</artifactId>
</dependency>
```
## Filtering Files

- Similar to the ***file*** component, you can use the*** fileName*** and ***include*** parameters to select or filter specific files in an FTP server. For example, you can select a subset of the files as the following example shows:
```
from(
    "ftp://localhost:21/documents?" +
    "username=myuser&password=mypass&" +
    "include=recipe.*txt"
)
.to("file:docs/recipes");

```
## FTP Connection Mode
- By default, the ***ftp*** component uses the FTP **active** connection mode. In this mode, the FTP server can initiate requests to the client, which means that the client must be publicly accessible to the FTP server. If the FTP server cannot reach the client to start a connection, then you must switch to the FTP passive mode. In passive mode, only the client starts the connection to the FTP server.
- To activate the passive mode, set the ***passiveMode*** endpoint option to **true**, as follows:

```
from(
    "ftp://localhost:21/documents?" +
    "username=myuser&password=mypass&" +
    "include=recipe.*txt&" +
*** "passiveMode=true"***
)
.to("file:docs/");

```
## Message Headers

- The ***ftp ***component provides producer and consumer headers for each message. For example, you can use the ***CamelFileName*** header to inspect the produced or consumed file name.

```
from("ftp://localhost:21/?include=record.*txt&")
***.log("File: ${header.CamelFileName}")***
.to("file:records");

```








