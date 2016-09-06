# Reusable

![Reusable](Example/ReusableDemo/Assets.xcassets/AppIcon.appiconset/AppIcon-167.png)

A Swift mixin to use `UITableViewCells`, `UICollectionViewCells` and `UIViewControllers` in a **type-safe way**, without the need to manipulate their `String`-typed `reuseIdentifiers`. This library also supports arbitrary `UIView` to be loaded via a XIB using a simple call to `loadFromNib()`

[![Platform](http://cocoapod-badges.herokuapp.com/p/Reusable/badge.png)](http://cocoadocs.org/docsets/Reusable)
[![Version](http://cocoapod-badges.herokuapp.com/v/Reusable/badge.png)](http://cocoadocs.org/docsets/Reusable)

## Introduction

This library aims to make it super-easy to create and dequeue/instantiate Reusable views, including `UITableViewCell`, `UICollectionViewCell`, and custom `UIView`s, by **simply marking your classes as conforming to a procols, without having to add any code**.

Ths concept is called a [Mixins](http://alisoftware.github.io/swift/protocol/2015/11/08/mixins-over-inheritance/) — basically a protocol with default implementation for all its methods — and its application for cells is explained [here in my blog post](http://alisoftware.github.io/swift/generics/2016/01/06/generic-tableviewcells/).

* Type-safe cells
* Type-safe XIB-based reusable views
* Type-safe ViewControllers from Storyboards 



---



## Type-safe `UITableViewCell` / `UICollectionViewCell`

### Declare your cells to conform to `Reusable` or `NibReusable`

* Use the `Reusable` protocol if they don't depend on a NIB (this will use `registerClass(…)` to register the cell)
* Use the `NibReusable` protocol if they use a `XIB` file for their content (this will use `registerNib(…)` to register the cell)

💡 Everything that applies for `UITableView` and `UITableViewCell` here also applies to `UICollectionView` and `UICollectionViewCell`.

```swift
final class CustomCell: UITableViewCell, Reusable { /* And that's it! */ }
```

> ✍️ **Notes**
> 
> * For cells embedded in a Storyboard's tableView, either one of those two protocols will work (as you won't register the cell them manually anyway)
> * If you create a XIB-based cell, don't forget to set its _Reuse Identifier_ field in Interface Builder to the same string as the name of the cell class itself.


<details>
<summary>Example for a Code-based custom tableView cell</summary>

```swift
final class CodeBasedCustomCell: UITableViewCell, Reusable {
  // By default this cell will have a reuseIdentifier of "CodeBasedCustomCell"
  // unless you provide an alternative implementation of `var reuseIdentifier`
  
  // No need to add anything to conform to Reusable. you can just keep your normal cell code
  @IBOutlet private weak var label: UILabel!
  func fillWithText(text: String?) { label.text = text }
}
```
</details>

<details>
<summary>Example for a Nib-based custom tableView cell</summary>

```swift
final class NibBasedCustomCell: UITableViewCell, NibReusable {
  // Here we provide a nib for this cell class (which, if we don't override the protocol's
  // default implementation of `nib`, will use a XIB of the same name as the class)
  
  // No need to add anything to conform to Reusable. you can just keep your normal cell code
  @IBOutlet private weak var pictureView: UIImageView!
  func fillWithImage(image: UIImage?) { pictureView.image = image }
}
```
</details>

<details>
<summary>Example for a Code-based custom collectionView cell</summary>

```swift
// A UICollectionViewCell which doesn't need a XIB to register
// Either because it's all-code, or because it's registered via Storyboard
final class CodeBasedCollectionViewCell: UICollectionViewCell, Reusable {
  // The rest of the cell code goes here
}
```
</details>

<details>
<summary>Example for a Nib-based custom collectionView cell</summary>

```swift
// A UICollectionViewCell using a XIB to define it's UI
// And that will need to register using that XIB
final class NibBasedCollectionViewCell: UICollectionViewCell, NibReusable {
  // The rest of the cell code goes here
}
```
</details>

### Register your cells

Unless you've prototyped your cell in a Storyboard, you'll have to register the cell class or Nib by code.

To do this, instead of calling `registerClass(…)` or `registerNib(…)` using a String-based `reuseIdentifier`, just call:

```swift
tableView.registerReusableCell(theCellClass)
```

<details>
<summary>Example of tableView cell registration</summary>

```swift
class MyViewController: UIViewController {
  @IBOutlet private weak var tableView: UITableView!
  
  override func viewDidLoad() {
    super.viewDidLoad()
    tableView.registerReusableCell(CodeBasedCustomCell) // This will register using the class without using a UINib
    tableView.registerReusableCell(NibBasedCustomCell) // This will register using NibBasedCustomCell.xib
  }
}
```
</details>

### Dequeue your cells

To dequeue a cell (typically in your `cellForRowAtIndexPath` implementation), simply call:

```swift
tableView.dequeueReusableCell(indexPath: indexPath) as MyCustomCell
```

Swift will use type-inference to understand that you'll want a cell of type `MyCustomCell` and thus magially infer both the cell class to use and thus its `reuseIdentifier` to use, and which exact type to return.

* No need for you to manipulate `reuseIdentifiers` Strings manually anymore!
* No need to force-casting the returned `UITableViewCell` instance down to your `MyCustomCell` class either!

<details>
<summary>Example implementation of `cellForRowAtIndexPath` using `Reusable`</summary>

```swift
extension MyViewController: UITableViewDataSource {
  func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
    if indexPath.section == 0 {
      let cell = tableView.dequeueReusableCell(indexPath: indexPath) as CodeBasedCustomCell
      // Customize the cell here. You can call any type-specific methods here without the need for type-casting
      cell.fillWithText("Foo")
      return cell
    } else {
      let cell = tableView.dequeueReusableCell(indexPath: indexPath) as NibBasedCustomCell
      // Customize the cell here. no need to downcasting here either!
      cell.fillWithImage(UIImage(named:"Bar"))
      return cell
    }
  }
}
```
</details>

Now all you have is **a beautiful code and type-safe cells**, with compile-type checking, and no more String-based API!



---



## Type-safe XIB-based reusable views

`Reusable` also allows you to create reusable custom views designed in Interface Builder to reuse them in other XIBs or by code, like creating custom UI widgets used in multiple places in your app.

### Declare your views to conform to `NibLoadable` or `NibOwnerLoadable`

In your swift source declaring your custom view class:

* Use the `NibLoadable` protocol if the XIB you're using don't use its "File's Owner" and the reusable view you're designing is the root view of the XIB
* Use the `NibOwnerLoadable` protocol if you used a "File's Owner" of the XIB being of the class of your reusable view, and the root view(s) of the XIB is to be set as a subview providing its content.

```swift
// a XIB-based custom UIView, used as root of the XIB
final class NibBasedRootView: UIView, NibLoadable { /* and that's it! */ }

// a XIB-based custom UIView, used as the XIB's "File's Owner"
final class NibBasedFileOwnerView: UIView, NibOwnerLoadable { /* and that's it! */ }
```

_💡 You should use the second approach if you plan to use your custom view in another XIB or Storyboard, because it will allow you to drop a UIView in yourXIB/Storyboard, change its class to the class of your custom XIB-based view, and that custom view will then automagically load its own content from the associated XIB, without having to write additional code to load the content manually every time._

### Design your view in Interface Builder

For example if you named your class `MyCustomView` and made it `NibOwnerLoadable`:

* Set the _File's Owner_'s class to `MyCustomView`
* Design the content of the view via the root view of that XIB (which is a standard `UIView` with no custom class) and its subviews
* Connect any `@IBOutlets` and `@IBActions` between the _File's Owner_ (the `MyCustomView`) and its content

<details>
<summary>A view configured to be `NibOwnerLoadable`</summary>

![NibOwnerLoadable view in Interface Builder](NibOwnerLoadable.png)

```swift
final class MyCustomWidget: UIView, NibOwnerLoadable {
  @IBOutlet private var rectView: UIView!
  @IBOutlet private var textLabel: UILabel!

  @IBInspectable var rectColor: UIColor? {
    didSet {
      self.rectView.backgroundColor = self.rectColor
    }
  }
  @IBInspectable var text: String? {
    didSet {
      self.textLabel.text = self.text
    }
  }
…
}
```
</details>

Then that widget can be integrated in a Storyboard Scene (or any other XIB) by simply dropping a `UIView` on the Storyboard, and changing its class to `MyCustomWidget`.

<details>
<summary>Example of a `NibOwnerLoadable` custom view once integrated in another Storyboard</summary>

* In the capture below, all blue square views have a custom class of `MyCustomWidget` set in Interface Builder.
* When selecting one of this custom class, you have direct access to all `@IBOutlet` that this `MyCustomWidget` exposes, which allows you to connect them to other views of the Storyboard if needed
* When selecting one of this custom class, you also have access to all the `@IBInspectable` properties. For example, n the capture below, you can see the "Rect color" and "Text" inspectable properties on the right panel, that you can change right from the Storyboard integrating your custom widget.

![NibOwnerLoadable integrated in a Storyboard](NibOwnerLoadable-InStoryboard.png)
</details>

### Auto-loading the content of a `NibOwnerLoadable` view

If you used `NibOwnerLoadable` and made your custom view the File's Owner of your XIB, you should then override `init?(coder:)` so that it load it's associated XIB as subviews and add constraints automatically:

```swift
final class MyCustomWidget: UIView, NibOwnerLoadable {
  …
  required init?(coder aDecoder: NSCoder) {
    super.init(coder: aDecoder)
    MyCustomWidget.loadFromNib(owner: self)
  }
  override init(frame: CGRect) {
    super.init(frame: frame)
  }
}
```

Overriding `init?(coder:)` allows your `MyCustomWidget` custom view to load its content from the associated XIB `MyCustomWidget.xib` and add it as subviews of itself.

_💡 Note: overriding `init(frame:)`, even just to call `super.init(frame: frame)` might seems pointless, but seems necessary in some cases due to an issue with Swift and dynamic dispatch not being able to detect that this function is declared in the superclass. I've sometimes seen crashes when not implementing it, so better safe than sorry._

### Instantiating a `NibLoadable` view

If you used `NibLoadable` and made your custom view the root view of your XIB (not using the File's Owner at all), there are not designed to be used in other Storyboards or XIBs like `NibOwnerLoadable` as they won't be able to auto-load their content.

Instead, you will instantiate those `NibLoadable` views by code, which is as simple as calling `loadFromNib()` on your custom class:

```swift
let view1 = NibBasedRootView.loadFromNib() // Create one instance
let view2 = NibBasedRootView.loadFromNib() // Create another one
let view3 = NibBasedRootView.loadFromNib() // and another one
…
```



---



## Type-safe ViewControllers from Storyboards 

`Reusable` also allows you to mark your `UIViewController` classes as `StoryboardBased` or `StoryboardSceneBased` to easily instantiate them from their associated Storyboard in a type-safe way.

### Declare your `UIViewController` to conform to `StoryboardBased` or `StoryboardSceneBased`

In your swift source declaring your custom `UIViewController` class:

* Use the `StoryboardBased` protocol if the `*.storyboard` file has the same name as the ViewController's class, and its scene is the "initial scene" of the storyboard.
  * This is typically ideal if you use one Storyboard per ViewController, for example.
* Use the `StoryboardSceneBased` protocol if scene in your storyboard has the same `sceneIdentifier` as the name of the ViewController's class, but the `*.storyboard` file name doesn't necessary match the ViewController's class name.
  * This is typically ideal for secondary scenes in bigger storyboards
  * You'll then be required to implement the `storyboard` type property to indicate the storyboard it belongs to.

<details>
<summary>Example of a ViewController being the initial ViewController of its Storyboard</summary>

In this example, `CustomVC` is designed as the initial ViewController of a Storyboard named `CustomVC.storyboard`:

```swift
final class CustomVC: UIViewController: StoryboardBased { /* and that's it! */ }
```
</details>

<details>
<summary>Example of a ViewController being an arbitrary scene in a Storyboard</summary>

In this example, `SecondaryVC` is designed in a Storyboard name `CustomVC.storyboard` (so with a different name than the class itself) and is _not_ the initial ViewController, but instead has its **"Scene Identifier"** set to the value `"SecondaryVC"`

Conforming to `StoryboardSceneBased` will still require you to implement `static var storyboard: UIStoryboard { get }` to indicate the Storyboard where this scene is designed. You can typically implement that property using a `let` type constant:

```swift
final class SecondaryVC: UIViewController: StoryboardSceneBased {
  static let storyboard = UIStoryboard(name: "CustomVC", bundle: nil)
  /* and that's it! */
}
```
</details>

### Instantiate your UIViewControllers

Simply call `instantiate()` on your custom class. This will automatically know which storyboard to load it from, and which scene (initial or not) to use to instantiate it.

```swift
func presentSecondary() {
  let vc = SecondaryVC.instantitate() // Init from the "SecondaryVC" scene of CustomVC.storyboard
  self.presentViewController(vc, animated: true) {}
}
```



---



## Tip: make your subclasses `final`

I advise your to mark your custom `UITableViewCell`, `UICollectionViewCell`, `UIView` and `UIViewController` subclasses as being `final`. This is because:

* In most cases, the custom cells and VCs you plan to instantiate are not intended to be subclassed themselves.
* more importantly, it helps the compiler a lot and gives you big optimizations
* it can be required in some cases when conforming to `protocols` that have `Self` requirements, like the ones used by this pod (`Reusable`, `StoryboardBased`, …). 

In some cases you can avoid making your classes `final`, but in general it's a good practice, and in the case of this pod, usually your custom `UIViewController` or whatever won't be subclassed anyway:

* Either they are intended to be used and instantiated directly and never be subclassed, so `final` makes sense here
* In case your custom `UIViewController`, `UITableViewCell`, etc… is intended to be subclassed and be the parent class of many classes in your app, it makes more sense to add the protocol conformance (`StoryboardBased`, `Reusable`, …) to the child classes (and mark _them_ `final`) than adding the protocol on the parent, abstract class.



---



## Customize reuseIdentifier, nib, etc for non-conventional uses

The protocols in this pod, like `Reusable`, `NibLoadable`, `NibReusable`, `NibOwnerLoadable`, `StoryboardBased`… are what is usually called [Mixins](http://alisoftware.github.io/swift/protocol/2015/11/08/mixins-over-inheritance/), which basically is a Swift protocol with a default implementation provided for all of its methods.

The main benefit is that **you don't need to add any code**: just conform to `Reusable`, `NibOwnerLoadable` or any of those protocol and you're ready to go with no additional code to write.

But of course, those provided implementations are just **default implementations**. That means that if you need **you can still provide your own implementations** in case for some reason some of your cells don't follow the classic configuration of using the same name for both the class, the `reuseIdentifier` and the XIB file.

```swift
class VeryCustomNibBasedCell: UITableViewCell, NibReusable {
  // This cell use a non-standard configuration: its reuseIdentifier and XIB file
  // have a different name as the class itself. So we need to provide a custom implementation or `NibReusable`
  static var reuseIdentifier: String { return "VeryCustomReuseIdentifier" }
  static var nib: UINib { return UINib(nibName: "VeryCustomUI", bundle: nil) } // Use VeryCustomUI.xib
  
  // Then continue with the rest of your normal cell code 
}
```



---



## Example Project

This repository comes with an example project in the `Example/` folder. Feel free to try it.

It demonstrate how `Reusable` work:

* both for `UITableViewCell` and `UICollectionViewCell` subclasses,
* both for cells whose UI template is either only provided by plain code, or provided by a XIB, or prototyped directly in a Storyboard.
* both for cells `UICollectionView`'s `SupplementaryViews` (section Headers)



## License

This code is distributed under the MIT license. See the `LICENSE` file for more info.
