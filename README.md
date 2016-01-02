![GK](http://www.graphkit.io/GK/GraphKit.png)

# Welcome to GraphKit

GraphKit is a data and algorithm framework built on top of CoreData. It is available for iOS and OS X. A major goal in the design of GraphKit is to allow data to be modeled as one would think. The following README is written to get you started, and is by no means a complete tutorial on all that is possible.

### CocoaPods Support

GraphKit is on CocoaPods under the name [GK](https://cocoapods.org/?q=GK).

### Carthage Support

Carthage is a decentralized dependency manager that builds your dependencies and provides you with binary frameworks.

You can install Carthage with Homebrew using the following command:

```bash
$ brew update
$ brew install carthage
```
To integrate GraphKit into your Xcode project using Carthage, specify it in your Cartfile:

```bash
github "CosmicMind/GraphKit"
```

Run carthage to build the framework and drag the built GraphKit.framework into your Xcode project.

### Table of Contents  

* [Entity](#entity)
* [Bond](#bond)
* [Action](#action)
* [Groups](#groups)
* [Probability](#probability)
* [Data Driven](#datadriven)
* [Faceted Search](#facetedsearch)
* [Data Structures](#datastructures)
* [DoublyLinkedList](#doublylinkedlist)
* [Stack](#stack)
* [Queue](#queue)
* [Deque](#deque)
* [RedBlackTree](#redblacktree)
* [SortedSet](#sortedset)
* [SortedMultiSet](#sortedmultiset)
* [SortedDictionary](#sorteddictionary)
* [SortedMultiDictionary](#sortedmultidictionary)

### Upcoming

* Example Projects
* Additional Data Structures

<a name="entity"/>
### Entity

Let's begin with creating a simple model object and saving it to the Graph. Model objects are known as Entity Objects, which represent a person, place, or thing. Each Entity has a type property that specifies the collection to which it belongs to. Below is an example of creating a "User" type Entity.

```swift
let graph: Graph = Graph()

let user: Entity = Entity(type: "User")
user["name"] = "Eve"
user["age"] = 27

graph.save()
```

<a name="bond"/>
### Bond

A Bond is used to form a relationship between two Entity Objects. Like an Entity, a Bond also has a type property that specifies the collection to which it belongs to. A Bond's relationship structure is like a sentence, in that it has a Subject and Object. Let's look at an example to clarify this concept. Below is an example of two Entity Objects, a User and a Book, that have a relationship that is defined by the User being the Author of the Book. The relationship should read as, "User is Author of Book."

```swift
let graph: Graph = Graph()

let user: Entity = Entity(type: "User")
user["name"] = "Michael Talbot"

let book: Entity = Entity(type: "Book")
book["title"] = "The Holographic Universe"
book.addGroup("Physics")

let author: Bond = Bond(type: "Author")
author["written"] = "May 6th 1992"
author.subject = user
author.object = book

graph.save()
```

<a name="action"/>
### Action

An Action is used to form a relationship between many Entity Objects. Like an Entity, an Action also has a type property that specifies the collection to which it belongs to. An Action's relationship structure is like a sentence, in that it relates a collection of Subjects to a collection of Objects. Below is an example of a User purchasing many Books. It may be thought of as "User Purchased these Book(s)."

```swift
let graph: Graph = Graph()

let user: Entity = Entity(type: "User")
let books: Array<Entity> = graph.searchForEntity(types: ["Book"])

let purchased: Action = Action(type: "Purchased")
purchased.addSubject(user)

for book in books {
	purchased.addObject(book)
}

graph.save()
```

<a name="groups"/>
### Groups

Groups are used to organize Entities, Bonds, and Actions into different collections from their types. This allows multiple types to exist in a single collection. For example, a Photo and Video Entity type may exist in a group called Media. Another example may be including a Photo and Book Entity type in a Favorites group for your users' account. Below are examples of using groups.

```swift
// Adding a group.
let photo: Entity = Entity(type: "Photo")
photo.addGroup("Media")
photo.addGroup("Favorites")
photo.addGroup("Holiday Album")

let video: Entity = Entity(type: "Video")
video.addGroup("Media")

let book: Entity = Entity(type: "Book")
book.addGroup("Favorites")
book.addGroup("To Read")

// Searching groups.
let favorites: Array<Entity> = graph.searchForEntity(groups: ["Favorites"])
```

<a name="probability"/>
### Probability

Probability is a core feature within GraphKit. Your application may be completely catered to your users' habits and usage. To demonstrate this wonderful feature, let's look at some examples:

Determining the probability of rolling a 3 on a die of 6 numbers.

```swift
let die: Array<Int> = Array<Int>(arrayLiteral: 1, 2, 3, 4, 5, 6)
print(die.probabilityOf(3)) // output: 0.166666666666667
```

The expected value of rolling a 3 or 6, 100 times on a die of 6 numbers.

```swift
let die: Array<Int> = Array<Int>(arrayLiteral: 1, 2, 3, 4, 5, 6)
print(die.expectedValueOf(100, elements: 3, 6)) // output: 33.3333333333333
```

Recommending a Physics book if the user is likely to buy a Physics book.

```swift
let purchased: Array<Action> = graph.searchForAction(types: ["Purchased"])

let probabilityOfX: Double = purchased.probabilityOf { (action: Action) in
	if let entity: Entity = action.objects.first {
		if "Book" == entity.type {
			return entity.hasGroup("Physics")
		}
	}
	return false
}

if 50 < probabilityOfX {
	// Recommend a Physics book.
}
```

<a name="datadriven"/>
### Data Driven

As data moves through your application, the state of information may be observed to create a reactive experience. Below is an example of watching when a "User Clicked a Button".

```swift
// Set the UIViewController's Protocol to GraphDelegate.
let graph: Graph = Graph()
graph.delegate = self

graph.watchForAction(types: ["Clicked"])

let user: Entity = Entity(type: "User")
let clicked: Action = Action(type: "Clicked")
let button: Entity = Entity(type: "Button")

clicked.addSubject(user)
clicked.addObject(button)

graph.save()

// Delegate method.
func graphDidInsertAction(graph: Graph, action: Action) {
    switch(action.type) {
    case "Clicked":
      print(action.subjects.first?.type) // User
      print(action.objects.first?.type) // Button
    case "Swiped":
      // Handle swipe.
    default:break
    }
 }
```

<a name="facetedsearch"/>
### Faceted Search

To explore the intricate relationships within the Graph, the search API adopts a faceted design. This allows the exploration of your data through any view point. The below examples show how to use the Graph search API:

Searching multiple Entity types.

```swift
let collection: Array<Entity> = graph.searchForEntity(types: ["Photo", "Video"])
```

Searching Entity groups.

```swift
let collection: Array<Entity> = graph.searchForEntity(groups: ["Media", "Favorites"])
```

Searching Entity properties.

```swift
let collection: Array<Entity> = graph.searchForEntity(properties: [(key: "name", value: "Eve"), ("age", "27")])
```

Searching multiple facets simultaneously will aggregate all results into a single collection.

```swift
let collection: Array<Entity> = graph.searchForEntity(types: ["Photo", "Friends"], groups: ["Media", "Favorites"])
```

Filters may be used to narrow in on a search result. For example, searching a book title and group within purchases.

```swift
let collection: Array<Action> = graph.searchForAction(types: ["Purchased"]).filter { (action: Action) -> Bool in
	if let entity: Entity = action.objects.first {
		if "Book" == entity.type && "The Holographic Universe" == entity["title"] as? String  {
			return entity.hasGroup("Physics")
		}
	}
	return false
}
```

<a name="datastructures"/>
### Data Structures

GraphKit comes packed with some powerful data structures to help write algorithms. The following structures are included: List, Stack, Queue, Deque, RedBlackTree, SortedSet, SortedMultiSet, SortedDictionary, and SortedMultiDictionary.

<a name="doublylinkedlist"/>
### DoublyLinkedList

The List data structure is an implementation of a Doubly-Linked List, and is excellent for large growing collections of data.

### License

[AGPL-3.0](http://choosealicense.com/licenses/agpl-3.0/)
