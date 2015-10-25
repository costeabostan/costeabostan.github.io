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
{% highlight xml %}
<string name="rate_dialog_title">Do you like application?</string>
<string name="rate_dialog_ok">Rate Us</string>
<string name="rate_dialog_no">No, Thanks</string>
<string name="rate_dialog_neutral">Maybe Later</string>
<string name="rate_dialog_message">"Please rate this app by selecting the stars. Thank you!"</string>
{% endhighlight %}

* Create a class named <i> RateAppDialogFragment </i>
First add all the necessary static variables
{% highlight xml %}
//fragment TAG
public static final String TAG = "RateAppDialogFragment";
public static final String RATE_APP = "rate_app";
public static final String DO_NOT_SHOW_AGAIN_TAG = "do_not_show_again";

public static final String LAUNCH_CODE_TAG = "launch_count";
public static final String DATE_FIRST_LAUNCH_TAG = "date_first_launch";
public static final int MIN_RATE_STAR = 4;

//Add 0 to start on the first day of installation
public static final int DAYS_UNTIL_PROMPT = 5;
public static final int LUNCH_UNTIL_PROMPT = 5;
{% endhighlight %}

Then the most important part is the static method used to save in the system preferences lunch count and time passed since the application installation, the code is simple to understand and there is no need for explanation:

{% highlight java %}
final static boolean showRateUs(Application application) {
    SharedPreferences sharedPreferences = application.getSharedPreferences(RATE_APP, 0);
    if(sharedPreferences==null || sharedPreferences.getBoolean(DO_NOT_SHOW_AGAIN_TAG, false)){
        return false;
    }
    SharedPreferences.Editor editor = sharedPreferences.edit();
    long launchCount = sharedPreferences.getLong(LAUNCH_CODE_TAG, 0) + 1;
    editor.putLong(LAUNCH_CODE_TAG, launchCount);
    Long dateFirstLaunch = sharedPreferences.getLong(DATE_FIRST_LAUNCH_TAG, 0);
    if(dateFirstLaunch == 0) {
        dateFirstLaunch = System.currentTimeMillis();
        editor.putLong(DATE_FIRST_LAUNCH_TAG, dateFirstLaunch);
    }
    editor.apply();
    if(launchCount >= LUNCH_UNTIL_PROMPT) {
        if(System.currentTimeMillis() >= dateFirstLaunch + (DAYS_UNTIL_PROMPT * 24 * 60 * 60 * 1000)){
            return true;
        }
    }
    return false;
}
{% endhighlight %}

Of course there is also need to add the layout <i>rate_app_dialog.xml</i> that was just created
This code sample does this thing also it contains the logic in case users whats to give bad reviews:

{% highlight java %}
@NonNull
@Override
public Dialog onCreateDialog(Bundle savedInstanceState) {
    final SharedPreferences.Editor editor = getActivity()
            .getSharedPreferences(RATE_APP, 0).edit();
    LayoutInflater inflater = getActivity().getLayoutInflater();
    final View view = (inflater.inflate(R.layout.rate_app_dialog, null));
    final RatingBar ratingBar = (RatingBar)view.findViewById(R.id.ratingBar);
    AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
    String title = "Application title";
    String message = getString(R.string.rate_dialog_message);
    builder.setView(view);
    builder.setMessage(message)
            .setTitle(getString(R.string.rate_dialog_title, title))
            .setIcon(getActivity().getApplicationInfo().icon)
            .setCancelable(false)
            .setPositiveButton(getString(R.string.rate_dialog_ok), new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) {
                    editor.putBoolean(DO_NOT_SHOW_AGAIN_TAG, true);
                    editor.apply();
                    if (ratingBar.getRating() >= MIN_RATE_STAR) {
                        try {
                            startActivity(new Intent(Intent.ACTION_VIEW,
                                    Uri.parse("market://details?id=" + getActivity().getPackageName())));
                        } catch (ActivityNotFoundException e) {
                            //No market found
                            startActivity(new Intent(Intent.ACTION_VIEW,
                                    Uri.parse("http://play.google.com/store/apps/details?id=" + getActivity().getPackageName())));
                        }
                    } else {
                        sendEmail();
                    }
                    dialog.dismiss();
                }
            })
            .setNeutralButton(getString(R.string.rate_dialog_neutral), new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) {
                    dialog.dismiss();
                }
            })
            .setNegativeButton(getString(R.string.rate_dialog_no), new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) {
                    if (editor != null) {
                        editor.putBoolean(DO_NOT_SHOW_AGAIN_TAG, true);
                        editor.apply();
                    }
                    dialog.dismiss();
                }
            });
    return builder.create();
}
{% endhighlight %}

And if the review is bad (less than three start) force the user to send an email to support team.

{% highlight java %}
public void sendEmail(){
    final Intent i = new Intent(android.content.Intent.ACTION_SEND);
    i.setType("message/rfc822");
    i.putExtra(android.content.Intent.EXTRA_EMAIL, new String[]{"test@gmail.com"});
    try {
        startActivity(Intent.createChooser(i, "Choose email client"));
    } catch (android.content.ActivityNotFoundException ex) {
        Toast.makeText(getActivity(), "No email client was found", Toast.LENGTH_SHORT).show();
    }
}
{% endhighlight %}

And the last thing is to not allow the user to rate this app till he chooses a star:

{% highlight java %}
@Override
public void onStart()
{
    super.onStart();
    final AlertDialog alertDialog = (AlertDialog)getDialog();
    if(alertDialog != null)
    {
        Button positiveButton = alertDialog.getButton(Dialog.BUTTON_POSITIVE);
        positiveButton.setEnabled(false);
        final RatingBar ratingBar = (RatingBar)alertDialog.findViewById(R.id.ratingBar);
        ratingBar.setOnRatingBarChangeListener(new RatingBar.OnRatingBarChangeListener() {
            @Override
            public void onRatingChanged(RatingBar ratingBar, float rating, boolean b) {
                if (rating >= 1) {
                    alertDialog.getButton(AlertDialog.BUTTON_POSITIVE).setEnabled(true);
                } else {
                    alertDialog.getButton(AlertDialog.BUTTON_POSITIVE).setEnabled(false);
                }
            }
        });
    }
}
{% endhighlight %}

download the full example here:

<a href="https://github.com/costeabostan/androidRateAppDialog">github project</a>
