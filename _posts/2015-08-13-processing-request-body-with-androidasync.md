---
layout: post
title: PROCESSING REQUEST BODY WITH ANDROIDASYNC
published: true
---

###The Problem

In a <a href="http://programminglife.io/android-http-server-with-androidasync/" target="_blank"> previous post </a> I showed how to create a HTTP Server using AndroidAsync library.
Another challenge of using it would be how to parse the request body of a request coming from a REST client.

###Preparation
In order to test the code I recommend installing RESTClient for Firefox or any REST client for the browser that you use. This will help us send the request to our server that we have <a href="http://programminglife.io/android-http-server-with-androidasync/" target="_blank"> previously </a> created. 
Lets also send as a JSON request the example bellow:

```json
{"menu": {
  "id": "file",
  "value": "File",
  "popup": {
    "menuitem": [
      {"value": "New", "onclick": "CreateNewDoc()"},
      {"value": "Open", "onclick": "OpenDoc()"},
      {"value": "Close", "onclick": "CloseDoc()"}
    ]
  }
}}
```

###The Solution
Let’s add the following piece of code to the startServer() method that we created:

```java
server.post("/testJson", new HttpServerRequestCallback() {
    @Override
    public void onRequest(AsyncHttpServerRequest asyncHttpServerRequest, 
                          AsyncHttpServerResponse asyncHttpServerResponse) {
        AsyncHttpRequestBody requestBody = asyncHttpServerRequest.getBody();
        try {
            asyncHttpServerResponse.code(200);
            asyncHttpServerResponse.send(
                new JSONObject(requestBody.toString()).getJSONObject("menu").getString("value"));
        } catch (JSONException e) {
            Log.e("MainActivity", e.getMessage(), e);
        }
    }
});
```
Now go open your browser, enter the REST client and type in: your.tablet.ip:8080/testJSON, with HTTP method POST and in the request body paste the JSON object above. If all works well you should see in the Response field the value <b>File</b>.

###The Explanation
A question that may come up from the code above is why the JSON transformation? Well, JSON is a format that is very easy to work with and I find it very practical to work with JSON objects. On the other hand the method above is performant and makes sense only if you know for sure that you will receive JSON format on you request object. If not use the parsing methods in the <b> AsyncHttpRequestBody </b> class.

Please let me know if there is any other part of the AndroidAsync that you want me to cover in a future post.

You can also find the complete code <a href="https://github.com/andreivisan/AndroidAsyncHttpServer" target="_blank"> here </a>.
