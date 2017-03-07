---
layout: post
title: Dynamic Factory Pattern for Android
published: true
---


### Introduction

The Dynamic Factory Pattern describes a factory that can create service instances based on concrete type definitions stored in external plugins which can be later loaded without having to modify the factory class or the core implementation.

### Context

You are building a framework that is composed of different modules that can be fully or partially included in a project. Your framework core is not aware of the contents of the underlying modules in order to offer full flexibility and modularity. The implementing system should be able to implement the framework and later add new modules without having to change the core implementation of the framework.

### Example

The framework system has a rule evaluation module. Each rule implements a well defined interface, and is injected into a container that evaluates it.
The framework vendor supplies a fixed set of modules. New modules could be added by simply implementing the core module interface. The problem comes at module instantiation, since the factories that contain the logic for creating instances of the module services may need to be modified to support new modules.

### Problem

How can we define an interface for creating objects that implement a given contract without tying it to concrete implementations of these contracts?

### Achievements

By implementing the Dynamic Factory Pattern we are trying to achieve the following results:

  1. <b>Flexibility</b>. The implementers of the products should be easily modifiable, even when the system is running, allowing the injection of new product types into an existing system.
  2. <b>Extensibility</b>. New product types should be easily added without requiring neither a new factory class nor modifying any existing one.
  3. <b>Controlled Evolution</b> : users can create new types of products conforming to the product interface, but providing unanticipated behavior or features.
  4. <b>Agility</b>. New types of products should added to the system in a quick and agile manner, avoiding reworking of a factory class any time a new concrete product is created.
  5. <b>Simplicity</b>. The client interface should be simple, hiding from the client the complex details of dynamic product creation.
  
### Solution

The dynamic factory pattern is a generalized implementation that is responsible for creating instances, while not making any a priori decisions about the concrete types of those instances.

![Dynamic Factory Pattern](/public/images/Dynamic_Factory_Pattern.png)

In order to achieve this and for simplicity, in the core module (called loader in the GitHub example you can find bellow) I created an enum that holds the name of each module (plugin). Each field contains a method called getServiceName() which returns the full name for the service implementations in each module.

NOTE: Enum was used for the sake of simplicity, for a more robust solution I recommend a stable persistence solution.
----

Plugins.java

```java
public enum Plugins {

    PLUGIN1 {
        @Override
        public String getServiceName() {
            return "io.programminglife.plugin1.Plugin1Service";
        }
    },

    PLUGIN2 {
        @Override
        public String getServiceName() {
            return "io.programminglife.plugin2.Plugin2Service";
        }
    },

    PLUGIN3 {
        @Override
        public String getServiceName() {
            return "io.programminglife.plugin3.Plugin3Service";
        }
    };

    public abstract String getServiceName();

}
```

I also created a service factory, which based on the service name defined above instantiate the appropriate implementation.

ServiceFactory.java

```java
public class ServiceFactory {

    public PluginService getService(String serviceName) throws ClassNotFoundException, 
                                 IllegalAccessException, InstantiationException {
        return (PluginService) Class.forName(serviceName).newInstance();
    }

}
```

The ```PluginService.java``` interface implementation is pretty basic as you can see bellow:

```java
public interface PluginService {

    String action();

}
```

In the demo app I created, I built 3 methods each containing a call to the appropriate module. In case the module is loaded the implementation will return a text that is set on a text view. In case the module is not loaded ```Plugin is not loaded``` text is set on the text view. Bellow I show you the implementation of one method, for the full source code please find the link to the Github repository bellow.

```java
private void callPlugin1() {
    try {
        pluginService = factory.getService(Plugins.PLUGIN1.getServiceName());
        plugin1Text.setText(pluginService.action());
    } catch (ClassNotFoundException e) {
        plugin1Text.setText("Plugin 1 not loaded");
    } catch (IllegalAccessException e) {
        plugin1Text.setText("Plugin 1 not loaded");
    } catch (InstantiationException e) {
        plugin1Text.setText("Plugin 1 not loaded");
    }
}
```

Please find bellow the ```Plugin 1``` implementation of the ```PluginService``` interface:

```java
public class Plugin1Service implements PluginService {

    private static final String TAG = "Plugin1Service";


    @Override
    public String action() {
        return "Plugin 1 activated";
    }

}
```

You can find the sources for this solution on my Github at [this link](https://github.com/andreivisan/DynamicModuleLoading). 

### Conclusion

The Dynamic Factory Pattern adds flexibility and better modularity to an application by abstracting the creation process of instances, exposing it through a well-known entity in the system, and storing all the details about the concrete implementers of the products in metadata. This removes the creation code from the application, hiding it in the factory and the creation metadata.
Additionally, it makes easier to introduce new implementers of a contract in a system, since the location implementation details are encoded in metadata.




