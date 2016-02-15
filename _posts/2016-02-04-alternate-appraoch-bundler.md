---
layout: post
title: Alternate approach for Bundler
cover: false
date:   2016-02-04 10:18:00
tags: bundler
subclass: 'post tag-bundler'
categories: 'madki'
navigation: True
logo: 'assets/images/ghost.png'
disqus: 'xyz.madki.alternate-approach-bundler'
---

## Problems with Bundler

* global fields with package level visibility
* use of generated classes all over the place
* may be generates too much code and should make users write more (?)
* `@State` is an entirely different problem that `Bundler` need not address
* An activity using `Bundler` to get type-safety is not an implementation detail and ideally it should be.
* Field defaults and resetting (what should be default value, should the field value be reset when a second value is injected)

Right now calling classes make use of the generated `Bundler` class for starting activities. But ideally Bundler should be an implementation detail for the calling classes. A better approach would be to not generate the `bundlerActivity()` methods in `Bundler`.

So a better version of `Bundler` though requiring more code would have the following features:

* No global fields
* calling classes shouldn't need to know the existence of `Bundler`
* minimize the use of generated classes

An approach similar to the super-awesome `AutoValue` library can be taken. Instead of getting the intent extras from fields they can be defined in an interface (or an abstract class ?)

```java
public class DemoActivity extends Activity {
	
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		Extras extras = DemoActivityExtras.get(this);
		// use id(-1) and name() methods of Extras
	}

	@Args
	interface Extras {
		@Key("some_key")
		int id(int defaultVal);
		@Required(false) @Nullable
		String name();

		// extra helper methods can be added to know if a key was sent or not
		@Helper
		boolean hasName();
	}

	public static DemoActivityExtras.Builder withId(int id) {
		return DemoActivityExtras.build(id);
	}
}
```

A calling class (say `AnotherActivty`) can start the above `DemoActivity` as:

```java
	// code in AnotherActivity.class
	DemoActivity.withId(1).name("xyz").start(context);
```

The only problem with this is `DemoActivityExtras.Builder` which is a generated class is being exposed. Ideal case would be to go the full `AutoValue` way and require a Builder interface inside Extras. But how much code is too much and where lies the line. This version of `Bundler` is more sound that the current one but it definitely increased the code, and a little more code `Bundler` would become an implementation detail completely.
