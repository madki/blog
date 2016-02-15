---
layout: post
title: Defaults in Bundler 2.0
cover: false
date:   2016-02-05 10:18:00
tags: bundler
subclass: 'post tag-bundler'
categories: 'madki'
navigation: True
logo: 'assets/images/ghost.png'
---

The previous version of `Bundler` used fields, so they could be initialized to some value, if the intent/bundle recieved didn't contain the key corresponding to the field it was left as is. So in a way it provided some defaults. For example:

```java
@RequireBundler(requireAll=false)
public class DemoActivity extends Activity {
	@Arg int id = 1;
	@Arg String name = "Jake";

	@Override
	protected void onCreate(Bundle savedState) {
		Bundler.inject(this);
	}
}
```

So if the intent didn't have an extra corresponding to `id` value of the field would be left as `1` and similarly `name` would default to `Jake`. Though this was useful there's an unwanted side-effect with this stategy when followed in a `Service` or activities that use `onNewIntent`. Cosider the following service:

```java
@RequireBundler(requireAll=false)
public class DemoService extends Service {
	@Arg int id;
	@Arg String name;

	@Override
	public int onStartCommand(Intent intent, int flags, int startId) {
		Bundler.inject(this);
		return super.onStartCommand(intent, flags, startId);
	} 
}
```

The problem with this is if first intent sets the value of `id` to `1` and `name` to `"Jake"` and there's a second intent that was intended to set `id` to `2` then `name` would still be `"Jake"` though nothing was sent. And there'd be no way to know that this happened. 

Now since there are no more fields and we have methods instead, we have more control. The methods can take a parameter of the same type:

```java
public class DemoService extends Service {
	
	@Override
	public int onStartCommand(Intent intent, int flags, int startId) {
		Extras extras = DemoService_Extras.from(intent);
		// use id and name as follows:
		extras.id(-1);
		extras.name(null);
		return super.onStartCommand(intent, flags, startId);
	} 

	@Args
	static abstract Extras {
		protected abstract int id(int defaultVal);
		protected abstract String name(String defaultVal);
	}
}
``` 

