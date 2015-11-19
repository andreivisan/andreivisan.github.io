---
layout: post
title: Android media server with Node.js web client
published: false
---

### Introduction

In this post I am going to show you how you can create a media server on your Android device and stream media files, images and videos in our case, to a web client. The web client will be built using Node.js.

#### Motivation

One may wonder how would an application like this help or which use cases it may serve. There are a lot of cases where people like to build their own media servers and clients in order to store and manage their personal photos and videos. There are multiple ways in which this task may be achieved with the help of different cloud solutions out there, like Dropbox or Google Drive. But there are cases where you don't want your media in the cloud and you prefer to have it all stored on your personal server and just sending them via mail just won't do it for you. 

#### Technologies

In order to achieve this result I have used AndroidAsync for the server as well as Apache commons IO and Node.js with Express, ejs and request module.
You can find on my Github account the code for the <a href="https://github.com/andreivisan/AndroidVideoStreamServer" target="_blank"> server </a> as well as for the <a href="https://github.com/andreivisan/WebsocketClient" target="_blank"> client </a>.

### The Server

Let's start by diving into how the server side was built. In a <a href="http://programminglife.io/android-http-server-with-androidasync" target="_blank"> previous post </a> I have presented ways to make a server out of your Android device, but the response was just a simple JSON object. In this post we will dive a bit deeper into how to stream data via a HTTP server.
For this I suggest you start a simple blank activity Android project. You may wonder why not just a service that exposes all the necessary information. I suggest to go with a simple blank activity for the purpose of extending this project with more functionality, like choosing which types of files to expose or which folders, etc.
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

As you can see above, pretty much everything is just standard code and I have created the `startHttpServer()` which creates our server.

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

The code above is pretty straight forward. We create a HTTP server and we expose the file list we have just retrieved in the `FileUtil` class.

### The Client

For the client, create a Node.js project using Express.js and ejs for the view templates. Also install the request module by running `npm install --save request` in your project's folder. If you are completely new to Node.js you can read one of my <a href="http://programminglife.io/searching-with-elasticsearch-and-node.js/" target="_blank"> previous posts </a> for a step by step tutorial on how to start a new project or just `git clone` the <a href="https://github.com/andreivisan/WebsocketClient" target="_blank"> project from my Github account </a> and run `npm install` inside the project's folder.

#### The Client - Display files

Now that you have the Node.js project initialized let's start by adding the code to request the list of files from the Android device that we want to display and stream. For this add the following code to your `routes/index.js` :

```javascript
var request = require("request");

router.get('/', function(req, res) {
  var url = "http://192.168.2.40:8080/get-media"

  request({
    url: url,
    json: true
  },
  function(error, response, body) {
    console.log("Response received %s", JSON.stringify(response.body.files));
    res.render('index', {files: response.body.files});
  });

});
```

As you can see above I started by adding the url for the request. The IP I used is the IP of your Android device which you can retrieve by going to `Settings -> Wi-Fi -> on the top right menu choose Advanced` and at the bottom of the `Advanced` screen you can find the IP address of your Android device. I suggest you put this in an environment variable or on a configuration server. I hard coded it for the sake of simplicity. The port `8080` is the port I defined in `HttpServer.java` by declaring `httpServer.listen(mAsyncServer, 8080)`.
Now that we have retrieved the file list let's display it by changing the `views/index.ejs` as you can see bellow:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>File list</title>
    <link rel='stylesheet' href='/stylesheets/style.css' />
  </head>
  <body>

    <ul>
      <% files.forEach( function(file) { %>
          <li>
            <a href="/show-file?fileName=<%= file.fileName %>&extension=<%= file.extension %>">
              <%= file.fileName %>
            </a>
          </li>
      <% }) %>
      <li>
    </ul>

  </body>
</html>
```

If you are not familiar with ejs syntax you can also use other template technologies like <a href="http://jade-lang.com/" target="_blank"> Jade </a> or you can find more documentation <a href="http://ejs.co/" target="_blank"> here </a>.
Now if you start the app on the device and you start the Node.js server by running `npm start` inside the client project folder and then run `localhost:3000` in your browser you should see a list of all the files you have inside the Camera folder on your device SD card. If you click on any of them you'll receive an error since we have not yet defined a method that responds to `show-files` request.

#### The Server - Stream images

Now that we can see the list of files let's start creating the server side code to help us stream the pictures from our Android device. Let's add the code bellow to our `createHttpServer()` method inside the `HttpServer` class:

```java
httpServer.get("/get-image", new HttpServerRequestCallback() {
    @Override
    public void onRequest(AsyncHttpServerRequest request, AsyncHttpServerResponse response) {
        Log.i(tag, "Request for image received!");
        String fileName = request.getQuery().getString("name");
        response.setContentType("image/jpeg");
        response.send(fileUtil.base64EncodedImage(fileName));
    }
});
```

As you can see above we are sending the Base64 encoded bytes of the image to our web client. In order to obtain the Base64 encoded bytes we need to add the following method to our `FileUtil` class:

```java
public String base64EncodedImage(String fileName) {
    Bitmap bitmap = BitmapFactory.decodeFile("/sdcard/DCIM/Camera/" + fileName);
    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    bitmap.compress(Bitmap.CompressFormat.JPEG, 100, baos);
    byte[] bytes = baos.toByteArray();
    String encodedImage = Base64.encodeToString(bytes, Base64.DEFAULT);
    return encodedImage;
}
```

Now that the code for image streaming is in place let's proceed to do the same for video streaming.

#### The Server - Stream video

Same as for the image streaming let's add the code bellow to our `createHttpServer()` method inside the `HttpServer` class:

```java
httpServer.get("/get-video", new HttpServerRequestCallback() {
    @Override
    public void onRequest(AsyncHttpServerRequest request, AsyncHttpServerResponse response) {
        Log.i(tag, "Request for video received!");
        String fileName = request.getQuery().getString("name");
        response.setContentType("video/mp4");
        response.send(fileUtil.base64EncodedVideo(fileName));
    }
});
```

Same as in the image streaming case we need to Base64 encode the bytes of our video file. For this let's add the code bellow to our `FileUtil` class:

```java
public String base64EncodedVideo(String fileName) {
    String movieString = null;
    File sdCardMovie = new File("/sdcard/DCIM/Camera/"+fileName);
    try {
        byte[] videoBytes = FileUtils.readFileToByteArray(sdCardMovie);
        movieString = Base64.encodeToString(videoBytes, Base64.NO_WRAP);
    } catch (IOException e) {
        e.printStackTrace();
    }
    return movieString;
}
```

The code above is pretty straight forward but I want to mention that here I have used the Apache commons IO `FileUtils.readFileToByteArray` method to get the byte array of the video file.
Now that the server code is all ready and done let's finish the client code too so we can stream our media files.

#### The Client - Render images and videos

For the client side code let's add the following code to our `routes/index.js` file:

```javascript
router.get('/show-file', function(req, res) {
  console.log("Request: %s | %s", req.query.fileName, req.query.extension);
  if(req.query.extension.indexOf(".jpg") > -1) {
    var url = "http://192.168.2.40:8080/get-image?name=" + req.query.fileName;
    console.log("URL: %s", url);

    request({
      url: url,
      json: true
    },
    function(error, response, body) {
      console.log("Response received ");
      res.render('mediaPlay', { image:response.body, video:null});
    });
  } else {
    var url = "http://192.168.2.40:8080/get-video?name=" + req.query.fileName;
    console.log("URL: %s", url);

    request({
      url: url,
      json: true
    },
    function(error, response, body) {
      console.log("Response received ");
      res.render('mediaPlay', { image:null, video:response.body});
    });
  }
});
```

The code above is pretty straight forward as well. We check the extension of the file we clicked on and based on that we call the proper server method in order to get the correct media stream. In order to display the media file we click on I created a new view template called `mediaPlay` for which you can find the code bellow:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Media play</title>
    <link rel='stylesheet' href='/stylesheets/style.css' />
  </head>
  <body>

    <% if(image) { %>
      <div>
        <img width="356" height="200" src="data:image/jpeg;base64, <%= image %>" />
      </div>
    <% } %>

    <% if(video) { %>
      <div>
        <video width="356" height="200" controls >
          <source src="data:video/mp4;base64, <%= video %>" type="video/mp4" />
        </video>
      </div>
    <% } %>

  </body>
</html>
```
Please note that for both the image and vide display you have to use `data:image/jpeg;base64` or `data:video/mp4;base64` inside the `src` attribute since the stream we send to our web client is Base64 encoded.
Now that all the code is in place you need to start again the app on your Android device or emulator and also start the Node.js server and enter `localhost:3000` in your browser then click on the media file you want to stream. After a bit of waiting based on the file size, since the server is an Android device, you should see the content of the media file you clicked on. If you want the vide to start automatically then you need to add `autoplay` attribute in the `video` tag.

### Conclusion

I hope you find this post useful or at least informative and please let me know in the comments bellow if I can help you further or if you enjoyed the reading.
There are lots of improvements you can add to the code above specially performance wise. I will try to present in a future post how we can stream the videos or larger media files in chunks so that we don't have to wait for the full video to be downloaded from the Android device.
