---
layout: post
title:  "Rate my app dialog box for Android application"
date:   2015-10-08 21:52:39
categories: Android tutorials
---

<b>Why do you need this dialog box ? </b>

You will need this popup to remind users of rating your application. The more users are rating your app the more popular your application become. The best possible result is to be in top 25 of your category.

<b>Creating the dialog box</b>

* First thing is to create a layout file in <i>/res/layout</i> and name it <i>rate_app_dialog.xml</i>

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical" >
    <RatingBar
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerHorizontal="true"
        android:stepSize = "1.0"
        android:numStars="5"
        android:id="@+id/ratingBar"
        android:layout_marginTop="16dp" />
</RelativeLayout>
{% endhighlight %}

Notice that this layout contains RatingBar element which will be used in the popup dialog.

* Add the necessary strings for popup in string.xml file

To be continued shortly technical problems...
