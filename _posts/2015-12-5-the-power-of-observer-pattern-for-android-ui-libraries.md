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
        inputMethodManager = (InputMethodManager) getActivity().getSystemService(Context.INPUT_METHOD_SERVICE);
        inputMethodManager.toggleSoftInput(InputMethodManager.SHOW_FORCED, InputMethodManager.HIDE_IMPLICIT_ONLY);
    }

    private void initView() {
        mPostContent.setOnEditTextImeBackListener(new EditTextImeBackListener() {
            @Override
            public void onImeBack(CustomEditText ctrl, String text) {
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
