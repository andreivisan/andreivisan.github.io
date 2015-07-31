---
layout: post
title: ANDROID HTTP SERVER WITH ANDROIDASYNC
published: true
---

Yesterday, in a previous post, I wrote about how to write an HTTP server inside your Android application using <a href="https://github.com/NanoHttpd/nanohttpd" target="_blank"> NanoHTTPD </a>. After a bit more research I found that <a href="https://popcorntime.io/" target="_blank"> Popcorntime </a> for Android were using NanoHTTPD as well and that they have replace it with another library called <a href="https://github.com/koush/AndroidAsync" target="_blank">AndroidAsync</a>.

After reading a bit about <a href="https://github.com/koush/AndroidAsync" target="_blank">AndroidAsync</a> I found out that the code is much better than the one for <a href="https://github.com/NanoHttpd/nanohttpd" target="_blank"> NanoHTTPD </a> as well as the way of handling requests is more clear from the code reading perspective.

###Let’s start coding
Before starting the exciting coding make sure you add the following code to your AndroidManifest.xml file:

```xml
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"></uses-permission>
<uses-permission android:name="android.permission.INTERNET"></uses-permission>
```

For this project I also used Gradle. So in order to add <a href="https://github.com/koush/AndroidAsync" target="_blank">AndroidAsync</a> to your project add the following line to your build.gradle file:

```java
compile 'com.koushikdutta.async:androidasync:2.+'
```

Now let’s modify MainActivity.java accordingly:

```java
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.Menu;
import android.view.MenuItem;

import com.koushikdutta.async.AsyncServer;
import com.koushikdutta.async.http.server.AsyncHttpServer;
import com.koushikdutta.async.http.server.AsyncHttpServerRequest;
import com.koushikdutta.async.http.server.AsyncHttpServerResponse;
import com.koushikdutta.async.http.server.HttpServerRequestCallback;


public class MainActivity extends AppCompatActivity {

    private AsyncHttpServer server = new AsyncHttpServer();
    private AsyncServer mAsyncServer = new AsyncServer();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        int id = item.getItemId();
        if(id == R.id.action_settings) {
            return true;
        }
        return super.onOptionsItemSelected(item);
    }

    @Override
    public void onResume() {
        super.onResume();
        startServer();
    }

    private void startServer() {
        server.get("/", new HttpServerRequestCallback() {
            @Override
            public void onRequest(AsyncHttpServerRequest request, AsyncHttpServerResponse response) {
                response.send("Hello!!!");
            }
        });
        server.listen(mAsyncServer, 8080);
    }
}
```

Now start the application and after go to your device’s browser or to your emulator’s browser and type http://localhost:8080/ and you should see:
<b>
Hello!!!
</b>

I assume the code above is pretty self explanatory and it doesn't need too much description. But what I did was to add a new AsyncHttpServer and handled a GET request to which we respond with the String "Hello!!!". Also we made our server to listen on port 8080. You can choose any port you want.

I suggest you create a separate class that handles all the server methods. In this case I have added it in the MainActivity.java for convenience purposes.
In order to stop the server you have to call the following methods:

```java
server.stop();
mAsyncServer.stop();
```

That’s about it. If you need further examples using <a href="https://github.com/NanoHttpd/nanohttpd" target="_blank"> NanoHTTPD </a> or <a href="https://github.com/koush/AndroidAsync" target="_blank">AndroidAsync</a> please let me know in the comments bellow and will come with more in detail examples.
You can find the full project on GitHub <a href="https://github.com/andreivisan/AndroidAsyncHttpServer" target="_blank"> here. </a>
