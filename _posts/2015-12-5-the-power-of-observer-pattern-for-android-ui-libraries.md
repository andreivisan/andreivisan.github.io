---
layout: post
title: The power of Observer pattern for Android UI libraries
published: false
---

### Introduction

### The Observer pattern

### The library

Our UI library will export an EditText field and a simple button that will post whatever we choose to write in our EditText. This is a simplified version of adding a post or a comment using a pre-defined component insinde a library to add the text we wish to post. The main purpose of this example is to emphasize the value of the Observer pattern when building a UI library.

First thing we need to create to get started is an Android project with an empty activity. Once this is done then we will create an Android library as a module inside our project. In Android Studio we can achieve this by right clicking on the project name and then New -> Module. In the selection window that pops up we will select Android library. I took the liberty to call mine just `uilibrary`.

Now if we take a look at our `settings.gradle` file inside our project we'll see that our library module was added as well. In order to be able to use it in our client project we will edit our `build.gradle` insinde the client project by adding the following line: `compile project(':uilibrary')`.

Let's create the view that we will export from our library to the client project. I created a new layout called `post_form.xml`. 

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">

    <EditText
        android:id="@+id/post_content"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <RelativeLayout
        android:id="@+id/post_button"
        android:layout_width="match_parent"
        android:layout_height="60dp"
        android:layout_gravity="bottom"
        android:background="@color/darkGrey"
        android:layout_marginTop="50dp">

        <TextView
            android:id="@+id/post_button_label"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="20sp"
            android:textColor="@color/white"
            android:layout_centerHorizontal="true"
            android:layout_centerVertical="true"
            android:text="Post content"/>

    </RelativeLayout>

</LinearLayout>
```

As you can see above I created a simple `EditText` and a `RelativeLayout` that will represent our post content button. Now let's continue with the code that represents the fragment that will inflate the view above.

#### Custom View

#### Custom EditText

### The client
