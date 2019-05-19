---
layout: post
title: "Constants in Swift"
date: 2015-02-03 18:10
comments: false
published: true
tags: swift
---

With Objective-C, a common way to define constants you use in your classes -- reuse identifiers, storyboard segue identifier, etc. is to do something like this:

``` objective_c
@import "ABCThingsTableViewController.h"

// Storyboard constants
NSString * const ABCThingTableViewCell = @"ThingTableViewCell";

// Layout constants
const CGFloat ABCHeaderViewTopMargin = 12.0f;

@implementation ABCThingsTableViewController
{
...
```

Because we use global constants we have to use a prefix to make them unique. 

If we translate the example above to Swift we get something like this:

``` swift
// Storyboard constants
static let thingTableViewCell = "ThingTableViewCell";

// Layout constants
static let headerViewTopMargin = 12.0;

class ABCThingsTableViewController {
	...
```

In Swift we don't need prefixes as Swift has namespaces; the global variables are defined within the scope of the module only -- by default the application module. 
Swift also supports something called _type properties_; properties that are defined within the type's definition and are scoped to the type, not the instance. There is a different syntax for _structs_ and _classes_:

``` swift 
struct SomeStruct {
	static let topBorderHeight:CGFloat = 44.0
	...

class SomeClass {
	class let topBorderHeight:CGFloat = 44.0
	...
```

Unfortunately, type properties for classes are not supported yet -- as of Februari 2015 it throws an `error: class variables not yet supported` error.  _The Swift Programming Language_ book does not mention this.

There is an easy workaround for this:

``` swift 
class SomeClass {
	struct SomeStruct {
		static var someTypeProperty
	}
	...
```
 
 We can even use this workaround to our advantage to make the constants structure more clear:
 
``` swift 
class ABCThingsTableViewController {
	struct Storyboard {
		static let myTableCellIdentifier = "MyTableViewIdentifier"
		static let detailSegueIdentifier = "DetailViewController"
	}
	
	struct Layout {
		static let topBorderHeight:CGFloat = 44.0
	}
	...
```
 
 You can use the constants in the following way:
 
``` swift 
func collectionView(collectionView: UICollectionView, cellForItemAtIndexPath indexPath: NSIndexPath) -> UICollectionViewCell {
	var cell = collectionView.dequeueReusableCellWithReuseIdentifier(Storyboard.myTableCellIdentifier, forIndexPath: indexPath) 
	...
```


``` swift 
// Access from another class
let offset = ABCThingsTableViewController.Layout.topBorderHeight
```

All this results in a nice self-documenting way to define constants without worrying about namespace issues.


## Edit

With the release of Swift 1.2 we do have static class variables. 

So this is now valid:

``` swift 
class SomeClass {
	static let topBorderHeight:CGFloat = 44.0
	...
```

This means we don't need the struct within a class construct to make this all work. I still like, and will use, the struct within a class to group the 
static variables as I think it will improve readability of your code.