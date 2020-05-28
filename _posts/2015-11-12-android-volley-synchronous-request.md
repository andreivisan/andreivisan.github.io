---
layout: post
title: Android Volley synchronous requests
published: true
---

### Introduction

<a href="http://developer.android.com/training/volley/index.html" target="_blank"> Volley </a> is an HTTP library that makes networking for Android apps easier and most importantly, faster. One of the main features that Volley has is eliminating the need for using AsyncTask for network operations as well as allowing you to avoid using HttpUrlConnection and HttpClient. The main feature of Volley is that it is asynchronous which can lead to some issues when you want to do a synchronous request.

### The problem

One of the problems I faced using Volley was calling a webservice via its JsonObjectRequest inside an Android Library and wanting to return one of the values I received inside the response. But wait ... you'll say ... can't you do it via Futures? 
Well, here comes the second problem. But first let's have a look how dealing with Futures while using Volley would look like:

``` java
int REQUEST_TIMEOUT = 10;

RequestFuture<JSONObject> future = RequestFuture.newFuture();
JsonObjectRequest request = new JsonObjectRequest(URL, null, future, future);
requestQueue.add(request);

//...

try {
  JSONObject response = future.get(REQUEST_TIMEOUT, TimeUnit.SECONDS); // this will block (forever)
} catch (InterruptedException e) {
  // exception handling
} catch (ExecutionException e) {
  // exception handling
}
```

Let's analyze the code above. What we do is to create a RequestFuture and we pass it as parameter to our JsonObjectRequest which then we add to the request queue.
Then the interesting part comes along. In order to get the response JSON object from our request we call the get() method on the future we declared above. We also set a timeout of let's say 10 seconds so we don't block the UI indefinitely in case our request times out.
But here's the catch - we can't call ``` future.get(REQUEST_TIMEOUT, TimeUnit.SECONDS) ``` on the main thread (the UI thread) as it is a blocking operation and it will result in a TimeoutException. So in order to solve this issue we need to call ``` future.get(REQUEST_TIMEOUT, TimeUnit.SECONDS) ``` in an AsyncTask, which is not something that we want.

### The code

One solution I came up with was to create a callback that I will pass to the method that has the call to JsonObjectRequest so that when we call this method we can use the callback to get the request value and process it further.

Let's start by creating a new Android project and add the following lines to the AndroidManifest.xml file:

``` xml
<uses-permission android:name="android.permission.INTERNET"/>

<application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:theme="@style/AppTheme"
        android:name=".controllers.NetworkController">
```

We must add INTERNET permission in order to make the HTTP request and we added a Controller that will initialize the request queue.
Next let's add the code for the controller:

```java
public class NetworkController extends Application {

    private static final String TAG = NetworkController.class.getSimpleName();

    private RequestQueue mRequestQueue;

    private static NetworkController mInstance;

    @Override
    public void onCreate() {
        super.onCreate();
        mInstance = this;
    }

    public static synchronized NetworkController getInstance() {
        return mInstance;
    }

    public RequestQueue getRequestQueue() {
        if (mRequestQueue == null) {
            mRequestQueue = Volley.newRequestQueue(getApplicationContext());
        }
        return mRequestQueue;
    }

    public <T> void addToRequestQueue(Request<T> req, String tag) {
        req.setTag(TextUtils.isEmpty(tag) ? TAG : tag);
        getRequestQueue().add(req);
    }

    public <T> void addToRequestQueue(Request<T> req) {
        req.setTag(TAG);
        getRequestQueue().add(req);
    }

    public void cancelPendingRequests(Object tag) {
        if (mRequestQueue != null) {
            mRequestQueue.cancelAll(tag);
        }
    }

}
```

Next step is to create the method that holds our JsonObjectRequest:

```java
public void fetchData(final DataCallback callback) {
        String url = "your-url-here";

        JsonObjectRequest jsonObjectRequest = new JsonObjectRequest
                (Request.Method.GET, url, null, new Response.Listener<JSONObject>() {
                    @Override
                    public void onResponse(JSONObject response) {
                        Log.d(tag, response.toString());

                        try {
                            callback.onSuccess(response);
                        } catch (JSONException e) {
                            Log.e(tag, e.getMessage(), e);
                        }
                    }
                }, new Response.ErrorListener() {
                    @Override
                    public void onErrorResponse(VolleyError error) {
                        VolleyLog.d(tag, "Error: " + error.getMessage());
                    }
                });

        NetworkController.getInstance().addToRequestQueue(jsonObjectRequest);
    }
```

As you can see above we added a parameter to the fetchData method representing the callback. Let's implement it:

```java
public interface DataCallback {
    void onSuccess(JSONObject result);
}
```

Now let's call fetchData and use the response that comes from the webservice:

```java
public void useData() {
    fetchData(new DataCallback() {
        @Override
        public void onSuccess(JSONObject result) {
            try {
                data = result.getString("data");
            } catch (JSONException e) {
                Log.e(tag, e.getMessage(), e);
            }
        }
    });
}
```

### Conclusion

As awesome as Volley can be it still lacks in working with Futures and it's hard to use for synchronous work. The only advantage that Volley has is that it comes from Google and a lot of companies where security is the first priority choose to use it. If that is not an issue for you I recommend <a href="https://github.com/koush/ion" target="_blank">ion</a> which is a much better library to help you with HTTP calls.
If you have any questions or you need further clarification and/or help please don't hesitate to comment bellow.


