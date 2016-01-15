---
layout: post
title: Improving Android Testing with Service Locator Design Pattern
published: false
---

### Introduction

Unit testing in Android is definitely a very challenging task. I find unit testing to be a critical component in raising the standard for the quality of any Android app. Besides being an Android developer, I am an Android user and I can objectively say that the quality of a big number of apps in the Google Play store is not as high as it should be and I think that a lot has to do with unit testing or the lack thereof.

The biggest challenge I found while writing unit tests in Android is injecting mock objects into Activities. There are a lot of cases where you build a service class to perform certain operations that you want to isolate from the Activity code and use in multiple places. This makes it very difficult for when, let's say, you want to test the functionality of the Activity and ignore the calls to the service methods. This can be achieved by injecting the mock service in the Activity, but in Android that is close to imposible since Android manages the creation and destruction of all activity instances, thereby making it difficult to inject dependencies. 

The solution that I came up with was to implement the Service Locator Design Pattern in order to set which service instance I want to use depending on the context.

Another solution would be to use <a href="http://square.github.io/dagger/" target="_blank"> Dagger </a> in order to make use of Dependency Injection, but I usualy prefer to add as few external libraries as possible in my apps.

My unit testing stack of choice at the moment of writing this article is JUnit, Robolectric and Mockito.

### Service Locator Design Pattern

As per Wikipedia definition, the Service Locator Pattern is a design pattern used in software development to encapsulate the processes involved in obtaining a service with a strong abstraction layer. This pattern uses a central registry known as the "service locator", which on request returns the information necessary to perform a certain task.

### Implementation

Let's assume we have a service interface that I will call `Service.java` and an implementation of it called `ServiceImpl.java`.

``` java 
public interface Service {
    void fetchInformation(String url, final MyCallback callback);
}
```

```java
public class ServiceImpl {
    void fetchInformation(String url, final MyCallback callback) {
        //Method implementation
    }
}
```

Now, lets implement the code for the Service Locator class that will provide our Service.

``` java
public class ServiceLocator {
    private static ServiceLocator instance;
    private static Service service;
    
    private ServiceLocator() {
    }
    
    public static ServiceLocator getInstance() {
        if(instance == null) {
            instance = new ServiceLocator();
        }
        return instance;
    }
    
    public static Service getService() {
        if(service != null) {
            return service;
        } else {
            return new ServiceImpl();
        }
    }

    public static void setService(Service service) {
        ServiceLocator.service = service;
    }
}
```

Now lets use this service locator to get our service in our Activity class.

``` java
public class MainActivity extends Activity {

    private Service service;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        //...
        service = ServiceLocator.getInstance().getService();
        //...
        service.fetchInformation(url, new MyCallback() {
            //...
        });
    }

}
```

Now that we have the Activity implementation, I would like to test its functionality without calling the `fetchInformation(url, myCallback)` method as that will be tested separately and I don't want it to bother me during the Activity testing. Let's see how our Service Locator Pattern implementation can help us for testing. Bellow is the implementation of our test class.

``` java
@RunWith(RobolectricGradleTestRunner.class)
@Config(constants = BuildConfig.class, sdk = 22, manifest = "src/main/AndroidManifest.xml")
public class MainActivityTest {

    private MainActivity mainActivity;
    private Service service;
    
    @Before
    public void setUp() throws Exception {
        super.setUp();
        
        service = mock(ServiceImpl.class);
        doNothing().when(service).fetchData(anyString(), (MyCallback)any());
        
        ServiceLocator.getInstance().setService(service);
        mainActivity = Robolectric.buildActivity(MainActivity.class).create().get();
        
        //...
    }
    
    @Test
    public void myTest() throws Exception {
        //...
    }

}
```

As you can see above, the service locator helped me inject the mock service into the Activity, so now the call to `fetchData(...)` will be ignored because of this line: `doNothing().when(service).fetchData(anyString(), (MyCallback)any());`.

### Conclusion

I find the Service Locator Pattern to be of huge help when writing unit tests and dealing with concepts like Dependency Injection and partial mocks. These concepts are very difficult to implement in Android and this patterns makes them accessible without introducing a third party library. If you need to use Dependency Injection for more than testing then I strongly suggest to use <a href="http://square.github.io/dagger/" target="_blank"> Dagger </a>.
Please let me know in the comments bellow if you found this post useful.



