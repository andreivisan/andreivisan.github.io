---
layout: post
title: Android media server with Node.js web client
published: false
---

### Introduction

In this post I am going to show you how you can creade a media server on your Android device and stream media files, images and videos in our case, to a web client. The web client will be built using Node.js.

#### Motivation

One may wonder how would an application like this help or which use cases it may serve. There are a lot of cases where people like to build their own media servers and clients in order to store and manage their personal photos and videos. There are multiple ways in which this task may be achieved with the help of different cloud solutions out there, like Dropbox or Google Drive. But there are cases where you don't want your media in the cloud and you prefer to have it all stored on your personal server and just sending them via mail just won't do it for you. 

#### Technologies

In order to achieve this result I have used AndroidAsync for the server as well as Apache commons IO and Node.js with Express, ejs and request module.
You can find on my Github account the code for the <a href="https://github.com/andreivisan/AndroidVideoStreamServer" target="_blank"> server </a> as well as for the <a href="https://github.com/andreivisan/WebsocketClient" target="_blank"> client </a>.

### The Server

Let's start by diving into how the server side was built. In a <a href="http://programminglife.io/android-http-server-with-androidasync" target="_blank"> previous post </a> I have presented ways to make a server out of your Android device, but the response was just a simple JSON object. In this post we will dive a bit deeper into how to stream data via a HTTP server.
For this I suggest you start a simple blank activity Android project. You may wonder why not just a service that exposes all the necessary infromation. I suggest to go with a simple blank activity for the purpose of extending this project with more functionality, like choosing which types of files to expose or which folders, etc.
On the `MainActivity.java` on the `onCreate` method I called `startHttpServer()`. Please find bellow the complete code for the `MainActivity.java`

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Snackbar.make(view, "Replace with your own action", Snackbar.LENGTH_LONG)
                        .setAction("Action", null).show();
            }
        });

        startHttpServer();
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        int id = item.getItemId();

        if (id == R.id.action_settings) {
            return true;
        }

        return super.onOptionsItemSelected(item);
    }

    private void startHttpServer() {
        HttpServer httpServer = new HttpServer();
        httpServer.createHttpServer();
    }
}
```

As you can see above, preety much everything is just standard code and I have created the `startHttpServer()` which creates our server.

#### The Server - Expose media files

Next step was to display all the media file on the SD card. Obviously, you can expose all the media files from all the folders on your device, but for the sake of simplicity I chose to only expose these.
For this I started by creating a class called `FileUtil.java` to which I added the following method:

```java
public JSONArray getMediaFilesList() {
    JSONArray mediaFilesList = new JSONArray();
    File sdCardMediaFolder = new File("/sdcard/DCIM/Camera/");
    try {
        if(sdCardMediaFolder.isDirectory()) {
            File[] files = sdCardMediaFolder.listFiles();
            for(int i = 0; i < files.length; i++) {
                JSONObject fileJson = new JSONObject();
                fileJson.put("fileName", files[i].getName());
                fileJson.put("extension", getFileExtension(files[i].getName()));
                mediaFilesList.put(i, fileJson);
            }
        }
    } catch (JSONException e) {
        e.printStackTrace();
    }
    return mediaFilesList;
}
```

So, as you can see above, I am getting all the contents of the Camera folder from my SD card and creating a JSON array in which I put a JSON object consisting of the file name and the extension of the files in the Camera folder.
As you can see for the extension part I created a special method called `getFileExtensions(String fileName)` which takes the file name as a parameter. Please find bellow the code for this method:

```java
private String getFileExtension(String fileName) {
    String extension = null;
    int lastIndexOfDot = fileName.lastIndexOf(".");
    extension = fileName.substring(lastIndexOfDot, fileName.length());
    Log.i(tag, "File " + fileName + " has extension " + extension);
    return extension;
}
```

Now, the next step is to expose this file list with the help of our HTTP server. For this I have created a class called `HttpServer` for which you can see the code bellow.

```java
public class HttpServer {

    private static final String tag = HttpServer.class.getSimpleName();

    private FileUtil fileUtil;

    public HttpServer() {
        fileUtil = new FileUtil();
    }

    public void createHttpServer() {
        AsyncHttpServer httpServer = new AsyncHttpServer();
        AsyncServer mAsyncServer = new AsyncServer();

        httpServer.get("/get-media", new HttpServerRequestCallback() {
            @Override
            public void onRequest(AsyncHttpServerRequest request, AsyncHttpServerResponse response) {
                try {
                    Log.i(tag, "Request for file system received!");
                    JSONArray mediaFilesArray = new FileUtil().getMediaFilesList();
                    JSONObject mediaFiles = new JSONObject();
                    mediaFiles.put("files", mediaFilesArray);
                    Log.i(tag, "Response object: " + mediaFiles);
                    response.send(mediaFiles);
                } catch (JSONException e) {
                    e.printStackTrace();
                }
            }
        });

        httpServer.listen(mAsyncServer, 8080);
    }

}
```

The code above is preety straight forward. We create a HTTP server and we expose the file list we have just retrieved in the `FileUtil` class.

### The Client

For the client, create a Node.js project using Express.js and ejs for the view templates. Also install the request module by running `npm install --save request` in your project's folder. If you are completely new to Node.js you can read one of my <a href="http://programminglife.io/searching-with-elasticsearch-and-node.js/" target="_blank"> previous posts </a> for a step by step tutorial on how to start a new project or just `git clone` the <a href="https://github.com/andreivisan/WebsocketClient" target="_blank"> project from my Github account </a> and run `npm install` inside the project's folder.

#### The Client - Display files

#### The Server - Stream images

#### The Server - Stream video

#### The Client - Render images and videos

### Conclusion
