# ReactiveReSwift

[![Build Status](https://img.shields.io/travis/ReSwift/ReactiveReSwift/master.svg?style=flat-square)](https://travis-ci.org/ReSwift/ReactiveReSwift) [![Code coverage status](https://img.shields.io/codecov/c/github/ReSwift/ReactiveReSwift.svg?style=flat-square)](http://codecov.io/github/ReSwift/ReactiveReSwift) [![Platform support](https://img.shields.io/badge/platform-ios%20%7C%20osx%20%7C%20tvos%20%7C%20watchos-lightgrey.svg?style=flat-square)](https://github.com/ReSwift/ReactiveReSwift/blob/master/LICENSE.md) [![License MIT](https://img.shields.io/badge/license-MIT-blue.svg?style=flat-square)](https://github.com/ReSwift/ReactiveReSwift/blob/master/LICENSE.md)

**Supported Swift Version:**: Swift 3.0.1

# Introduction

ReactiveReSwift is a [Redux](https://github.com/reactjs/redux)-like implementation of the unidirectional data flow architecture in Swift, derived from [ReSwift](https://github.com/ReSwift/ReSwift). ReactiveReSwift helps you to separate three important concerns of your app's components:

- **State**: in a ReactiveReSwift app the entire app state is explicitly stored in a data structure. This helps avoid complicated state management code, enables better debugging and has many, many more benefits...
- **Views**: in a ReactiveReSwift app your views update when your state changes. Your views become simple visualizations of the current app state.
- **State Changes**: in a ReactiveReSwift app you can only perform state changes through actions. Actions are small pieces of data that describe a state change. By drastically limiting the way state can be mutated, your app becomes easier to understand and it gets easier to work with many collaborators.

The ReactiveReSwift library is tiny - allowing users to dive into the code, understand every single line and [hopefully contribute](#contributing).

Check out our [public gitter chat!](https://gitter.im/ReSwift/public)

# Table of Contents

- [About ReactiveReSwift](#about-reswift)
- [Why ReactiveReSwift?](#why-reswift)
- [Getting Started Guide](#getting-started-guide)
- [Installation](#installation)
- [Testing](#testing)
- [Checking Out Source Code](#checking-out-source-code)
- [Demo](#demo)
- [Extensions](#extensions)
- [Example Projects](#example-projects)
- [Contributing](#contributing)
- [Credits](#credits)
- [Get in touch](#get-in-touch)

# About ReactiveReSwift

ReactiveReSwift relies on a few principles:
- **The Store** stores your entire app state in the form of a single data structure. This state can only be modified by dispatching Actions to the store. Whenever the state in the store changes, the store will notify all observers.
- **Actions** are a declarative way of describing a state change. Actions don't contain any code, they are consumed by the store and forwarded to reducers. Reducers will handle the actions by implementing a different state change for each action.
- **Reducers** provide pure functions, that based on the current action and the current app state, create a new app state

![](Docs/img/reswift_concept.png)

For a very simple app, that maintains a counter that can be increased and decreased, you can define the app state as following:

```swift
struct AppState: StateType {
  let counter: Int
}
```

You would also define two actions, one for increasing and one for decreasing the counter. In the [Getting Started Guide](http://reswift.github.io/ReactiveReSwift/master/getting-started-guide.html) you can find out how to construct complex actions. For the simple actions in this example we can an enum that conforms to action:

```swift
enum AppAction: Action {
	case Increase
    case Decrease
}
```

Your reducer needs to respond to these different actions, that can be done by switching over the value of action:

```swift
struct AppReducer: Reducer {
	func handleAction(action: Action, state: AppState) -> AppState {
		return AppState(
          counter: counterReducer(action: action, counter: state.counter)
		)
	}
}

func counterReducer(action: Action, counter: Int) -> Int {
	switch action as? AppAction {
	case .Increase?:
        return counter + 1
	case .Decrease?:
        return max(0, counter - 1)
	default:
		return counter
	}
}
```
In order to have a predictable app state, it is important that the reducer is always free of side effects, it receives the current app state and an action and returns the new app state.

To maintain our state and delegate the actions to the reducers, we need a store. Let's call it `mainStore` and define it as a global constant, for example in the app delegate file:

```swift
let mainStore = ObservableStore(
  reducer: AppReducer(),
  stateType: AppState.self,
  observable: ObservableProperty(AppState(counter: 0))
)

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
	[...]
}
```


Lastly, your view layer, in this case a view controller, needs to tie into this system by subscribing to store updates and emitting actions whenever the app state needs to be changed:

```swift
class CounterViewController: UIViewController {

  private let disposeBag = SubscriptionReferenceBag()
  @IBOutlet var counterLabel: UILabel!

  override func viewDidLoad() {
    disposeBag += mainStore.observable.subscribe { [weak self] state in
      self?.counterLabel.text = "\(state.counter)"
    }
  }

  @IBAction func increaseButtonTapped(sender: UIButton) {
    mainStore.dispatch(
      AppState.CounterActionIncrease
    )
  }

  @IBAction func decreaseButtonTapped(sender: UIButton) {
    mainStore.dispatch(
      AppState.CounterActionDecrease
    )
  }

}
```

The `mainStore.observable.subscribe` block will be called by the `ObservableStore` whenever a new app state is available, this is where we need to adjust our view to reflect the latest app state.

Button taps result in dispatched actions that will be handled by the store and its reducers, resulting in a new app state.

This is a very basic example that only shows a subset of ReSwift's features, read the Getting Started Guide to see how you can build entire apps with this architecture. For a complete implementation of this example see the [CounterExample](https://github.com/ReSwift/RxCounterExample) project.

[You can also watch this talk on the motivation behind ReSwift](https://realm.io/news/benji-encz-unidirectional-data-flow-swift/).

# Reactive Extensions

Here are some examples of what your code would look like if you were to leverage certain FRP libraries when writing your application.

[Documentation for ReactiveSwift](https://github.com/ReSwift/ReactiveReSwift/Docs/Reactive/RAC.md)
[Documentation for ReactiveKit](https://github.com/ReSwift/ReactiveReSwift/Docs/Reactive/ReactiveKit.md)
[Documentation for RxSwift](https://github.com/ReSwift/ReactiveReSwift/Docs/Reactive/RxSwift.md)

# Why ReactiveReSwift?

Model-View-Controller (MVC) is not a holistic application architecture. Typical Cocoa apps defer a lot of complexity to controllers since MVC doesn't offer other solutions for state management, one of the most complex issues in app development.

Apps built upon MVC often end up with a lot of complexity around state management and propagation. We need to use callbacks, delegations, Key-Value-Observation and notifications to pass information around in our apps and to ensure that all the relevant views have the latest state.

This approach involves a lot of manual steps and is thus error prone and doesn't scale well in complex code bases.

It also leads to code that is difficult to understand at a glance, since dependencies can be hidden deep inside of view controllers. Lastly, you mostly end up with inconsistent code, where each developer uses the state propagation procedure they personally prefer. You can circumvent this issue by style guides and code reviews but you cannot automatically verify the adherence to these guidelines.

ReactiveReSwift attempts to solve these problem by placing strong constraints on the way applications can be written. This reduces the room for programmer error and leads to applications that can be easily understood - by inspecting the application state data structure, the actions and the reducers.

This architecture provides further benefits beyond improving your code base:

- Stores, Reducers, Actions and extensions are entirely platform independent - you can easily use the same business logic and share it between apps for multiple platforms (iOS, tvOS, etc.)
- Want to collaborate with a co-worker on fixing an app crash? Use [ReSwift Recorder](https://github.com/ReSwift/ReSwift-Recorder) to record the actions that lead up to the crash and send them the JSON file so that they can replay the actions and reproduce the issue right away.
- Maybe recorded actions can be used to build UI and integration tests?

The ReactiveReSwift tooling is still in a very early stage, but aforementioned prospects excite me and hopefully others in the community as well!

# Getting Started Guide

[A Getting Started Guide that describes the core components of apps built with ReactiveReSwift lives here](http://reswift.github.io/ReactiveReSwift/master/getting-started-guide.html). It will be expanded in the next few weeks. To get an understanding of the core principles we recommend reading the brilliant [redux documentation](http://redux.js.org/).

# Installation

## Carthage

You can install ReSwift via [Carthage](https://github.com/Carthage/Carthage) by adding the following line to your `Cartfile`:

```
github "ReSwift/ReactiveReSwift"
```

## Swift Package Manager

You can install ReSwift via [Swift Package Manager](https://swift.org/package-manager/) by adding the following line to your `Package.swift`:

```
import PackageDescription

let package = Package(
    [...]
    dependencies: [
        .Package(url: "https://github.com/ReSwift/ReSwift.git", majorVersion: XYZ)
    ]
)
```

# Checking out Source Code

ReactiveReSwift no longer has any carthage dependencies for development. Just checkout the project and run.

# Example Projects

- [CounterExample](https://github.com/ReSwift/RxCounterExample): A very simple counter app implemented with ReactiveReSwift.

# Contributing

There's still a lot of work to do here! We would love to see you involved! You can find all the details on how to get started in the [Contributing Guide](/CONTRIBUTING.md).

# Credits

- Thanks a lot to [Dan Abramov](https://github.com/gaearon) for building [Redux](https://github.com/reactjs/redux) - all ideas in here and many implementation details were provided by his library.

# Get in touch

If you have any questions, you can find the core team on twitter:

- [@benjaminencz](https://twitter.com/benjaminencz)
- [@karlbowden](https://twitter.com/karlbowden)
- [@ARendtslev](https://twitter.com/ARendtslev)
- [@ctietze](https://twitter.com/ctietze)
- [@chartortorella](https://twitter.com/chartortorella)

We also have a [public gitter chat!](https://gitter.im/ReSwift/public)
