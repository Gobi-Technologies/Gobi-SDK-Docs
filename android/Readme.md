# Gobi SDK


## Installing

> Check out https://github.com/Gobitech/Gobi-Android-SDK-Sample-App for a sample

### Repository

Add this in `app/build.gradle`:

```groovy
repositories {
    maven { url "http://gobi-public-maven.s3-website.eu-central-1.amazonaws.com/" }
}
```

### SDK

Add this in `app/build.gradle`:
```groovy
dependencies {
    implementation 'no.gobiapp.gobi.sdk:sdk:1.4.0'
}
```

## Get your access key

Get them from Gobitech AS on email, `contact@gobiapp.com`.

## Integrate in the app

### 1. Initialize `Gobi` in your Application subclass

```java
public class SampleApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        Gobi.with(this, "ZDY5OGI0YZQ3MJG0MWE3ZDA0YWEXZDLK"); // "Sample SDK app" customer
    }    
}
```

### 2. Store your story keys

```java
/**
 * Contains all stories we own
 *
 * @author Kristian 'krissrex' Rekstad
 */
public class Stories {
    private Stories() {}

    /** "Team Gobi" story */
    public static final String TEAM_GOBI = "OWEXMJUWM2FKMTE5M2U3NWIXZTIZZDK0NJQ2NJUYNZRKOGZHNZM1ZJFINWVHMJBK";
}
```

## Usage
### 1. Show a story

Use the method `Gobi.showStory(storyKey, supportFragmentManager)`.  
If you are not using `AppCompat`, use `Gobi.showStory(storyKey, activity)`.

```java
import android.widget.Button;
import android.support.v7.app.AppCompatActivity;
import no.gobiapp.gobi.sdk.Gobi;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button showStoryButton = findViewById(R.id.button_activityMain_showStory);

        showStoryButton.setOnClickListener(v -> showStory());
    }

    public void showStory() {
        Gobi.showStory(Stories.TEAM_GOBI, getSupportFragmentManager());
    }
}
```

### 2. Get info about a story

Use the method `gobi.getStoryDataForId(storyKey)`. It returns a [RxJava 1](https://github.com/ReactiveX/RxJava/tree/1.x) [Single](https://static.javadoc.io/io.reactivex/rxjava/1.2.1/rx/Single.html),
which emits 1 item or errs.  
A `Single` is quite close to a javascript [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).

```java
import android.widget.Button;
import android.support.v7.app.AppCompatActivity;
import no.gobiapp.gobi.sdk.Gobi;
import no.gobiapp.gobi.sdk.data.StoryData;
import rx.SingleSubscriber;
import rx.Subscription;
import rx.subscriptions.CompositeSubscription;

public class MainActivity extends AppCompatActivity {
    private final CompositeSubscription subscriptions = new CompositeSubscription();

    TextView storyText;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        storyText = findViewById(R.id.textView_activityMain_storyText);

        loadStoryInfo();
    }

    private void loadStoryInfo() {
        // The SDK uses RxJava 1 to handle asynchronous actions. The call to `getStoryDataForId`
        // can be canceled by unsubscribing from the subscription, as seen in `onStop`.
        final Subscription storyDataSubscription = Gobi.getStoryDataForId(Stories.TEAM_GOBI)
                .subscribe(new SingleSubscriber<StoryData>() {
                    @Override
                    public void onSuccess(StoryData storyData) {
                        storyText.setText("View " + storyData.getStoryName());
                    }

                    @Override
                    public void onError(Throwable error) {
                        Log.e(TAG, "Failed to get story data", error);
                        storyText.setText("Error: Failed to get story info");
                    }
                });
        subscriptions.add(storyDataSubscription);
    }


    @Override
    protected void onStop() {
        super.onStop();
        subscriptions.clear();
    }
}
```


## Configuration

### Logging

If you already use [Timber](https://github.com/JakeWharton/timber), you can skip this step if you have installed a `DebugTree`.

Implement the `Logger` interface, and then add your implementation as a `logger`:

```java
Gobi.logger(new GobiAndroidLogger()); // Add a logger bridge
```

Heres an example of using Android's default `Log` class:

```java
import android.util.Log;

/**
 * Makes Gobi log to android {@link Log}.
 */
@SuppressLint("LogNotTimber")
private static class GobiAndroidLogger implements Logger {
    @Override
    public void debug(@Nullable String tag, @Nullable Throwable throwable, @NonNull String message) {
        Log.d(tag, message, throwable);
    }

    @Override
    public void info(@Nullable String tag, @Nullable Throwable throwable, @NonNull String message) {
        Log.i(tag, message, throwable);
    }

    @Override
    public void warn(@Nullable String tag, @Nullable Throwable throwable, @NonNull String message) {
        Log.w(tag, message, throwable);
    }

    @Override
    public void error(@Nullable String tag, @Nullable Throwable throwable, @NonNull String message) {
        Log.e(tag, message, throwable);
    }
}
```
