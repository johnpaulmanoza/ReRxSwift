# ReRxSwift
![Swift Version 4](https://img.shields.io/badge/Swift-v4-yellow.svg)

*[RxSwift][1] bindings for [ReSwift][2]*

## Introduction

In case you don't know them yet, these are two awesome frameworks:

- [ReSwift][2] is a small framework that implement the
  [unidirectional data flow architecture][3] in Swift. It is inspired
  by the JavaScript implementation of that architecture: [Redux][4].
- [RxSwift][1] is a Swift implementation of [ReactiveX][5]. Amongst
  other things, it allows you to easily bind properties to user interface
  elements.

Using those two frameworks you can make very nice app architectures.
RxSwift allows you to bind application state to your UI, and ReSwift
emits state updates in response to actions. We take it one step further
though.

Similar to [react-redux][6], ReRxSwift allows you to create view
controllers that have `props` and `actions`. View controllers read all
data they need from their `props` (instead of directly from the state),
and they change data by invoking callbacks defined by `actions` (instead
of directly dispatching ReSwift actions). This has some nice advantages:

- Better separation of concerns. It is easier to understand what your
  view controller does and what data it uses.
- Unit-testing. Because of the separation of concerns, you can easily
  unit-test your view controllers, all the way up to the Interface
  Builder connections.
- Better reusability. Reusing your view controllers is as simple as
  specifying different mappings from state to `props` and from ReSwift
  actions to `actions`. (See also section 'Open Issues' below.)
- Rapid prototyping. You can easily use dummy `props` and `actions` so
  that you get a working UI layer prototype. Without writing any of your
  application's business logic, you can implement your presentation
  layer in such a way that it is very simple to replace the dummies
  with real state and actions.

## Usage

This section assumes that there is a global variable `store` that contains
your app's store, and that it's type is `Store<AppState>`. You have a view
controller that manages a text field; the text field displays a value from
your state, and on `editingDidEnd` you trigger an action to store the text
field's content back into your state. To use ReRxSwift for your view
controller `MyViewController`, you use the following steps.

1. Create an extension to your view controller to make it `Connectable`,
   defining the `Props` and `Actions` that your view controller needs:

   ```swift
   extension MyViewController: Connectable {
       struct Props {
           let text: String
       }
       struct Actions {
           let updatedText: (String) -> Void
       }
   }
   ```

2. Define how your state is mapped to the above `Props` type:

   ```swift
   private let mapStateToProps = { (appState: AppState) in
       return MyViewController.Props(
           text: appState.content
       )
   }
   ```

3. Define the actions that are dispatched:

   ```swift
   private let mapDispatchToActions = { (dispatch: @escaping DispatchFunction) in
       return MyViewController.Actions(
           updatedText: { newText in dispatch(SetContent(newContent: newText)) }
       )
   }
   ```

4. Define the connection and hook it up:

   ```swift
   class MyViewController: UIViewController {
       @IBOutlet weak var textField: UITextField!

       let connection = Connection(
           store: store,
           mapStateToProps: mapStateToProps,
           mapDispatchToActions: mapDispatchToActions
       )

       override func viewWillAppear(_ animated: Bool) {
           super.viewWillAppear(animated)
           connection.connect()
       }

       override func viewDidDisappear(_ animated: Bool) {
           super.viewDidDisappear(animated)
           connection.disconnect()
       }
   }
   ```

5. Bind the text field's text:

   ```swift
   override func viewDidLoad() {
       super.viewDidLoad()
       connection.bind(\Props.text, to: textField.rx.text)
   }
   ```

6. Call the action:

   ```swift
   @IBAction func editingChanged(_ sender: UITextField) {
       actions.updatedText(sender.text ?? "")
   }
   ```

This is pretty much the [`SimpleTextFieldViewController`][10] inside the sample
app. That view controller comes with complete unit tests:
[`SimpleTextFieldViewControllerSpec`][11].

## Note: Swift 4

This project depends on Swift 4, because it uses [key paths][7] in its
API. I am currently using Xcode 9.0 beta 6 and the Swift compiler that
comes with it. Some earlier Xcode 9.0 betas fail at compiling this
project!

As soon as Xcode 9 and Swift 4 are released, and RxSwift and ReSwift
have migrated to Swift 4, I will formally release this framework through
CocoaPods et al.

## Status: Feedback Requested

This project is fully functional and usable. I recommend using it for
any project that already use ReSwift and RxSwift - and for projects that
don't, because I really believe you get better app architectures. I
intend to add more examples and publish this (CocoaPods, Swift package
manager, Carthage support).

Having said that, I am curious to know what you think about this
framework. Do you agree with the advantages? What do you think about
the API? Do you see any improvement opportunities? Please let me know
by opening [issues on GitHub][8].


## API

*todo*

## Example App
The folder `Examples` contains the following examples:

- **SimpleTextField**: Most basic use case of ReRxSwift, containing a text field that has its value bound, and an action.
- **SteppingUp**: Use case with multiple bound values and actions, also showing how to transform values when binding them.


## Open Issue: Make ReRx-enabled view controllers strictly pure

When using `react` and `redux`, there is a module called `react-redux`
that is the inspiration for ReRxSwift. Using that module allows you
to cleanly separate _stateful_ and _pure_ components. I would like to
create the same strict separation in ReRxSwift, but I don't know how
to do that yet. I created an [issue on GitHub for this][9], I am inviting
you to join the discussion there!


[1]: https://github.com/ReactiveX/RxSwift
[2]: https://github.com/ReSwift/ReSwift
[3]: http://redux.js.org/docs/basics/DataFlow.html
[4]: https://github.com/reactjs/redux
[5]: http://reactivex.io
[6]: http://redux.js.org/docs/basics/UsageWithReact.html
[7]: https://github.com/apple/swift-evolution/blob/master/proposals/0161-key-paths.md
[8]: https://github.com/svdo/ReRxSwift/issues
[9]: https://github.com/svdo/ReRxSwift/issues/1
[10]: https://github.com/svdo/ReRxSwift/blob/master/Example/SimpleTextField/SimpleTextFieldViewController.swift
[11]: https://github.com/svdo/ReRxSwift/blob/master/ExampleTests/SimpleTextField/SimpleTextFieldViewControllerSpec.swift
