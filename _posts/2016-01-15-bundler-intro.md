---
layout: post
title: Bundler
cover: false
date:   2016-01-15 10:18:00
tags: bundler
subclass: 'post tag-bundler'
categories: 'madki'
navigation: True
logo: 'assets/images/ghost.png'
disqus: 'xyz.madki.bundler-intro'
---

### The problem
Bundles and intents in android are used every where. They're android's way of serializing data,
persisting state and decoupling elements completely so that one component (activity/service/..)
does not need to depend on an interface or a method from another component to start it. The problem
with intents and bundles is two-fold.
1. There is too much broiler plate
2. Type safety is lost in the process of serialization

Consider a simple example app where in `PostActivity` we ask the user to provide a `title`,
`content` of a post and post it. This information along with the current time is passed on
to the `DisplayActivity` which displays the `title`, `content` and `time`. So to perform this
simple task the activities will look something like this:

First let's see the `DisplayActivity`

```java
public DisplayActivity extends Activity {
    // intent keys
    public static final String KEY_TITLE   = "activity_display_title"
    public static final String KEY_CONTENT = "activity_display_content"
    public static final String KEY_TIME    = "activity_display_time"

    String title;
    String content;
    String time;

    TextView titleTv;
    TextView contentTv;
    TextView timeTv;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_display);

        // init the views (or use ButterKnife)

        Intent intent = getIntent();
        if(intent != null) {
            title = intent.getStringExtra(KEY_TITLE, null);
            content = intent.getStringExtra(KEY_CONTENT, null);
            time = intent.getStringExtra(KEY_TIME, null);

            titleTv.setText(title);
            contentTv.setText(content);
            timeTv.setText(time);
        }
    }

}
```

Now the `PostActivity` looks like this

```java
public PostActivity extends Activity implements View.OnClickListener {
    EditText title;
    EditText content;
    Button post;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_post);

        // init the views (or use ButterKnife)

        post.setOnClickListener(this);
      }

    @Override
    public void onClick(View v) {
        Intent intent = new Intent(this, DisplayActivity.class)
                                .putExtra(DisplayActivity.KEY_TITLE,
                                          title.getText().toString())
                                .putExtra(DisplayActivity.KEY_CONTENT,
                                          content.getText().toString())
                                .putExtra(DisplayActivity.KEY_TIME,
                                          getTimeAsString());
        startActivity(intent);
    }

    private String getTimeAsString() {
        // obtain date time from calendar, format to string and return time
        return timeStr;
    }
}
```

For achieving this small result we had to define `KEYS` for the intent, construct that intent
in `PostActivity` and parse the intent in `DisplayActivity`. And after all this if in future
we decide that `DisplayActivity` would accept `time` as `long` instead of `String` then if
all the activities opening `DisplayActivity` would have to be altered and the compiler would
not be of any help if there are multiple activities accessing `DisplayActivity`. Since
`intent.putExtra(key, value)` accepts both `String` and `long` no compile time error would be
 shown. Only a runtime exception (which is also caught and printed as a warning in the logs) is
 thrown. How do we solve this then?

### The solution
 So a slightly better way and type safe way to do the above would be to define a static `start()`
 method in `DisplayActivity` and use only the static method to start activity:

 ```java
 public class DisplayActivity extends Activity {
    // intent keys can now be private
    private static final String KEY_TITLE   = "activity_display_title"
    private static final String KEY_CONTENT = "activity_display_content"
    private static final String KEY_TIME    = "activity_display_time"

    // same as the previous version

    public static void start(String title, String content, String time, Context context) {
        Intent intent = new Intent(context, DisplayActivity.class)
                                .putExtra(DisplayActivity.KEY_TITLE,
                                          title)
                                .putExtra(DisplayActivity.KEY_CONTENT,
                                          content)
                                .putExtra(DisplayActivity.KEY_TIME,
                                          time);
        context.startActivity(intent);
    }
 }
 ```

 Now `PostActivity` can simply use this method without bothering with the intent keys

 ```java
 public class PostActivity extends Activity implement View.OnClickListener {

    // same code as above version

    public void onClick(View v) {
        DisplayActivity.start(title.getText().toString,
                              content.getText().toString(),
                              getTimesAsString(),
                              this);
    }
 }
 ```

 By this calling code becomes much cleaner and also if in future `time` in `DisplayActivity` is changed
 to `long` then it would show an error at all the places `String` is being used in the `start()` method
 in compile time. This is a very good solution to the problem but increases the trivial code that needs
 to be written.

### [Bundler](https://github.com/workarounds/bundler) to reduce the broiler plate

[Bundler](https://github.com/workarounds/bundler) is an annotation processor that aims to generate
this trivial code for you. Using bundler the `DisplayActivity` will change to:

```java
@RequiresBundler
public class DisplayActivity extends Activity {
    @Arg
    String title;
    @Arg
    String content;
    @Arg
    String time;

    TextView titleTv;
    TextView contentTv;
    TextView timeTv;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_display);

        // init the views (or use ButterKnife)

        Bundler.inject(this);
    }

}
```

And now `DisplayActivity` can be started from `PostActivity` as follows:

```java
public class PostActivity extends Activity implement View.OnClickListener {

    // same code as the initial version

    public void onClick(View v) {
        Bundler.diplayActivity(title.getText().toString(),
                               content.getText().toString(),
                               getTimeAsString())
                               .start();
    }
}
```

`Bundler` is the generated class. It has static methods to start and inject activities (also fragments).
There are also some helper classes generated which provide additional advanced functionality (more about
this in a later post). It also has a `@State` annotation which can be used to save and restore instance
state (like IcePick library).

The objective of the library is to help make type safe ways to start and create activities, services and
fragments a breeze (as easy as defining annotated fields). Reviews of the API, comments and suggestions on
how to make it better, help with docs, testing and any sort of PRs in general are welcome. We're currently
using this library in this app:
[Define](https://play.google.com/apps/publish/?dev_acc=04786613567846403539#MarketListingPlace:p=in.workarounds.define)
and this library [Portal](https://github.com/workarounds/portal).

Please create issues or comment on this reddit thread for suggestions:
[reddit post]()