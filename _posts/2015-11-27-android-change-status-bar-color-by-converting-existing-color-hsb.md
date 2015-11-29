---
layout: post
title: Android change status bar color by converting existing color's HSB
published: true
---

### Introduction

This will be a very short post but I decided to write about this topic since I had to deal with it myself and couldn't find many resources that were helpful. I also find it an interesting problem since with the new Material Design you might have a custom toolbar and you want the status bar to be a darker shade of the same color.

### The problem

Let's assume you build an Android application that looks like the image bellow.

![Android Material Design toolbar](/public/images/ThemeColors.png)

As you can see in the image above the way to add a custom color to your toolbar and status bar is by setting the `colorPrimary` and `colorPrimaryDark` in your custom theme.

``` xml
<resources>
  <!-- inherit from the material theme -->
  <style name="AppTheme" parent="android:Theme.Material">
    <!-- Main theme colors -->
    <!--   your app branding color for the app bar -->
    <item name="android:colorPrimary">@color/primary</item>
    <!--   darker variant for the status bar and contextual app bars -->
    <item name="android:colorPrimaryDark">@color/primary_dark</item>
    <!--   theme UI controls like checkboxes and text fields -->
    <item name="android:colorAccent">@color/accent</item>
  </style>
</resources>
```

But let's say we want to build an application that can change the toolbar color based on certain parameters and we want to set the status bar color too, based on the toolbar color and that can't be achieved by hardcoding the colors.

### The solution

The solution I came up with was to create, for example, a `ColorUtil.java` class in which I coded the following method:

``` java
public static String changeColorHSB(String color) {
    float[] hsv = new float[3];
    int brandColor = Color.parseColor(color);
    Color.colorToHSV(brandColor, hsv);
    hsv[1] = hsv[1] + 0.1f;
    hsv[2] = hsv[2] - 0.1f;
    int argbColor = Color.HSVToColor(hsv);
    String hexColor = String.format("#%08X", argbColor);
    return hexColor;
}
```

#### Getting the string HEX of a color from colors.xml

The call to this method should look something like this: `changeColorHSB(getResources().getString(R.color.black))`. This should give you the string of the hex value of the color under the name `black` defined in your `colors.xml` file.

#### Converting HEX color to HSV

So what I did in the method above was to get the string hex code of a color I set in `colors.xml` and convert it to `int` so I can then convert it to an `HSV float array` which has size 3 and contains the `Hue`, the `Saturation`  and the `Value` for our color. After adding `0.1` to the `Saturation` and subtracting `0.1` from `Value` I converted the `HSV` color into `ARGB` color.

#### Converting ARGB color to HEX

In order to convert the `ARGB` color to `HEX` I used the following statement: `String.format("#%08X", argbColor);`.
And that's it, now we have a darker shade of the color we pass as parameter to our `changeColorHSB` method. 
Now in order to change the color of your status bar call this method in your activity: 

``` java
getWindow().
  setStatusBarColor(
    Color.parseColor(
      ColorUtil.changeColorHSB(parentActivity.getResources().getString(R.color.grey))));
``` 

or from your fragment: 

```java
getActivity().
  getWindow().
    setStatusBarColor(
      Color.parseColor(
        ColorUtil.changeColorHSB(parentActivity.getResources().getString(R.color.grey))));
```

### Conclusion

Even though this might not be a common use case, I hope the solution above will help somebody.
Please leave a comment bellow if it helped you or if you need any further help or just want to leave some suggestions.
