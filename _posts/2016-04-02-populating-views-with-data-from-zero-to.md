---
layout: post
title: Populating views with data, from zero to Hero
date: '2016-04-02T11:00:00.000+01:00'
author: luciano_marisi
tags:
- software architecture
- swift
- design patterns
permalink: /2016/03/populating-views-with-data-from-zero-to.html
---

**TL;DR - Stop putting your data source on your view controller when populating a table view or a collection view**

As iOS developers we have had to populate a `UITableView` countless amounts of times. A lot of this times we write the same code in the same application multiple times. This post is about thinking about go a table view can be populated in a way that provides testable and reusable components. Note that I'm assuming that you're familiar with how to populate a `UITableView` and you have a *reasonable* knowledge about Swift. The post is not meant to be read as a tutorial but more like a journey to understand how the architecture can be improved. The code from this post can be downloaded from [DataArchitectureSample](https://github.com/lucianomarisi/DataArchitectureSample).

## Problem statement

Let's say you have to complete the following task:

<p align="center">
	<i>"Display a list of countries using a custom style."</i>
</p>

For the sake of simplicity this could look something like this:

<p align="center">
	<img src ="/assets/images/basic_example.gif"/>
</p>

*Disclaimer: No idea if the list of countries is correct, got it from a dodgy looking website...*

## Ways of solving this problem

To start solving this problem I will define some types to provide context. Let's say we have already created a new Xcode project with a `UIViewController` subclass with a `UITableView` that has a custom `UITableViewCell` with a nib (called `CountryTableViewCell`).

```swift
final class CountryTableViewCell : UITableViewCell {
  @IBOutlet var countryNameLabel: UILabel!
}
```

Also for simplicity, each country is represented by a very basic `Country` struct:

```swift
struct Country {
  let name: String
}
```
A convenience struct returns a mock list of countries:

```swift
struct CountryLoader {
  static func allCountries() -> [Country] { ... } // Returns an array of countries  
}
```

### 1. MVC (Massive View Controller)

The first solution and most basic solution is the gold old Massive View Controller. This barely needs introduction, effectively this *pattern* suggests that everthing should go into one view controller. For example, in this case the conformance to `UITableViewDataSource`. The code is pretty self explanatory, so I won't go into much detail:

```swift
private let reuseIdentifier = "CountryTableViewCell"

final class MassiveViewController: UIViewController, UITableViewDataSource {

  @IBOutlet weak var tableView: UITableView!
  
  private let countries: [Country] = CountryLoader.allCountries()
  
  override func viewDidLoad() {
    super.viewDidLoad()
    tableView.dataSource = self
    let customCellNib = UINib(nibName: "\(CountryTableViewCell.self)", bundle: nil)
    tableView.registerNib(customCellNib, forCellReuseIdentifier: reuseIdentifier)
  }
  
  //MARK: UITableViewDataSource
  
  func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
    guard let cell = tableView.dequeueReusableCellWithIdentifier(reuseIdentifier) as? CountryTableViewCell else {
      fatalError("Could not dequeue cell with identifier: \(reuseIdentifier)")
    }
    let country = countries[indexPath.row]
    cell.countryNameLabel?.text = country.name
    return cell
  }
  
  func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return countries.count
  }

}
```

Naturally this doesn't look very massive since it's only a `UIViewController` with a very simple `UITableView` without a `UITableViewDelegate`. However, hopefully it's possible to understand how this can get very big very quickly after adding table view sections and user interaction among other things. It would quite hard to write unit tests for this code. If you had to draw the architecure of this, it would like this...

<p align="center">
	<img src ="/assets/images/MassiveViewController.png"/>
</p>


### 2. Extracting the `UITableViewDataSource` to another object

A very good improvement to the MVC pattern is extract the `UITableViewDataSource` onto another object. I first read about this in 2013 on the [Lighter View Controllers](https://www.objc.io/issues/1-view-controllers/lighter-view-controllers/) article from objc.io's first issue. For our case, the code would look something like this:

```swift
private let reuseIdentifier = "CountryTableViewCell"

final class LighterViewController: UIViewController {
  
  @IBOutlet weak var tableView: UITableView!
  
  let countriesDataSource = CountriesDataSource(reuseIdentifier: reuseIdentifier)
  
  override func viewDidLoad() {
    super.viewDidLoad()
    tableView.dataSource = countriesDataSource
    let customCellNib = UINib(nibName: "\(CountryTableViewCell.self)", bundle: nil)
    tableView.registerNib(customCellNib, forCellReuseIdentifier: reuseIdentifier)
  }
  
}
```

```swift
final class CountriesDataSource: NSObject, UITableViewDataSource {
  
  private let countries: [Country] = CountryLoader.allCountries()
  private let reuseIdentifier: String
  
  init(reuseIdentifier: String) {
    self.reuseIdentifier = reuseIdentifier
  }
  
  func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
    guard let cell = tableView.dequeueReusableCellWithIdentifier(reuseIdentifier) as? CountryTableViewCell else {
      fatalError("Could not dequeue cell with identifier: \(reuseIdentifier)")
    }
    let country = countries[indexPath.row]
    cell.countryNameLabel?.text = country.name
    return cell
  }
  
  func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return countries.count
  }
  
}
```

This is certainly an improvement since now we have a much smaller view controller. We could add some unit tests to the `CountriesDataSource` and reuse it somewhere else if we wished to as well. This could be drawn like this:

<p align="center">
	<img src ="/assets/images/DataSource.png"/>
</p>

### 3. Introducing the DataProvider

The previous idea is great and I have used it for quite some time but it could be improved. The data itself is a bit too coupled to the `UITableViewDataSource`. We could extract this responsibility into a new object. This new object could be defined by a new `DataProvider` protocol that uses an associatedtype to define the model object:

```swift
protocol DataProvider {
  
  associatedtype DataObject
  
  func objectAtIndexPath(indexPath: NSIndexPath) -> DataObject?
  func numberOfItemsInSection(section: Int) -> Int
}
```

Then we can create a `CountriesDataProvider` type that will provide the countries data to the data source:

```swift
struct CountriesDataProvider : DataProvider {
  
  private let allCountries: [Country] = CountryLoader.allCountries()
  
  func objectAtIndexPath(indexPath: NSIndexPath) -> Country? {
    return allCountries.indices.contains(indexPath.row) ? allCountries[indexPath.row] : nil
  }
  
  func numberOfItemsInSection(section: Int) -> Int {
    return allCountries.count
  }
}
```

This may not seem that much because the example is so simple but now we have extracted the data from the table view datasource. This means we can now easily test the data is provided correctly, for example if the order is correct. Similarly, the source of the data could be changed by swapping only the `CountriesDataProvider` for another object that conforms to the `DataProvider` protocol. For example, if the data now comes from CoreData instead of an array of countries.

The next step is to move the registering of the custom nib for the cell to our original `CountriesDataSource` and make it retrieve the data from the `CountriesDataProvider`. At this point I have renamed the `CountriesDataSource` because the responsibility has changed slightly as it no longer is just a data source.

```swift
private let reuseIdentifier = "CountryTableViewCell"

final class CountriesDataCoordinator : NSObject, UITableViewDataSource {
  
  let countriesDataProvider = CountriesDataProvider()
  
  init(tableView: UITableView) {
    super.init()
    let customCellNib = UINib(nibName: "\(CountryTableViewCell.self)", bundle: nil)
    tableView.registerNib(customCellNib, forCellReuseIdentifier: reuseIdentifier)
    tableView.dataSource = self
  }
  
  //MARK: UITableViewDataSource
  
  func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
    guard let cell = tableView.dequeueReusableCellWithIdentifier(reuseIdentifier) as? CountryTableViewCell else {
      fatalError("Could not dequeue CountryTableViewCell with identifier: \(reuseIdentifier)")
    }
    
    if let country = countriesDataProvider.objectAtIndexPath(indexPath) {
      cell.countryNameLabel?.text = country.name
    }
    
    return cell
  }
  
  func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return countriesDataProvider.numberOfItemsInSection(section)
  }
}
```

A view controller that uses the `CountriesDataCoordinator` would look similar to the previous one though slightly lighter as there is no reference to a custom `UITableViewCell` (Why should the view controller need to know about the type of cell anyways?):

```swift
final class DataCoordinatorViewController: UIViewController {
  
  @IBOutlet weak var tableView: UITableView!
  
  var countriesDataCoordinator: CountriesDataCoordinator?
  
  override func viewDidLoad() {
    super.viewDidLoad()
    countriesDataCoordinator = CountriesDataCoordinator(tableView: tableView)
  }
  
}
```

This new architecture could be drawn like this:

<p align="center">
	<img src ="/assets/images/DataCoordinator.png"/>
</p>

### 4. Using generics to rewriting the `UITableViewDataSource` code

As you may or may not have figured out from the title (the \<Hero> part) the final solution I will explain will use generics. Wouldn't it be great if we didn't have to write the `UITableViewDataSource` and `UICollectionViewDataSource` over and over again? As an iOS developer I rarely write anything custom for these protocol that cannot be reused. After several years, I'm tired of writing this, and I'll like my code [DRY](https://en.wikipedia.org/wiki/Don't_repeat_yourself).

The first thing to do is create a `ConfigurableCell` that the custom cell will conform to. There is an `associatedtype` used for the model object that will be used to configure the cell.

```swift
protocol ConfigurableCell {
  
  associatedtype DataObject
 
  func configureForObject(object: DataObject)
 
  static func reuseIdentifier() -> String
}
```

Making our original `CountryTableViewCell` conform to `ConfigurableCell` would look something like this:

```swift
extension CountryTableViewCell: ConfigurableCell {
  func configureForObject(country: Country) {
    countryNameLabel.text = country.name
  }
  
  static func reuseIdentifier() -> String {
    return "CountryTableViewCell"
  }
}
```

Now comes one of the hardest things to understand about this post. We need to make our original data coordinator into a generic `TableViewDataCoordinator`. This is key to avoid writing the `UITableViewDataSource` code over and over again. The generic defition could be broken down in into 4 parts:

1. A generic `DataProviderType` needs to conform to `DataProvider`
	- i.e. `DataProviderType: DataProvider`
- A generic `CellType` needs to inherit from `UITableViewCell`
 	- i.e. `CellType: UITableViewCell`
- The generic `CellType` needs to conform to `ConfigurableCell`
 	- i.e. `CellType: ConfigurableCell`
- The `DataObject` from the `CellType` needs to be the same as the `DataObject` from the `DataProviderType`, so that the objects the data provider vends can be used to configure the cell
 	- i.e. `CellType.DataObject == DataProviderType.DataObject`

Putting all these statements together we get:

```swift
<DataProviderType: DataProvider, CellType: UITableViewCell where CellType: ConfigurableCell, CellType.DataObject == DataProviderType.DataObject>
```

Now that we have our generic definition for the generic data coordinator, we now replace the specific types from the previous `CountriesDataCoordinator` with the generic types and update the cell using the new `configureForObject(_:)` method.

```swift
class GenericTableViewDataCoordinator<DataProviderType: DataProvider, CellType: UITableViewCell where CellType: ConfigurableCell, CellType.DataObject == DataProviderType.DataObject> : NSObject, UITableViewDataSource {
  
  private let dataProvider: DataProviderType
  
  init(tableView: UITableView, dataProvider: DataProviderType) {
    self.dataProvider = dataProvider
    super.init()
    tableView.dataSource = self
  }

  //MARK: UITableViewDataSource
  
  func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
    
    guard let cell = tableView.dequeueReusableCellWithIdentifier(CellType.reuseIdentifier()) as? CellType else {
      fatalError("Could not dequeue cell of type: \(CellType.self) with identifier: \(CellType.reuseIdentifier())")
    }
    
    if let object = dataProvider.objectAtIndexPath(indexPath) {
      cell.configureForObject(object)
    }
    
    return cell
  }
  
  func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return dataProvider.numberOfItemsInSection(section)
  }
  
}
```

Subclassing the reusable `GenericTableViewDataCoordinator` and specializing it with our previous `CountriesDataProvider` and `CountryTableViewCell` we create the `CountriesTableViewDataCoordinator`. This can then be used in a view controller to populate a table view.

```swift
final class CountriesTableViewDataCoordinator : GenericTableViewDataCoordinator<CountriesDataProvider, CountryTableViewCell> {
  
  init(tableView: UITableView, countriesDataProvider: CountriesDataProvider = CountriesDataProvider()) {
    let customCellNib = UINib(nibName: "\(CountryTableViewCell.self)", bundle: nil)
    tableView.registerNib(customCellNib, forCellReuseIdentifier: CountryTableViewCell.reuseIdentifier())
    super.init(tableView: tableView, dataProvider: countriesDataProvider)
  }
  
}
```

These concepts can be applied in a very similar fashion to create a `GenericCollectionViewDataCoordinator` and `CountriesTableViewDataCoordinator`. If it's not immediately obvious you can check the files on the [github repository](https://github.com/lucianomarisi/DataArchitectureSample). Putting it altogether into a view controller: 

```swift
final class GenericDataCoordinatorViewController: UIViewController {
  
  @IBOutlet weak var tableView: UITableView!
  @IBOutlet weak var collectionView: UICollectionView!
  
  var tableViewDataCoordinator: CountriesTableViewDataCoordinator?
  var collectionViewDataCoordinator: CountriesCollectionViewDataCoordinator?

  override func viewDidLoad() {
    super.viewDidLoad()
    let countriesDataProvider = CountriesDataProvider()
    // Example of how the CountriesDataProvider can be reuse for different coordinators
    tableViewDataCoordinator = CountriesTableViewDataCoordinator(tableView: tableView, countriesDataProvider: countriesDataProvider)
    collectionViewDataCoordinator = CountriesCollectionViewDataCoordinator(collectionView: collectionView, countriesDataProvider: countriesDataProvider)
  }
}
```

This new architecture could be drawn like this:

<p align="center">
	<img src ="/assets/images/GenericDataCoordinator.png"/>
</p>

## Conclusion

Naturally, when comparing the MassiveViewController example with the generic data coordinator example there is a lot more code. However, it's important to understand that this "extra" code will only need to be written once and so in most cases there is huge value in following this pattern.

There is definitely a lot to take in and the concepts describe here are by no means new. The basic concepts come from reading the [Lighter View Controllers](https://www.objc.io/issues/1-view-controllers/lighter-view-controllers/) article from objc.io and the code from [SPXDataSources](https://github.com/shaps80/SPXDataSources). Swift has allowed me to extend some of these concepts further by using generics, to have reusable and type-safe components. In the interest of keeping an already long post short, this architecture is only the tip of the iceberg and hopefully it's a good starting point. For example, it can be extended to use view models, to avoid having to pass the model itself to the cell.

### Advantages

- Highly reusable, it's very easy to setup a new `UITableView` once the pattern is implemented, i.e. [DRY](https://en.wikipedia.org/wiki/Don't_repeat_yourself)
- Very modular approach where the objects have clear responsiblity, i.e [SRP](https://en.wikipedia.org/wiki/Single_responsibility_principle)
- Easy to test that the various components do what they are meant to do
- Easy to interchange `UITableView` for `UICollectionView` and viceversa if needed
- Easy to change the data backing, e.g. from backing a `UITableView` with a static array to an array of CoreData entities using a `NSFetchedResultsController`

### Limitations

- One reusable identifier per cell class as per the `ConfigurableCell` protocol
- If multiple cell classes are needed, some type safety will be lost as the `DataObject` would need to be a protocol or a base class
- Going from the "Massive View Controller" approach to generic data coordinators, may take some time to get you head around it if you are new to Swift and generics