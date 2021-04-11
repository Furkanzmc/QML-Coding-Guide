# Introduction

The content that you see here is mostly based on my experience working with QML
in a large project with a diverse team of developers and designers. I update
the document as my opinions about certain things become validated by real life
experience.

You may not agree with some of the ideas laid out here, in those cases please
create an issue to discuss and update accordingly. I'll keep updating this guide
as I learn new things. Contributions are vital to this document because it needs
to reflect tried and validated ideas of more people to make sense in a general
sense. It's likely that I may not have done a good job at explaining a concept.
I would appreciate any contributions to improve it.

Please don't hesitate to raise issues and submit PRs. Even the tiniest
contribution matters.

# Table of Contents

- [Code Style](#code-style)
    - [CS-1: Signal Handler Ordering](#cs-1-signal-handler-ordering)
    - [CS-2: Property Ordering](#cs-2-property-ordering)
    - [CS-3: Function Ordering](#cs-3-function-ordering)
    - [CS-4: Animations](#cs-4-animations)
    - [CS-5: Giving Components `id`s](#cs-5-giving-components-ids)
    - [CS-6: Property Assignments](#cs-6-property-assignments)
    - [CS-7: Import Statements](#cs-7-import-statements)
    - [Full Example](#full-example)
- [Bindings](#bindings)
    - [B-1: Prefer Bindings over Imperative Assignments](#b-1-prefer-bindings-over-imperative-assignments)
    - [B-2: Making `Connections`](#b-2-making-connections)
    - [B-3: Use `Binding` Object](#b-3-use-binding-object)
        + [Transient Bindings](#transient-bindings)
    - [B-4: KISS It](#b-4-kiss-it)
    - [B-5: Be Lazy](#b-5-be-lazy)
    - [B-6: Avoid Unnecessary Re-Evaluations](#b-6-avoid-unnecessary-re-evaluations)
- [C++ Integration](#c-integration)
    - [CI-1: Avoid Context Properties](#ci-1-avoid-context-properties)
    - [CI-2: Use Singleton for Common API Access](#ci-2-use-singleton-for-common-api-access)
    - [CI-3: Prefer Instantiated Types Over Singletons For Data](#ci-3-prefer-instantiated-types-over-singletons-for-data)
    - [CI-4: Watch Out for Object Ownership Rules](#ci-4-watch-out-for-object-ownership-rules)
- [Performance and Memory](#performance-and-memory)
    - [PM-1: Reduce the Number of Implicit Types](#pm-1-reduce-the-number-of-implicit-types)
- [Signal Handling](#signal-handling)
    - [SH-1: Try to Avoid Using connect Function in Models](#sh-1-try-to-avoid-using-connect-function-in-models)
    - [SH-2: When to use Functions and Signals](#sh-2-when-to-use-functions-and-signals)
- [JavaScript](#javascript)
    - [JS-1: Use Arrow Functions](#js-1-use-arrow-functions)
    - [JS-2: Use the Modern Way of Declaring Variables](#js-2-use-the-modern-way-of-declaring-variables)
- [States and Transitions](#states-and-transitions)
    - [ST-1: Don't Define Top Level States](#st-1-dont-define-top-level-states)
- [Visual Items](#visual-items)
    - [VI-1: Distinguish Between Different Types of Sizes](#vi-1-distinguish-between-different-types-of-sizes)

# Code Style

This section provides details about how to format the order of properties, signals,
and functions to make things easy on the eyes and quickly switch to related code block.

[QML object attributes](https://doc.qt.io/qt-5/qtqml-syntax-objectattributes.html)
are always structured in the following order:

- id
- Property declarations
- Signal declarations
- Object properties
- Attached properties and signal handlers
- States
- Transitions
- Signal handlers
- Child objects
  + Visual Items
  + Qt provided non-visual items
  + Custom non-visual items
- `QtObject` for encapsulating private members[1](https://bugreports.qt.io/browse/QTBUG-11984)
- JavaScript functions

The main purpose for this order is to make sure that the most intrinsic properties of a type is
always the most visible one in order to make the interface easier to digest at a first glance.
Although it could be argued that the JavaScript functions are also part of the interface, the ideal
is to have no functions at all.

## CS-1: Signal Handler Ordering

When handling the signals attached to an `Item`, make sure to always leave
`Component.onCompleted` to the last line.

```qml
// Wrong
Item {
    Component.onCompleted: {
    }
    onSomethingHappened: {
    }
}

// Correct
Item {
    onSomethingHappened: {
    }
    Component.onCompleted: {
    }
}
```

This is because it mentally makes for a better picture because
`Component.onCompleted` is expected to be fired when the components construction
is complete.

------

If there are multiple signal handlers in an `Item`, then the ones with least amount
of lines may be placed at the top. As the implementation lines increases, the handler
also moves down. The only exception to this is `Component.onCompleted` signal, it
is always placed at the bottom.

```qml
// Wrong
Item {
    onOtherEvent: {
        // Line 1
        // Line 2
        // Line 3
        // Line 4
    }
    onSomethingHappened: {
        // Line 1
        // Line 2
    }
}

// Correct
Item {
    onSomethingHappened: {
        // Line 1
        // Line 2
    }
    onOtherEvent: {
        // Line 1
        // Line 2
        // Line 3
        // Line 4
    }
}
```

## CS-2: Property Ordering

The first property assignment must always be the `id` of the component. If you
want to declare custom properties for a component, the declarations are always
above the first property assignment.

```qml
// Wrong
Item {
    someProperty: false
    property int otherProperty: -1
    id: myItem
}

// Correct
Item {
    id: myItem

    property int otherProperty: -1

    someProperty: false
}
```

There's also a bit of predefined order for property assignments. The order goes
as follows:

- id
- x
- y
- width
- height
- anchors

The goal here is to put the most obvious and defining properties at the top for
easy access and visibility. For example, for an `Image` you may decide to also
put `sourceSize` above `anchors`.

------

If there are also property assignments along with signal handlers, make sure to
always put property assignments above the signal handlers.

```qml
// Wrong
Item {
    onOtherEvent: {
    }
    someProperty: true
    onSomethingHappened: {
    }
    x: 23
    y: 32
}

// Correct
Item {
    x: 23
    y: 32
    someProperty: true
    onOtherEvent: {
    }
    onSomethingHappened: {
    }
}
```

It is usually harder to see the property assignments If they are mixed with
signal handlers. That's why we are putting the assignments above the signal
handlers.

### CS-3: Function Ordering

Although there are no private and public functions in QML, you can provide a
similar mechanism by wrapping the properties and functions that are only supposed
to be used internally in `QtObject `.

Public function implementations are always put at the very bottom of the file.

```qml
// Wrong
Item {

    function someFunction() {
    }

    someProperty: true
}

// Correct
Item {
    someProperty: true
    onOtherEvent: {
    }
    onSomethingHappened: {
    }

    function someFunction() {
    }
}
```

### CS-4: Animations

When using any subclass of `Animation`, especially nested ones like
`SequentialAnimation`, try to reduce the number of properties in one line.
More than 2-3 assignments on the same line becomes harder to reason with after
a while. Or maybe you can keep the one line assignments to whatever line length
convention you have set up for your project.

Since animations are harder to imagine in your mind, you will benefit from
keeping the animations as simple as possible.

```qml
// Bad
NumberAnimation { target: root; property: "opacity"; duration: root.animationDuration; from: 0; to: 1 }

// Depends on your convention. The line does not exceed 80 characters.
PropertyAction { target: root; property: "visible"; value: true }

// Good.
SequentialAnimation {
    PropertyAction {
        target: root
        property: "visible"
        value: true
    }

    NumberAnimation {
        target: root
        property: "opacity"
        duration: root.animationDuration
        from: 0
        to: 1
    }
}
```

### CS-5: Giving Components `id`s

If a component does not need to be accessed for a functionality, avoid setting
the `id` property. This way you don't clutter the namespace with unused `id`s and
you'll be less likely to run into duplicate `id` problem. Also, having an id for
a type puts additional cognitive stress because it now means that there's
additional relationships that we need to care for.

If you want to mark the type with a descriptor but you don't intend to reference
the type, you can use `objectName` instead old just plain old comments.

Make sure that the top most component in the file always has `root` as its `id`.
Qt will make unqualified name look up deprecated in QML 3, so it's better to
start giving IDs to your components now and use qualified look up.

See [QTBUG-71578](https://bugreports.qt.io/browse/QTBUG-71578) and
[QTBUG-76016](https://bugreports.qt.io/browse/QTBUG-76016) for more details
on this.

### CS-6: Property Assignments

When assigning grouped properties, always prefer the dot notation If you are only
altering just one property. Otherwise, always use the group notation.

```qml
Image {
    anchors.left: parent.left // Dot notation
    sourceSize { // Group notation
        width: 32
        height: 32
    }
}
```

When you are assigning the component to a `Loader`'s `sourceComponent` in different
places in the same file, consider using the same implementation. For example, in
the following example there are two instances of the same component. If both of
those `SomeSpecialComponent` are meant to be identical it is a better idea to
wrap `SomeSpecialComponent` in a `Component`.

```qml
// BEGIN bad.
Loader {
    id: loaderOne
    sourceComponent: SomeSpecialComponent {
        text: "Some Component"
    }
}

Loader {
    id: loaderTwo
    sourceComponent: SomeSpecialComponent {
        text: "Some Component"
    }
}
// END bad.

// BEGIN good.
Loader {
    id: loaderOne
    sourceComponent: specialComponent
}

Loader {
    id: loaderTwo
    sourceComponent: specialComponent
}

Component {
    id: specialComponent

    SomeSpecialComponent {
        text: "Some Component"
    }
}
// END good.
```

This ensures that whenever you make a change to `specialComponent` it will take
effect in all of the `Loader`s. In the bad example, you would have to duplicate
the same change.

When in a similar situation without the use of `Loader`, you can use inline
components.

```qml
component SomeSpecialComponent: Rectangle {

}
```

### CS-7: Import Statements

If you are importing a JavaScript file, make sure to not include the same
module in both the QML file and the JavaScript file. JavaScript files share the
imports from the QML file so you can take advantage of that. If the JavaScript
file is meant as a library, this does not apply.

If you are not making use of the imported module in the QML file, consider moving
the import statement to the JavaScript file. But note that once you import something
in the JavaScript file, the imports will no longer be shared. For the complete
rules see [here](https://doc.qt.io/qt-5/qtqml-javascript-imports.html#imports-within-javascript-resources).

Alternatively, you can use `Qt.include()` which copies the contents of the
included file and you will not have to worry about the import sharing rules.

As a general rule, you should avoid having unused import statements.

#### Import Order

When importing other modules, use the following order;

- Qt modules
- Third party modules
- Local C++ module imports
- QML folder imports

### Full Example

```qml
// First Qt imports
import QtQuick 2.15
import QtQuick.Controls 2.15
// Then custom imports
import my.library 1.0

Item {
    id: root

    // ----- Property Declarations

    // Required properties should be at the top.
    required property int radius: 0

    property int radius: 0
    property color borderColor: "blue"

    // ----- Signal declarations

    signal clicked()
    signal doubleClicked()

    // ----- In this section, we group the size and position information together.

    x: 0
    y: 0
    z: 0
    width: 100
    height: 100
    anchors.top: parent.top // If a single assignment, dot notation can be used.
    // If the item is an image, sourceSize is also set here.
    // sourceSize: Qt.size(12, 12)

    // ----- Then comes the other properties. There's no predefined order to these.

    // Do not use empty lines to separate the assignments. Empty lines are reserved
    // for separating type declarations.
    enabled: true
    layer.enabled: true

    // ----- Then attached properties and attached signal handlers.

    Layout.fillWidth: true
    Drag.active: false
    Drag.onActiveChanged: {

    }

    // ----- States and transitions.

    states: [
        State {

        }
    ]
    transitions: [
        Transitions {

        }
    ]

    // ----- Signal handlers

    onWidthChanged: { // Always use curly braces.

    }
    // onCompleted and onDestruction signal handlers are always the last in
    // the order.
    Component.onCompleted: {

    }
    Component.onDestruction: {

    }

    // ----- Visual children.

    Rectangle {
        height: 50
        anchors: { // For multiple assignments, use group notation.
            top: parent.top
            left: parent.left
            right: parent.right
        }
        color: "red"
        layer: {
            enabled: true
            samples: 4
        }
    }

    Rectangle {
        width: parent.width
        height: 1
        color: "green"
    }

    // ----- Qt provided non-visual children

    Timer {

    }

    // ----- Custom non-visual children

    MyCustomNonVisualType {

    }

    QtObject {
        id: privates

        property int diameter: 0
    }

    // ----- JavaScript functions

    function collapse() {

    }

    function setCollapsed(value: bool) {
        if (value === true) {
        }
        else {
        }
    }
}
```

# Bindings

Bindings are a powerful tool when used responsibly. Bindings are evaluated
whenever a property it depends on changes and this may result in poor performance
or unexpected behaviors. Even when the binding is simple, its consequence can be
expensive. For instance, a binding can cause the position of an item to change
and every other item that depends on the position of that item or is anchored to
it will also update its position.

So consider the following rules when you are using bindings.

## B-1: Prefer Bindings over Imperative Assignments

See the related section on [Qt Documentation](https://doc.qt.io/qt-5/qtquick-bestpractices.html#prefer-declarative-bindings-over-imperative-assignments).

The official documentation explains things well, but it is also important to
understand the performance complications of bindings and understand where the
bottlenecks can be.

If you suspect that the performance issue you are having is related to
excessive evaluations of bindings, then use the QML profiler to confirm your
suspicion and then opt-in to use imperative option.

Refer to the [official documentation](https://doc.qt.io/qtcreator/creator-qml-performance-monitor.html)
on how to use QML profiler.

## B-2: Making `Connections`

A `Connections` object is used to handle signals from arbitrary `QObject` derived
classes in QML. One thing to keep in mind when using connections is the default
value of `target` property of the `Connections` is its parent if not explicitly
set to something else. If you are setting the target after dynamically creating
a QML object, you might want to set the `target` to `null` otherwise you might
get signals that are not meant to be handled.

```qml
// Bad
Item {
    id: root
    onSomethingHappened: {
        // Set the target of the Connections.
    }

    Connections {
        // Notice that target is not set so it's implicitly set to root.
        onWidthChanged: {
            // Do something. But since Item also has a width property we may
            // handle the change for root until the target is set explicitly.
        }
    }
}

// Good
Item {
    id: root
    onSomethingHappened: {
        // Set the target of the Connections.
    }

    Connections {
        target: null // Good. Now we won't have the same problem.
        onWidthChanged: {
            // Do something. Only handles the changes for the intended target.
        }
    }
}
```

## B-3: Use `Binding` Object

`Binding`'s `when` property can be used to enable or disable a binding expression
depending on a condition. If the binding that you are using is complex and does
not need to be executed every time a property changes, this is a good idea to
reduce the binding execution count.

Using the same example above, we can rewrite it as follows using a `Binding` object.

```qml
Rectangle {
    id: root

    Binding on color {
        when: mouseArea.pressed
        value: mouseArea.pressed ? "red" : "yellow"
    }

    MouseArea {
        id: mouseArea
        anchors.fill: parent
    }
}
```

Again, this is a really simple example to get the point out. In a real life
situation, you would not get more benefit from using `Binding` object in this
case unless the binding expression is expensive (e.g It changes the item's
`anchor` which causes a whole chain reaction and causes other items to be
repositioned.).

`Binding` objects can also be used to provide bidirectional binding for
properties without the risk of breaking the bindings. Consider the following
example:

```qml
Rectangle {
    id: rect
    width: 50
    height: 50
    anchors.centerIn: parent
    color: cd.color

    MouseArea {
        anchors.fill: parent
        acceptedButtons: Qt.LeftButton | Qt.RightButton
        onClicked: {
            if (mouse.button === Qt.LeftButton) {
                cd.visible = true;
            }
            else {
                parent.color = "black"
            }
        }
    }
}

ColorDialog {
    id: cd
    color: rect.color
}
```

The binding to `color` properties of `ColorDialog` and `Rectangle` will be broken
once those proeprties are set from outside. If you play around with the example,
you'll see that `parent.color = "black"` breaks the binding.

Now, see the following example and you'll find that bindings are not broken.

```qml
Rectangle {
    id: rect
    width: 50
    height: 50
    anchors.centerIn: parent
    color: "red"

    Binding on color {
        value: cd.color
    }

    MouseArea {
        anchors.fill: parent
        acceptedButtons: Qt.LeftButton | Qt.RightButton
        onClicked: {
            if (mouse.button === Qt.LeftButton) {
                cd.visible = true;
            }
            else {
                parent.color = "black"
            }
        }
    }
}

ColorDialog {
    id: cd

    Binding on color {
        value: rect.color
    }
}
```

### Transient Bindings

There may be cases where you have to end up using an imperative assignment. But
naturally this will break the binding. In that case, you can create transient
`Binding` objects to safely set the new property without breaking the existing
binding.

```qml
Item {
    property var contentItem

    onContentItemChanged: {
        contentItem.width = 100 // This will break the binding.
        // -----
        const temp = cmpBinding.createObject(root, {
            "target": contentItem,
            "property": "width",
            "value": 100
        })
        // Now the width property is safely updated to 100 without breaking
        // any existing bindings.
        temp.destroy() // Don't forget to destroy it.
    }

    Component {
        id: cmpBinding

        Binding { }
    }
}
```

## B-4: KISS It

You are probably already familiar with the [KISS principle](https://en.wikipedia.org/wiki/KISS_principle).
QML supports optimization of binding expressions. Optimized bindings do not require
a JavaScript environment hence it runs faster. The basic requirement for optimization
of bindings is that the type  information of every symbol accessed must be known at
compile time.

So, avoid accessing `var` properties. You can see the full list of prerequisites
of optimized bindings [here](https://doc.qt.io/qt-5/qtquick-performance.html#bindings).

## B-5: Be Lazy

There may be cases where you don't need the binding immediately but when a certain
condition is met. By lazily creating a binding, you can avoid unnecessary executions.
To create a binding during runtime, you can use `Qt.binding()`.

```qml
Item {
    property int edgePosition: 0

    Component.onCompleted: {
        if (checkForSomeCondition() == true) {
            // bind to the result of the binding expression passed to Qt.binding()
            edgePosition = Qt.binding(function() { return x + width })
        }
    }
}
```

You can also use `Qt.callLater` to reduce the redundant calls to a function.

## B-6: Avoid Unnecessary Re-Evaluations

If you have a loop or process where you update the value of the property, you may
want to use a temporary local variable where you accumulate those changes and only
report the last value to the property. This way you can avoid triggering re-evaluation
of binding expressions during the intermediate stages of accumulation.

Here's a bad example straight from Qt documentation:

```qml
import QtQuick 2.3

Item {
    id: root

    property int accumulatedValue: 0

    width: 200
    height: 200
    Component.onCompleted: {
        const someData = [ 1, 2, 3, 4, 5, 20 ];
        for (let i = 0; i < someData.length; ++i) {
            accumulatedValue = accumulatedValue + someData[i];
        }
    }

    Text {
        anchors.fill: parent
        text: root.accumulatedValue.toString()
        onTextChanged: console.log("text binding re-evaluated")
    }
}
```

And here is the proper way of doing it:

```qml
import QtQuick 2.3

Item {
    id: root

    property int accumulatedValue: 0

    width: 200
    height: 200
    Component.onCompleted: {
        const someData = [ 1, 2, 3, 4, 5, 20 ];
        let temp = accumulatedValue;
        for (let i = 0; i < someData.length; ++i) {
            temp = temp + someData[i];
        }

        accumulatedValue = temp;
    }

    Text {
        anchors.fill: parent
        text: root.accumulatedValue.toString()
        onTextChanged: console.log("text binding re-evaluated")
    }
}
```

# C++ Integration

QML can be extended with C++ by exposing the `QObject` classes using the `Q_OBJECT`
macro or custom data types using `Q_GADGET` macro.
It always should be preferred to use C++ to add functionality to a QML application.
But it is important to know which is the best way to expose your C++ classes, and
it depends on your use case.

## CI-1: Avoid Context Properties

Context properties are registered using

```cpp
rootContext()->setContextProperty("someProperty", QVariant());
```

Context properties always takes in a `QVariant`, which means that whenever you access the property
it is re-evaluated because in between each access the property may be changed as
`setContextProperty()` can be used at any moment in time.

Context properties are expensive to access, and hard to reason with. When you are writing QML code,
you should strive to reduce the use of contextual variables (A variable that doesn't exist in the
immediate scope, but the one above it.) and global state. Each QML document should be able to run
with QML scene provided that the required properties are set.

See [QTBUG-73064](https://bugreports.qt.io/browse/QTBUG-73064).

## CI-2: Use Singleton for Common API Access

There are bound to be cases where you have to provide a single instance for a
functionality or common data access. In this situation, resort to using a singleton
as it will have a better performance and be easier to read. Singletons are also
a good option to expose enums to QML.

```cpp
class MySingletonClass : public QObject
{
public:
    static QObject *singletonProvider(QQmlEngine *qmlEngine, QJSEngine *jsEngine)
    {
        if (m_Instance == nullptr) {
            m_Instance = new MySingletonClass(qmlEngine);
        }

        Q_UNUSED(jsEngine);
        return m_Instance;
    }
};

// In main.cpp
qmlRegisterSingletonType<SingletonTest>("MyNameSpace", 1, 0, "MySingletonClass",
                                        MySingletonClass::singletonProvider);
```

You should strive to not use singletons for shared data access. Reusable components are especially
a bad place to access singletons. Ideally, all QML documents should rely on the customization
through properties to change its content.

Let's imagine a scenario where we are creating a paint app where we can change the currently
selected color on the palette. We only have one instance of the palette, and the data from this is
accessed throughout our C++ code. So we decided that it makes sense to expose it as a singleton to
QML side.

```qml
// ColorViewer.qml
Row {
    id: root

    Rectangle {
        color: Palette.selectedColor
    }

    Text {
        text: Palette.selectedColorName
    }
}
```

With this code, we bind our component to `Palette` singleton. Who ever wants to use our `ColorViewer`
they won't be able to change it so they can show some other selected color.

```qml
// ColorViewer_2.qml
Row {
    id: root

    property alias selectedColor: colorIndicator.color
    property alias selectedColorName: colorLabel.color

    Rectangle {
        id: colorIndicator
        color: Palette.selectedColor
    }

    Text {
        id: colorLabel
        text: Palette.selectedColorName
    }
}
```

This would allow the users of this component to set the color and the name from outside, but we
still have a dependency on the singleton.

```qml
// ColorViewer_3.qml
Row {
    id: root

    property alias selectedColor: colorIndicator.color
    property alias selectedColorName: colorLabel.color

    Rectangle {
        id: colorIndicator
    }

    Text {
        id: colorLabel
    }
}
```

This version allows you to de-couple from the singleton, enable it to be resuable in any context
that wants to show a selected color, and you could easily run this through `qmlscene` and inspect
its behavior.

## CI-3: Prefer Instantiated Types Over Singletons For Data

Instantiated types are exposed to QML using:

```cpp
// In main.cpp
qmlRegisterType<ColorModel>("MyNameSpace", 1, 0, "ColorModel");
```

Instantiated types have the benefit of having everything available to you to understand and digest
in the same document. They are easier to change at run-time without creating side effects, and easy
to reason with because when looking at a document, you don't need to worry about any global state
but the state of the type that you are dealing with at hand.

```qml
// ColorsWindow.qml
Window {
    id: root

    Column {
        Repeater {
            model: Palette.selectedColors
            delegate: ColorViewer {
                required property color color
                required property string colorName

                selectedColor: color
                selectedColorName: colorName
            }
        }
    }
}
```

The code above is a perfectly valid QML code. We'll get our model from the singleton, and display it
with the reusable component we created in CI-2. However, there's still a problem here. `ColorsWindow`
is now bound to the model from `Palette` singleton. And If I wanted to have the user select two
different sets of colors, I would need to create another file with the same contents and use that.
Now we have 2 components doing basically the same thing. And those two components need to be
maintained.

This also makes it hard to prototype. If I wanted to see two different versions of this window with
different colors at the same time, I can't do it because I'm using a singleton. Or, If I wanted to
pop up a new window that shows the users the variants of a color set, I can't do it because the data
is bound to the singleton.

A better approach here is to either use an instantiated type or expect the model as a property.

```qml
// ColorsWindow.qml
Window {
    id: root

    property PaletteColorsModel model

    Column {
        Repeater {
            model: root.model
            // Alternatively
            model: PaletteColorsModel { }
            delegate: ColorViewer {
                required property color color
                required property string colorName

                selectedColor: color
                selectedColorName: colorName
            }
        }
    }
}
```

Now, I can have the same window up at the same time with different color sets because they are not
bound to a singleton. During prototyping, I can provide a dummy data easily by adding
`PaletteColorElement` types to the model, or by requesting test dataset with something like:

```qml
PaletteColorsModel {
    testData: "prototype_1"
}
```

This test data could be auto-generated, or it could be provided by a JSON file. The beauty is that
I'm no longer bound to a singleton, that I have the freedom to instantiate as many of these windows
as I want.

There may be cases where you actually truly want the data to be the same every where. In these
cases, you should still provide an instantiated type instead of a singleton. You can still access
the same resource in the C++ implementation of your model and provide that to QML. And you would
still retain the freedom of making your data easily pluggable in different context and it would
increase the re-usability of your code.

```cpp
class PaletteColorsModel
{
    explicit PaletteColorsModel(QObject* parent = nullptr)
    {
        initializeModel(MyColorPaletteSingleton::instance().selectedColors());
    }
};
```

## CI-4: Watch Out for Object Ownership Rules

When you are exposing data to QML from C++, you are likely to pass around custom
data types as well. It is important to realize the implications of ownership when
you are passing data to QML. Otherwise you might end up scratching your head trying
to figure out why your app crashes.

If you are exposing custom data type, prefer to set the parent of that data to the
C++ class that transmits it to QML. This way, when the C++ class gets destroyed
the custom data type also gets destroyed and you won't have to worry about releasing
memory manually.

There might also be cases where you expose data from a singleton class without a
parent and the data gets destroyed because QML object that receives it will take
ownership and destroy it. And you will end up accessing data that doesn't exist.
Ownership is **not** transferred as the result of a property access. For data
ownership rules see [here](https://doc.qt.io/qt-5/qtqml-cppintegration-data.html#data-ownership).

To learn more about the real life implications of this read [this blog post](https://www.embeddeduse.com/2018/04/02/qml-engine-deletes-c-objects-still-in-use/).

# Performance and Memory

Most applications are not likely to have memory limitations. But in case you are
working on a memory limited hardware or you just really care about memory allocations,
follow these steps to reduce your memory usage.

## PM-1: Reduce the Number of Implicit Types

If a type defines custom properties, that type becomes an implicit type to the JS
engine and additional type information has to be stored.

```qml
Rectangle { } // Explicit type because it doesn't contain any custom properties

Rectangle {
    // The deceleration of this property makes this Rectangle an implicit type.
    property int meaningOfLife: 42
}
```

You should follow the advice from the [official documentation](http://doc.qt.io/qt-5/qtquick-performance.html#avoid-defining-multiple-identical-implicit-types)
and split the type into its own component If it's used in more than one place.
But sometimes, that might not make sense for your case. If you are using a lot of
custom properties in your QML file, consider wrapping the custom properties of
types in a `QtObject`. Obviously, JS engine will still need to allocate memory
for those types, but you already gain the memory efficiency by avoiding the
implicit types. Additionally, wrapping the properties in a `QtObject` uses less
memory than scattering those properties to different types.

Consider the following example:

```qml
Window {
    Rectangle { id: r1 } // Explicit type. Memory 64b, 1 allocation.

    // Implicit type. Memory 128b, 3 allocations.
    Rectangle { id: r2; property string nameTwo: "" }

    QtObject { // Implicit type. Memory 128b, 3 allocations.
        id: privates
        property string name: ""
    }
}
```

In this example, the introduction of a custom property to added additional 64b
of memory and 2 more allocations. Along with `privates`, memory usage adds up to
256b. The total memory usage is 320b.

You can use the QML profiler to see the allocations and memory usage for each
type. If we change that example to the following, you'll see that both memory
usage and number of allocations are reduced.

```qml
Window {
    Rectangle { id: r1 } // Explicit type. Memory 64b, 1 allocation.

    Rectangle { id: r2 } // Explicit type. Memory 64b, 1 allocation.

    QtObject { // Implicit type. Memory 160b, 4 allocations.
        id: privates

        property string name: ""
        property string nameTwo: ""
    }
}
```

In the second example, total memory usage is 288b. This is really a minute
difference in this context, but as the number of components increase in a
project with memory constrained hardware, it can start to make a difference.

# Signal Handling

Signals are a very powerful mechanism in Qt/QML. And the fact that you can
connect to signals from C++ makes it even better. But in some situations, If you
don't handle them correctly you might end up scratching your head.

## SH-1: Try to Avoid Using connect Function in Models

You can have signals in the QML side, and the C++ side. Here's an example for
both cases.

QML Example.

```qml
// MyButton.qml
import QtQuick.Controls 2.3

Button {
    id: root

    signal rightClicked()
}
```

C++ Example:

```cpp
class MyButton
{
    Q_OBJECT

signals:
    void rightClicked();
};
```

The way you connect to signals is using the syntax

```qml
item.somethingChanged.connect(function() {})
```

When this method is used, you create a function that is connected to the
`somethingChanged` signal.

Consider the following example:

```qml
// MyItem.qml
Item {
    id: root

    property QtObject customObject

    objectName: "my_item_is_alive"
    onCustomObjectChanged: {
        customObject.somethingChanged.connect(() => {
            console.log(root.objectName)
        })
    }
}
```

This is a perfectly legal code. And it would most likely work in most scenarios.
But, if the life time of the `customObject` is not managed in `MyItem`, meaning
if the `customObject` can keep on living when the `MyItem` instance is destroyed,
you run into problems.

The connection is created in the context of `MyItem`, and the function naturally
has access to its enclosing context. So, as long as we have the instance of
`MyItem`, whenever `somethingChanged` is emitted we'd get a log saying
`my_item_is_alive`.

Here's a quote directly from [Qt documentation](https://doc.qt.io/qt-5/qml-qtquick-listview.html):

> Delegates are instantiated as needed and may be destroyed at any time. They
> are parented to `ListView`'s `contentItem`, not to the view itself. State
> should never be stored in a delegate.

So you might be making use of an external object to store state. But what If
`MyItem` is used in a `ListView`, and it went out of view and it was destroyed
by `ListView`?

Let's examine what happens with a more concrete example.

```qml
ApplicationWindow {
    id: root

    property list<QtObject> myObjects: [
        QtObject {
            signal somethingHappened()
        },
        QtObject {
            signal somethingHappened()
        },
        QtObject {
            signal somethingHappened()
        },
        QtObject {
            signal somethingHappened()
        },
        QtObject {
            signal somethingHappened()
        },
        QtObject {
            signal somethingHappened()
        },
        QtObject {
            signal somethingHappened()
        },
        QtObject {
            signal somethingHappened()
        }
    ]

    width: 640
    height: 480

    ListView {
        anchors {
            top: parent.top
            left: parent.left
            right: parent.right
            bottom: btn.top
        }
        // Low enough we can resize the window to destroy buttons.
        cacheBuffer: 1
        model: root.myObjects.length
        delegate: Button {
            id: self

            readonly property string name: "Button #" + index

            text: "Button " + index
            onClicked: {
                root.myObjects[index].somethingHappened()
            }
            Component.onCompleted: {
                root.myObjects[index].somethingHappened.connect(() => {
                    // When the button is destroyed, this will cause the following
                    // error: TypeError: Type error
                    console.log(self.name)
                })
            }
            Component.onDestruction: {
                console.log("Destroyed #", index)
            }
        }
    }

    Button {
        id: btn
        anchors {
            bottom: parent.bottom
            horizontalCenter: parent.horizontalCenter
        }
        text: "Emit Last Signal"
        onClicked: {
            root.myObjects[root.myObjects.length - 1].somethingHappened()
        }
    }
}
```

In this example, once one of the buttons is destroyed we still have the object
instance. And then object instance still contains the connection we made in
`Component.onCompleted`. So, when we click on `btn`, we get an error:
`TypeError: Type error`. But once we expand the window so that the button is
created again, we don't get that error. That is, we don't get that error for the
newly created button. But the previous connection still exists and still causes
error. But now that a new one is created, we end up with two connections on the
same object.

This is obviously not ideal and should be avoided. But how do you do it?

The simplest and most elegant solution (That I have found) is to simply use a
`Connections` object and handle the signal there. So, If we change the code to
this:

```qml
delegate: Button {
    id: self

    readonly property string name: "Button #" + index

    text: "Button " + index
    onClicked: {
        root.myObjects[index].somethingHappened()
    }

    Connections {
        target: root.myObjects[index]
        onSomethingHappened: {
            console.log(self.name)
        }
    }
}
```

Now, whenever the delegate is destroyed so is the connection. This method can
be used even for multiple objects. You can simply put the `Connections` in a
`Component` and use `createObject` to instantiate it for a specific object.

```qml
Item {
    id: root
    onObjectAdded: {
        cmp.createObject(root, {"target": newObject})
    }

    Component {
        id: cmp

        Connections {
            target: root.myObjects[index]
            onSomethingHappened: {
                console.log(self.name)
            }
        }
    }
}
```

## SH-2: When to use Functions and Signals

When coming from imperative programming, it might be very tempting to use signals
very similar to functions. Resist this temptation. Especially when communicating
between the C++ layer of your application, misusing signals can be very confusing
down the line.

Let's first clearly define what a signal should be doing. Here's how
[Qt](https://doc.qt.io/qt-5/signalsandslots.html#signals) defines it.

> Signals are emitted by an object when its internal state has changed in some
> way that might be interesting to the object's client or owner. 

This means that whatever happens in the signal handler is a reaction to an
internal state change of an object. The signal handler should not be changing
something else in the same object.

See the following example. We have a `ColorPicker` component that we want to use
to show the user a message when the color is picked. As far as component design
goes, the fact that the customer sees a message is not `ColorPicker`'s job.
Its job is to present a dialog and change the color it represents.

```qml
// ColorPicker.qml
Rectangle {
    id: root

    signal colorPicked()

    ColorDialog {
        onColorChanged: {
            root.color = color
            root.colorPicked()
        }
    }
}

// main.qml
Window {
    ColorPicker {
        onColorPicked: {
            label.text = "Color Changed"
        }
    }

    Label {
        id: label
    }
}
```

The above example is pretty straightforward, the signal handler only reacts to
a change and does something with that information after which the `ColorPicker`
object is not affected.

```qml
// ColorPicker.qml
Rectangle {
    id: root

    signal colorPicked(color pickedColor)

    ColorDialog {
        onColorChanged: {
            root.colorPicked(color)
        }
    }
}

// main.qml
Window {
    ColorPicker {
        onColorPicked: {
            color = pickedColor
            label.text = "Color Changed"
        }
    }

    Label {
        id: label
    }
}
```

In this example, the signal handler not only reacts to an internal state but it
also changes it. This is a very simple example, and it'll be easy to spot an
error. However complex your application is, you will always benefit from
making the distinction clear. Otherwise what you think to be a function at first
glance might end up being a signal and it loses its semantics of an internal
state change.

Here's a general principle to follow:

1. When communicating up, use signals.
2. When communicating down, use functions.

### Communicating with C++ Using Signals

When you have a model that you use in the QML side, it's very possible that you
are going to run into cases where something that happens in the QML side needs
to trigger an action in the C++ side.

In these cases, prefer not to invoke any C++ signals from QML side. Instead,
use a function call or better a property assignment. The C++ object then should
make the decision whether to fire a signal or not.

If you are using a C++ type instantiated in QML, the same rules apply. You should
not be emitting signals from QML side.

# JavaScript

It is the prevalent advice that you should avoid using JavaScript as much as possible
in your QML code and have the C++ side handle all the logic. This is a sound advice
and should be followed, but there are cases where you can't avoid having JavaScript
code for your UI. In those cases, follow these guidelines to ensure a good use of
JavaScript in QML.

## JS-1: Use Arrow Functions

Arrow functions were introduced in ES6. Its syntax is pretty close to C++ lambdas
and they have a pretty neat feature that makes them most comfortable to use
when you are using the `connect()` function to create a binding. If there's no
block within the arrow function, it has an implicit return statement.

Let's compare the arrow function version with the old way.

```qml
Item {
    property int value: -1

    Component.onCompelted: {
        // Arrow function
        root.value = Qt.binding(() => root.someOtherValue)
        // The old way.
        root.value = Qt.binding(function() { return root.someOtherValue })
    }
}
```

The arrow function version is easier on the eyes and cleaner to write.
For more information about arrow functions, head over to the [MDN Blog](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

## JS-2: Use the Modern Way of Declaring Variables

With ES6, there are 3 ways of delcaring a variable: `var`, `let`, and `const`.

You should leverage `let` and `const` in your codebase and avoid using `var`.
`let` and `const` enables a scope based naming wheras `var` only knows about one
scope.

```qml
Item {
    onClicked: {
        const value = 32;
        let valueTwo = 42;
        {
            // Valid assignment since we are in a different scope.
            const value = 32;
            let valueTwo = 42;
        }
    }
}
```

Much like in C++, prefer using `const` If you don't want the variable to be assigned.
But keep in mind that `const` variables in JavaScript are not immutable. It just
means they can't be reassigned, but their contents can be changed.

```javascript
const value = 32;
value = 42; // ERROR!

const obj = {value: 32};
obj.value = 42; // Valid.
```

See the MDN posts on [const](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)
and [let](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let)

# States and Transitions

States and transitions are a powerful way to create dynamic UIs. Here are some things to keep in
mind when you are using them in your projects.

## ST-1: Don't Define Top Level States

Defining states at the top-level of a reusable component can cause breakages if the user of your
components also define their own states for their specific use case. 

```qml
// MyButton.qml
Rectangle {
    id: root

    property alias text: lb.text
    property alias hovered: ma.containsMouse

    color: "red"
    states: [
        State {
            when: ma.containsMouse

            PropertyChanges {
                target: root
                color: "yellow"
            }
        }
    ]

    MouseArea {
        id: ma
        anchors.fill: parent
        hoverEnabled: true
    }

    Label {
        id: lb
        anchors.centerIn: parent
    }
}

// MyItem.qml
Item {
    MyButton {
        id: btn
        text: "Not Hovering"
        // The states of the original component are not actually overwritten.
        // The new state is added to the existing states.
        states: [
            State {
                when: btn.hovered

                PropertyChanges {
                    target: btn
                    text: "Hovering"
                }
            }
        ]
    }
}
```
When you assign a new value to `states` or any other `QQmlListProperty`, the new value does not
overwrite the existing one but adds to it. In the example above, the new state is added to the
existing list of states that we already have in `MyButton.qml`. Since we can only have one active
state in an item, our hover state will be messed up.

In order to avoid this problem, create your top-level state in a separate item or use a
`StateGroup`.

```qml
Rectangle {
    id: root

    property alias text: lb.text
    property alias hovered: ma.containsMouse

    color: "red"

    MouseArea {
        id: ma
        anchors.fill: parent
        hoverEnabled: true
    }

    Label {
        id: lb
        anchors.centerIn: parent
    }

    // A State group or
    StateGroup {
        states: [
            State {
                when: ma.containsMouse

                PropertyChanges {
                    target: root
                    color: "yellow"
                }
            }
        ]
    }

    // another item
    Item {
        states: [
            State {
                when: ma.containsMouse

                PropertyChanges {
                    target: root
                    color: "yellow"
                }
            }
        ]
    }
}
```

With this change, the button will both change its color and text when the mouse is hovered above it.


# Visual Items

Visual items are at the core of QML, anything that you see in the window (or don't see because of
transparency) are visual items. Having a good understanding of the visual items, their relationship
to each other, sizing, and positioning will help you create a more robust UI for your application.

## VI-1: Distinguish Between Different Types of Sizes

When thinking about geometry, we think in terms of `x`, `y`, `width` and `height`. This defines
where our items shows up in the scene and how big it is. `x` and `y` are pretty straightforward but
we can't really say the same about the size information in QML.

There's 2 different types of size information that you get from various visual items:

1. Explicit size: `width`, `height`
2. Implicit size: `implicitWidth`, `implicitHeight`

A good understanding of these different types is important to building a reusable library of
components.

### Explicit Size

It's in the name. This is the size that you explicitly assign to an `Item`. By default, `Item`s do
not have an explicit size and its size will always be `Qt.size(0, 0)`.

```qml
// No explicit size is set. You won't see this in your window.
Rectangle {
    color: "red"
}

// Explicit size is set. You'll see a yellow rectangle.
Rectangle {
    width: 100
    height: 100
    color: "yellow"
}
```

### Implicit Size

Implicit size refers to the size that an `Item` occupies by default to display itself properly.
This size is not set automatically for any `Item`. You, as a component designer, need to make a
decision about this size and set it to your component.

The other thing to note is that [Qt internally
knows](https://github.com/qt/qtdeclarative/blob/dev/src/quick/items/qquickitem.h#L418) if it has an
explicit size or not. So, when an explicit size is not set, it will use the implicit size.

```qml
// Even though there's no explicit size, it will have a size of Qt.size(100, 100)
Rectangle {
    implicitWidth: 100
    implicitHeight: 100
    color: "red"
}
```

-----

Whenever you are building a reusable component, never set an explicit size within the component but
instead choose to provide a sensible implicit size. This way, the user of your components can freely
manipulate its size and when they need to return to a default size, they can always default to the
implicit size so they don't have to store a different default size for the component. This feature
is also very useful if you want to implement a resize-to-fit feature.

When a user is using your component, they may not bother to set a size for it.

```qml
CheckBox {
    text: "Check Me Out"
}
```

In the example above, the check box would only be visible If there was a sensible implicit size for
it. This implicit size needs to take into account its visual components (the box, the label etc.) so
that we can see the component properly. If this is not provided, it's difficult for the user of your
component to set a proper size for it.
