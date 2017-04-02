---
layout: post
title: Coordinator Layout and Floating Action Button Part 2 - Custom Behavior
published: true
---

### Introduction

In the <a href="http://programminglife.io/coordinatorlayout-and-floatingactionbutton/" target="_blank"> previous blog post </a> we covered the basic and most used components
of the Design Support library that Google released for Android.
In this blog post I will introduce a very important, yet not talked much about, feature that will make your apps look more snappy.
The feature that I will detail in this post is the <b> custom behavior </b>, and I have used the example from the <a href="http://programminglife.io/coordinatorlayout-and-floatingactionbutton/" target="_blank"> previous post </a>
by implementing a full functioning shopping cart.
The Design Support library also comes with pre defined behaviors like for example <b> appbar scrolling view behavior</b>. We will not cover
these in this post as they are already covered by Google in 
<a href="https://developer.android.com/intl/zh-cn/reference/android/support/design/widget/AppBarLayout.html" target="_blank"> their documentation </a>.
Bellow I will describe how we can build custom behavior for our FAB(Floating Action Button). What I wanted to achieve is that our FAB zooms every time an item is added to the cart.

### The code

You can find the full source code on <a href="https://github.com/andreivisan/AndroidPayDemo" target="_blank"> GitHub </a> as usual.
I have also added an action to our FAB so that when you click on it will get you to the check out page. I will not describe this here
as this is pretty simple to code and you can find it in the source files.

First let's create the custom behavior class that will implement the logic for our animation on the FAB.

```java
public class ZoomingFabBehavior extends CoordinatorLayout.Behavior<FloatingActionButton> {

    private static final String tag = ZoomingFabBehavior.class.getSimpleName();

    public ZoomingFabBehavior() { }

    public ZoomingFabBehavior(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public boolean layoutDependsOn(CoordinatorLayout parent, 
                                   FloatingActionButton child, 
                                   View dependency) {
        return dependency instanceof TextView;
    }

    @Override
    public boolean onInterceptTouchEvent(CoordinatorLayout parent, 
                                         final FloatingActionButton child, 
                                         MotionEvent ev) {
        child.animate()
                .scaleX(1.3f)
                .scaleY(1.3f)
                .setInterpolator(new LinearOutSlowInInterpolator())
                .withEndAction(new Runnable() {
                    @Override
                    public void run() {
                        child.animate().scaleX(1).scaleY(1).setDuration(500);
                    }
                })
                .setDuration(300);
        return false;
    }
}
```

### Code explanation

The <b>layoutDependsOn</b> method goes through all the elements of the layout in which the FAB is placed and when it reaches an element
of type TextView then it will return true. You can also make your FAB depend on any element in the layout it is placed into
as long as the parent layout is of type CoordinatorLayout.

The second method, <b>onInterceptTouchEvent</b>, intercepts a touch event on the element the FAB depends on (or the parent - which is a bit
sloppy in our situation because every time we scroll the list the FAB will zoom - but it doesn't interfere with the purpose of this demo).
The moment the event is caught we call animate on the child, in our case the FAB, and we scale it to be 1.3 times the normal size.
I also implemented <b>withEndAction</b> method so that when the animation is over the FAB is scaled back to its initial size.

Now let's apply this behavior that we have just implemented to the FAB.

```xml
<android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="end|bottom"
        android:layout_margin="16dp"
        app:layout_anchor="@id/itemsCount"
        android:src="@mipmap/shopping_cart"
        app:fabSize="normal"
        android:elevation="3dp"
        android:clickable="true"
        android:onClick="proceedToCheckout"
        app:backgroundTint="@color/adyen_green"
        app:layout_behavior=".behavior.ZoomingFabBehavior"/>
```

As you can see in the following line <b> app:layout_behavior=".behavior.ZoomingFabBehavior" </b> we have applied this custom
behavior to our cart button by adding it's name to the <b> layout_behavior </b> attribute.

Now you can run the code and check that when you add an item to the cart (by clicking on the item you wish to add). All the code for
adding items to the cart as well as the code above is on <a href="https://github.com/andreivisan/AndroidPayDemo" target="_blank">here</a>.

More common methods used to trigger custom behavior are <b> onDependentViewChanged </b> and <b> onNestedScroll </b>. One interesting fact
about the scroll methods is that you can calculate how far you scrolled in the screen and act based on that (hide the top action bar for example).

Please leave your comments bellow and let me know if this was helpful or if you need more information about the Design Support library.
