---
layout: post
title: Learning Spring Boot by building a concrete project - Part 2
published: true
---

### Part 1 summary

In <a href="http://programminglife.io/learning-spring-boot-by-building-a-concrete-project-part-1/" target="_blank"> Part 1</a> of this tutorial we created a basic structure of our Personal Finance project by using Spring Initializr to configure the package name and dependencies for our project. 
Import it into your IDE of choice and try running it. If you followed the steps I described there you should have the following error:

```
APPLICATION FAILED TO START


Description:

Cannot determine embedded database driver class for database type NONE

Action:

If you want an embedded database please put a supported one on the classpath. 
If you have database settings to be loaded from a particular profile you may need to active it (no profiles are currently active).

Process finished with exit code 1
```

This means that we didn't give Spring Boot enough information to auto-configure a ```DataSource```. 

### Making your application run

If you want to make your application run without configuring any data sources you have to add the following line to your ```FinanceApplication.java``` file:  ```@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class, HibernateJpaAutoConfiguration.class})```. Now your class should look like this:

``` java
@SpringBootApplication
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class,HibernateJpaAutoConfiguration.class})
public class FinanceApplication {

   public static void main(String[] args) {
      SpringApplication.run(FinanceApplication.class, args);
   }

}
```

We also added Spring Session and Spring Security as dependencies. We will comment them out so that we can run or app. Your ```pom.xml``` file should look like this:

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>

   <groupId>io.programminglife</groupId>
   <artifactId>finance</artifactId>
   <version>0.0.1-SNAPSHOT</version>
   <packaging>jar</packaging>

   <name>finance</name>
   <description>Demo project for Spring Boot</description>

   <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>1.5.2.RELEASE</version>
      <relativePath/> <!-- lookup parent from repository -->
   </parent>

   <properties>
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
      <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
      <java.version>1.8</java.version>
   </properties>

   <dependencies>
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-actuator</artifactId>
      </dependency>
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-data-jpa</artifactId>
      </dependency>
      <!--dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-security</artifactId>
      </dependency>
      <dependency>
         <groupId>org.springframework.session</groupId>
         <artifactId>spring-session</artifactId>
      </dependency-->
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-thymeleaf</artifactId>
      </dependency>
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-web</artifactId>
      </dependency>

      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-test</artifactId>
         <scope>test</scope>
      </dependency>

      <dependency>   
         <groupId>mysql</groupId>   
         <artifactId>mysql-connector-java</artifactId>   
         <scope>runtime</scope>
      </dependency>
   </dependencies>

   <build>
      <plugins>
         <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
         </plugin>
      </plugins>
   </build>

</project>
```

NOTE: I also added the ```mysql-connector``` dependency.

### Configuring the data source

To configure the datasource I chose ClearDB for Heroku. If you want to learn how you can do that check <a href="https://devcenter.heroku.com/articles/cleardb" target="_blank"> here </a>.

Now we need to configure the following properties inside your ```application.properties``` file in order for Spring Boot to configure our datasource.

```
spring.datasource.url=
spring.datasource.username=
spring.datasource.password=
```

Once you have configure it, go into your Heroku app for which you have this plugin setup, Settings menu and click on ```Reveal Vars``` in order to get the DB Url. 

For username and password, go back to your app dashboard on Heroku and under ```Installed add-ons``` click on your ClearDB MySQL ignite instance. You will be redirected to the ClearDB dashboard. Under ```My Databases``` click on the name of your database and choose ```Endpoint Information``` tab. Here you will find your database username and password.

In order to finish the datasource integration we need to create the model for our database. In this tutorial we will only create the ```User``` model in order to get our application working and in the next part we will complete  the model creation and security … so stay tuned.

Let’s create a ```model``` package and inside it the ```User.java``` class:

``` java
@Entity@Table(name = "user")
public class User {

    @Id    
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "user_id")
    private int id;

    @Column(name = "email")
    @Email(message = "*Please provide a valid Email")
    @NotEmpty(message = "*Please provide an email")
    private String email;

    @Column(name = "password")
    @Length(min = 5, message = "*Your password must have at least 5 characters")
    @NotEmpty(message = "*Please provide your password")
    @Transient    
    private String password;

    @Column(name = "name")
    @NotEmpty(message = "*Please provide your name")
    private String name;

    @Column(name = "last_name")
    @NotEmpty(message = "*Please provide your last name")
    private String lastName;

    @Column(name = "active")
    private int active;

    public int getId() {
        return id;    }

    public void setId(int id) {
        this.id = id;    }

    public String getEmail() {
        return email;    }

    public void setEmail(String email) {
        this.email = email;    }

    public String getPassword() {
        return password;    }

    public void setPassword(String password) {
        this.password = password;    }

    public String getName() {
        return name;    }

    public void setName(String name) {
        this.name = name;    }

    public String getLastName() {
        return lastName;    }

    public void setLastName(String lastName) {
        this.lastName = lastName;    }

    public int getActive() {
        return active;    }

    public void setActive(int active) {
        this.active = active;    }

}
```

Now let’s update ```application.properties``` and make it look like this:

```
# ===============================
# = DATA SOURCE# 
# ===============================
spring.datasource.url=
spring.datasource.username=
spring.datasource.password=

# ===============================
# = JPA / HIBERNATE# 
# ===============================
spring.jpa.show-sql = true
spring.jpa.hibernate.ddl-auto = update
spring.jpa.hibernate.naming-strategy = org.hibernate.cfg.ImprovedNamingStrategy
spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.MySQL5Dialect
```

NOTE: Don’t forget to update with your database credentials. 

Now you can run again the app and go to ```http://localhost:8080``` and you can see again the ```Hello World``` message.

In the next post we will finish our JPA implementation and we will cover security as well. In the meantime you can find the sources for this tutorial on <a href="https://github.com/andreivisan/finance" target="_blank"> my GitHub </a>.

