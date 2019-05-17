# Example of converting Objective-C dynamics to Swift

Some time ago I had to convert an Objective-C category I use to load UIView's from XIB's.

Here is the category:

``` objc
@implementation UIView (NIB)

+ (UIView *)loadInstanceFromNib
{ 
    NSArray* topLevelObjects = [[NSBundle mainBundle] loadNibNamed: NSStringFromClass([self class]) owner: nil options: nil];
    for (id object in topLevelObjects)
    { 
        if ([object isKindOfClass:[self class]])
        { 
            return object;
            break; 
        } 
    }
    return nil; 
}
```

This category lets you load a custom view from a XIB file with the same name as the custom view's class. It does this by loading the XIB, looping through all the high-level objects in the XIB and return the first object with the same class as  the `loadInstanceFromNib` class method is called from.

``` objc
CustomView *view = [CustomView loadInstanceFromNib];
```

How would we convert this category to Swift? Easy.

The category will become an extension on UIView in Swift. 

With loading the XIB we encounter the first problem: `NSStringFromClass(self)` in Swift will return us the class name prefixed by the module. In most cases this is the application name (e.g. `MyApplication.CustomView`)
The XIB is called CustomView.xib. We need to strip the module name from the string returned by `NSStringFromClass()`:

``` swift
var className = NSStringFromClass(self)
className = split(className, { $0 == "."})[1]	

```

Looping through the top level objects is the same in Swift. 

Swift has special syntax to test if an object is of a certain type. 

The _is_ operator lets you test if an object is of a certain class. `"Hello" is String` renders as `true`

The _as?_ operator used in an _if let_ construct will type cast the object to a type if the type matches.

``` swift
// object is of type Any
if let text = object as? String {
	// we only get here if object is a String. text is of type String
} 
```

We can use the _as?_ operator to test if the top level object is of our desired type.

``` swift
 for object in topLevelObjects {
 	if let view = object as? self {
       	return view
 	}
 }
```

Unfortunately, we cannot use _self_ as the return type. Swift wants to known the type at compile time. Objective-C didn't care about that. In Objective-C the `loadInstanceFromNib` method returns an `UIView *` in Swift we want to return the right type. 

We want this extension to be useable for any `UIView` subclass. We can use Swift [Generics](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Generics.html#//apple_ref/doc/uid/TP40014097-CH26-ID179) to accomplish that:


``` swift
extension UIView {
    class func loadInstanceFromNib<T: UIView>() -> T? {
        var className = T.description()
        className = split(className, { $0 == "."})[1]
        let nib = UINib(nibName:className, bundle:nil)
        let topLevelObjects = nib.instantiateWithOwner(self, options:nil)
        
        for object in topLevelObjects {
            if let view = object as? T {
                return view
            }
        }
        return nil
    }
}
```

Instead of looping through the top level objects with a for-loop, a more functional approach could be taken. Using a filter on the top level objects array will give us only the objects that match our criteria. This time we can use the _is_ operator.

``` swift
extension UIView {
    static func loadInstanceFromNib<T: UIView>() -> T? {
        var className = T.description()
        className = split(className, { $0 == "."})[1]
        let nib = UINib(nibName:className, bundle:nil)
        let topLevelObjects = nib.instantiateWithOwner(self, options:nil)
        
        return topLevelObjects.filter{ $0 is T }.first as? T
    }
}
```

The extension can be used in the following way:

``` swift
if let view:CustomView = UIView.loadInstanceFromNib() {
	// Do something with the view
}
```
The type annotation is needed to let the compiler determine the type of T. 