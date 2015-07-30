---
layout: post
title: ANDROID HTTP SERVER WITH NANOHTTPD
published: true
---

###Introduction
In this post I will show you how to create a http server inside your Android application. You might ask yourself why would you ever create a web server inside your Android app which in most cases should serve as a client and not as a server. Well, one use case that I have encountered would be the one where we have used a robot arm to test our payment application together with a payment terminal. In this case the robot arm would operate on the terminal and would also send REST calls to the application to send and receive data. This would be one case, I can imagine there could be many other applications of creating a http server inside your Android application.

###The start
First let’s create a new Android project. I will use Gradle to build it but there are no dependencies. We could use nanoHTTPD as a JAR file but the whole idea behind it is that it is very light so the only thing we need to do is to copy and paste the class NanoHTTPD.java found in the /core folder in the GitHub repository. You can find the NanoHTTPD repository <a href=“https://github.com/NanoHttpd/nanohttpd”> here </a>.

###Create the server
In the /src/main/java/your_package_name folder let’s create a package called <b>server</b>. Inside this package let’s create a class called MyServer.java. Also copy and paste NanoHTTPD.java from GitHub to this folder. Use the code bellow in order to create it.

```java
import java.io.IOException;

/**
 * Created by andrei on 7/30/15.
 */
public class MyServer extends NanoHTTPD {
    private final static int PORT = 8080;

    public MyServer() throws IOException {
        super(PORT);
        start();
        System.out.println( "\nRunning! Point your browers to http://localhost:8080/ \n" );
    }

    @Override
    public Response serve(IHTTPSession session) {
        String msg = "<html><body><h1>Hello server</h1>\n";
        msg += "<p>We serve " + session.getUri() + " !</p>";
        return newFixedLengthResponse( msg + "</body></html>\n" );
    }
}
```

Now that we have the server created let’s modify MainActivity.java. First let’s add a global variable.

```java
private MyServer server;
```

Next we will add the following code to the onResume() method.

```java
@Override
public void onResume() {
    super.onResume();
    try {
        server = new MyServer();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```
I suggest you also close the server on the onPause() method. For the purpose of this demo I will not use it but please find bellow the implementation of the onPause() method.

```java
@Override
public void onPause() {
    super.onPause();
    if(server != null) {
        server.stop();
    }
}
```

Before running the app make sure that inside your AndroidManifest.xml you have the following permissions:

```xml
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"></uses-permission>
<uses-permission android:name="android.permission.INTERNET"></uses-permission>
```

After all the changes have been made we can run this demo. If you run the demo on a real device then make sure you are connected to the same wifi network as your development machine. Then, after the app starts, open the browser on your development machine and type http://your-phone-or-tablet-ip:8080/HELLO_WORLD and you should see the following:

<b>
HELLO SERVER

We serve /HELLO_WORLD !
</b>

In order to find your phone or tablet ip go to WiFi Setting and then to Advanced and there you should find the ip.
If you run on your emulator then open a browser window inside the emulator and type: http://localhost:8080/HELLO_WORLD and you should see the same as above.

You can find all the full project on GitHub <a href=“https://github.com/andreivisan/AndroidHttpServer”> here. </a>
