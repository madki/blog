---
layout: post
title: Interface vs abstract class for `@Args`
cover: false
date:   2016-02-05 10:18:00
tags: bundler
subclass: 'post tag-bundler'
categories: 'madki'
navigation: True
logo: 'assets/images/ghost.png'
---

Having decided defining the `@Arg`s are better off by not being fields. Having an inner `class/interface` for them seems like a good idea. For a DemoActivity with two fields `int id` and `String name` the `class` and `interface` will look as follows:

```java
	// inner class of DemoActivity
	@Args
	interface Extras {
		int id();
		String name();
	}
```

```java
	// inner class of DemoActivity
	@Args
	static abstract class Extras {
		protected abstract int id();
		protected abstract String name();
	} 
```

Point weighing in the direction of `interface` is the marginally lesser amount of code. But the advantages with using a `class` for this job are evident the `class` does not need to have `public` visibility, it could be package-default and the methods inside can be `protected` as only the surrounding class `DemoActivity` will be accessing it. The fields cannot be `private` because then the generated class won't be able to extend it. And additionally methods can be added to a class that act as derived fields for example:

```java
	// inner class of DemoActivity
	@Args
	static abstract class Extras {
		protected abstract String firstName();
		protected abstract String lastName();

		// not an Intent Extra
		protected String fullName() {
			return firstName() + " " + lastName();
		}
	} 
```

`extras.fullName()` can now be accessed although it's not data sent in intent. Any data modifications can be done in such a manner. Although I'm not sure how useful this is going to be.