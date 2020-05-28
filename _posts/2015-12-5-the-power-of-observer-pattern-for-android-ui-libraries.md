---
layout: post
title: The power of Observer pattern for Android UI libraries
published: true
---

### Introduction

In this post I want to emphasize the importance of the Observer pattern in creating custom UI components and/or custom UI libraries. Observer pattern is one of the most important design pattern, but we might not see its immediate benefits when we have to build Android projects although it is implemented in a lot of default components that we use. One of the most common is `View.OnClickListener()`.
The full code for this project can be found on my <a href="https://github.com/andreivisan/AndroidListeners" target="_blank"> GitHub account </a>.

### The Observer pattern

As the <a href="https://en.wikipedia.org/wiki/Design_Patterns" target="_blank"> Gof (Gang of Four) </a> states in their famous <a href="http://amzn.to/1llzMHz" target="_blank"> Design Patterns </a> book, the Observer pattern defines a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically. Observer pattern falls under behavioral pattern category.

### The library

Our UI library will export an EditText field and a simple button that will post whatever we choose to write in our EditText. This is a simplified version of adding a post or a comment using a pre-defined component inside a library to add the text we wish to post. The main purpose of this example is to emphasize the value of the Observer pattern when building a UI library.

First thing we need to create to get started is an Android project with an empty activity. Once this is done then we will create an Android library as a module inside our project. In Android Studio we can achieve this by right clicking on the project name and then New -> Module. In the selection window that pops up we will select Android library. I took the liberty to call mine just `uilibrary`.

Now if we take a look at our `settings.gradle` file inside our project we'll see that our library module was added as well. In order to be able to use it in our client project we will edit our `build.gradle` inside the client project by adding the following line: `compile project(':uilibrary')`.

Let's create the view that we will export from our library to the client project. I created a new layout called `post_form.xml`. 

``` xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">

    <io.programminglife.uilibrary.customcomponents.CustomEditText
        android:id="@+id/post_content"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <RelativeLayout
        android:id="@+id/post_button"
        android:layout_width="match_parent"
        android:layout_height="60dp"
        android:layout_gravity="bottom"
        android:background="@color/darkGrey"
        android:layout_marginTop="50dp"
        android:layout_marginBottom="300dp">

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

As you can see above I created a custom `EditText` called `CustomEditText` because I want to propagate the event that will hide my soft keyboard so that when this happens the whole view is closed. 

```java
public class CustomEditText extends EditText {

    private EditTextImeBackListener mOnImeBack;

    public CustomEditText(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);

    }

    public CustomEditText(Context context, AttributeSet attrs) {
        super(context, attrs);

    }

    public CustomEditText(Context context) {
        super(context);

    }

    @Override
    public boolean onKeyPreIme(int keyCode, KeyEvent event) {
        if (event.getKeyCode() == KeyEvent.KEYCODE_BACK) {
            if (mOnImeBack != null) {
                mOnImeBack.onImeBack(this, this.getText().toString());
            }
        }
        return super.dispatchKeyEvent(event);
    }

    public void setOnEditTextImeBackListener(EditTextImeBackListener listener) {
        mOnImeBack = listener;
    }

}
```

At this point we can already see the power of Observer pattern in our custom component. What I did was to create a custom listener called `EditTextImeBackListener` that contains one method `void onImeBack(CustomEditText ctrl, String text)` which gets triggered when the back button that closes the soft keyboard is pressed. Now, when we will implement this method in our fragment we can decide what action we want to take when the back button that closes the soft keyboard is pressed. In our case we will just close our view together with the keyboard.

Now let's continue with the code that represents the fragment that will inflate the view above.

```java
public class AddPostFragment extends Fragment {

    private CustomEditText mPostContent;
    private RelativeLayout mPostButton;
    private View mFragmentView;
    private Fragment mCurrentFragment;
    private InputMethodManager inputMethodManager;

    private PublishPostListener mPublishPostListener;

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        mFragmentView = inflater.inflate(R.layout.post_form, container, false);
        mCurrentFragment = this;

        mPostContent = (CustomEditText)mFragmentView.findViewById(R.id.post_content);
        mPostButton = (RelativeLayout)mFragmentView.findViewById(R.id.post_button);

        showInputKeyboard();
        initView();

        return mFragmentView;
    }

    @Override
    public void onPause() {
        super.onPause();
        inputMethodManager.hideSoftInputFromWindow(getView().getWindowToken(), 0);
    }

    private void showInputKeyboard() {
        inputMethodManager = (InputMethodManager)getActivity().
                                                    getSystemService(Context.INPUT_METHOD_SERVICE);
        inputMethodManager.toggleSoftInput(InputMethodManager.SHOW_FORCED, 
                                           InputMethodManager.HIDE_IMPLICIT_ONLY);
    }

    private void initView() {
        mPostContent.requestFocus();
        mPostContent.setOnEditTextImeBackListener(new EditTextImeBackListener() {
            @Override
            public void onImeBack(CustomEditText ctrl, String text) {
                getActivity().getSupportFragmentManager().popBackStack();
                getActivity().getSupportFragmentManager().beginTransaction().remove(mCurrentFragment).commit();
            }
        });

        mPostButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if(!TextUtils.isEmpty(mPostContent.getText())) {
                    mPublishPostListener.onPostPublished(mPostContent.getText().toString());
                }
            }
        });
    }

    public void setmPublishPostListener(PublishPostListener mPublishPostListener) {
        this.mPublishPostListener = mPublishPostListener;
    }
}
```

Above we have another demonstration of how powerful the Observer pattern can get when exporting custom components.
Let's first look at the way we implemented the pressing on the back button to dismiss the soft keyboard. As I said above we can implement this any way we want and in our case it makes perfect sense to just dismiss the whole view together with the soft keyboard.
Now let's look at another listener we have implemented and that is `PublishPostListener`. This listener has a method called `void onPostPublished(String postContent)` that will get triggered once we click on `Post content` button and will provide us with the text inside our custom `EditText`. Obviously you can make this more complex by adding on `onError` method that gets triggered for example when there is no text inside the `EditText`. But for the purpose of this post one method should do.

### The client

The `activity_main.xml` will look like the code bellow.

``` xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools" android:layout_width="match_parent"
    android:layout_height="match_parent" android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:paddingBottom="@dimen/activity_vertical_margin" tools:context=".MainActivity">

    <TextView
        android:id="@+id/posted_content"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

    <FrameLayout
        android:id="@+id/fragment_container"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"/>

</RelativeLayout>
```

The `TextView` will hold the content of our post and the `FrameLayout` is the fragment container which initially will hold our `Add post` button which on click will be replaced with our fragment from the UI library. The code for the `Add post` button is:

``` xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/add_post_button"
    android:layout_width="match_parent"
    android:layout_height="60dp"
    android:layout_gravity="bottom"
    android:background="@color/darkGrey"
    android:layout_marginTop="50dp"
    android:onClick="addPost">

    <TextView
        android:id="@+id/add_post_button_label"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="20sp"
        android:textColor="@color/white"
        android:layout_centerHorizontal="true"
        android:layout_centerVertical="true"
        android:text="Add post"/>

</RelativeLayout>
```

Next we have to build the `MainActivity.java` that will start our `AddPostFragment` and that will post whatever we choose to.

``` java
public class MainActivity extends FragmentActivity {

    private CreatePostFragment mCreatePostFragment;
    private TextView mPostContent;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mCreatePostFragment = new CreatePostFragment();
        mPostContent = (TextView)findViewById(R.id.posted_content);

        initView();
    }

    private void initView() {
        getSupportFragmentManager().
                beginTransaction().
                add(R.id.fragment_container, mCreatePostFragment).
                addToBackStack(null).
                commit();
    }

    public void addPost(View view) {
        AddPostFragment addPostFragment = new AddPostFragment();
        addPostFragment.setmPublishPostListener(new PublishPostListener() {
            @Override
            public void onPostPublished(String postContent) {
                mPostContent.setText(postContent);
            }
        });

        getSupportFragmentManager().
                beginTransaction().
                replace(R.id.fragment_container, addPostFragment).
                addToBackStack(null).
                commit();
    }
}
```

The part that we have to focus on is the `public void addPost(View view)` which is the method that is called on click on the `Add post` button. What this method does is that it initializes the `AddPostFragment` and sets the `PublishPostListener`. On the `public void onPostPublished(String postContent)` we get the post content and we set it on our `TextView`.

### Conclusion

I hope the post above shows how important the Observer pattern is when we have to create and/or export custom UI components and libraries. Please leave a comment bellow if you find this post useful.
The full code for this project can be found on my <a href="https://github.com/andreivisan/AndroidListeners" target=_blank"> GitHub account </a>.
