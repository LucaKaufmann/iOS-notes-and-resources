# iOS Frameworks

## Common issues

### Framework paths
Fix for the following error: `dyld: Library not loaded: @rpath/FRAMEWORK NAME.framework/FRAMEWORK NAME
  Referenced from: /var/containers/Bundle/Application/78046CEF-403C-4103-A188-29B52FA061E4/APP NAME.app/APP NAME
  Reason: image not found
`
First, check *Build Settings > Linking > Runpath Search Paths* and make sure `@executable_path/Frameworks` is included.

Then, in **Build Phases** add a new **Copy Files** phase. Select destination **Frameworks** and add your .framework file to the list.

### Dependencies
If you have any dependencies, it's possible to add them using Cocoapods. Keep in mind that those will not get packaged into the framework, any app implementing the framework will have to add the same Cocoapods as the framework.

### Swift - ObjC issues

#### Using Objective-C in Swift
Bridging Headers to expose Objective-C classes to Swift are not supported. Headers for classes used by Swift can be put in the Framework umbrella header file. This will also expose them publicly to anyone using the Framework. One workaround for that is using a [modular framework](http://nsomar.com/modular-framework-creating-and-using-them/). The ObjC headers that need to be exposed to Swift are added to a private module, which is then imported into Swift. See an example here [https://github.com/danieleggert/mixed-swift-objc-framework](https://github.com/danieleggert/mixed-swift-objc-framework).

**Note:** I received a redefinition error when adding the framework to an app, so to make this work I had to rename the x.private.modulemap ([See here](https://github.com/Randoramma/Frameworking/blob/master/mixed-swift-objc-framework/Foo/Foo/FooPrivate/module.modulemap)).

#### Using Swift in Objective-C
The problem here is that only public Swift classes will be included in the generated PROJECTNAME-Swift.h Header file. But what if there are classes you don't want to expose to anyone using your Framework? The solution is quite simple, create an Objective-C header including all the Swift classes you'd like to use in Objective-C. Import this new header into every Objective-C class that needs access to those Swift classes. You can now leave the Swift classes declared as internal so they don't get included in the public PROJECTNAME-Swift.h header file.

Example:

```Swift
@objc(SWIFTYetAnotherSwiftObject)
internal class YetAnotherSwiftObject: NSObject {
    internal class func doSomething() {}
}
```

```Swift
// PrivateSwiftHeaders.h

SWIFT_CLASS("SWIFTYetAnotherSwiftObject")
@interface YetAnotherSwiftObject : NSObject

+ (void)doSomething;

@end
```

Source: [https://stackoverflow.com..](https://stackoverflow.com/questions/32700078/how-to-access-an-internal-swift-class-in-objective-c-within-the-same-framework)


## Universal Frameworks
To make development easier for developers implementing your framework, you should build it for both the device and simulator architecture. One way to create an universal framework is by creating a new "aggregate" target in XCode, adding a run script phase with the following script: https://gist.github.com/syshen/c24d127e1adc2783e0e7

Then go to *Manage Schemes* and check *shared* for your Framework and the new Aggregate target.

Go here for the full guide: https://medium.com/@syshen/create-an-ios-universal-framework-148eb130a46c



## Links

* http://www.jensmeder.de/swift/xcode/c/2016/07/17/swift-c-framework.html
* http://nsomar.com/project-and-private-headers-in-a-swift-and-objective-c-framework/
* http://nsomar.com/modular-framework-creating-and-using-them/
* https://medium.com/swift-and-ios-writing/using-a-c-library-inside-a-swift-framework-d041d7b701d9
* https://solidgeargroup.com/bridging-swift-objective-c
* https://medium.com/@syshen/create-an-ios-universal-framework-148eb130a46c
