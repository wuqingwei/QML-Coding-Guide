# Introduction

这里的内容主要基于我在一个大型项目中与不同的开发人员和设计师团队一起使用 QML 的经验。当我对某些事情的看法得到实际生活经验的验证时，我会更新本文档。


您可能不同意这里提出的某些观点，在这种情况下，请创建一个问题进行讨论，并进行相应的更新。我会不断更新本指南，大家的贡献对本文档至关重要，因为它需要反映更多经过尝试和验证的想法，这样才能具有普遍意义。我很可能没有很好地解释某个概念。
如果有任何改进意见，我将不胜感激。

请不要犹豫，提出问题并提交 PR。即使是最微小的 贡献都很重要。

# Table of Contents

- [Code Style](#code-style)
  - [CS-1: Signal Handler Ordering](#cs-1-signal-handler-ordering)
  - [CS-2: Property Initialization Order](#cs-2-property-initialization-order)
  - [CS-3: Function Ordering](#cs-3-function-ordering)
  - [CS-4: Animations](#cs-4-animations)
  - [CS-5: Specifying IDs for Objects](#cs-5-specifying-ids-for-objects)
  - [CS-6: Property Assignments](#cs-6-property-assignments)
  - [CS-7: Import Statements](#cs-7-import-statements)
  - [Full Example](#full-example)
- [Bindings](#bindings)
  - [B-1: Prefer Bindings over Imperative Assignments](#b-1-prefer-bindings-over-imperative-assignments)
  - [B-2: Making `Connections`](#b-2-making-connections)
  - [B-3: Use `Binding` Object](#b-3-use-binding-object)
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
  - [VI-2: Be Careful with a Transparent `Rectangle`](#vi-2-be-careful-with-a-transparent-rectangle)

# Code Style

本节将详细介绍如何格式化属性、信号和函数的顺序、 和函数的顺序格式，以便于阅读和快速切换到相关代码块。

[QML object attributes](https://doc.qt.io/qt-5/qtqml-syntax-objectattributes.html)总是按以下顺序编排

- id
- Property declarations （属性声明）
- Signal declarations （信号声明）
- Property initializations （属性初始化）
- Attached properties and signal handlers （附加属性和信号处理器）
- States （状态）
- Transitions （特效）
- Signal handlers （信号处理函数）
- Child objects （子对象）
  + Visual Items （可见对象）
  + Qt provided non-visual items （qt提供的非可见对象）
  + Custom non-visual items （自定义的非可见对象）
- `QtObject` for encapsulating private members[1 ](https://bugreports.qt.io/browse/QTBUG-11984)（QtObject封装私有成员）
- JavaScript functions （js函数）

这样排序的主要目的是确保一个类型最内在的属性总是最显而易见的，以便让界面更容易被第一眼消化。
虽然可以说 JavaScript 函数也是界面的一部分，但理想的情况是完全没有函数

## CS-1: Signal Handler Ordering

在处理连接到Item的信号时，请确保始终将Component.onCompleted 放在最后一行。

```js
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

这是因为，从心理上讲，这样会使画面更美观，因为 Component.onCompleted 会在组件构建完成时触发。

如果一个项目中有多个信号处理器，那么行数最少的处理器可能会被放在最前面。行数最少的处理程序可以放在最前面。随着执行行数的增加，处理程序 也会向下移动。唯一的例外是 Component.onCompleted 信号。总是放在最下面。

```js
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

## CS-2: Property Initialization Order

第一个属性赋值必须始终是组件的 id。如果 要声明组件的自定义属性，声明总是在第一个属性赋值之上。

```js
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

属性分配也有一些预定义的顺序。顺序如下

- id
- x
- y
- width
- height
- anchors

这样做的目的是将最明显、最有定义性的属性放在最上面，以便于访问和查看。便于访问和查看。例如，对于Image图像，您可以决定将 将 sourceSize 放在anchors之上。

------

如果在信号处理程序中也有属性赋值，请确保 始终将属性赋值放在信号处理程序之上。

```js
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

如果属性赋值与信号处理程序混在一起，通常更难查看。这就是为什么我们要把赋值放在信号 处理程序上。

### CS-3: Function Ordering

虽然 QML 中没有 private 和 public 函数，但您可以通过封装只应在 QtObject 内部使用的属性和函数来提供类似的机制。

公共函数的实现总是放在文件的最底部。尽管我们 优先将其他类型的公共声明放在文件顶部，但我还是鼓励您将公共函数放在底部。因为如果一个函数的行数变多，会大大降低 QML 文档的可读性、 会大大降低 QML 文档的可读性。理想情况下，您不应该有任何 函数，而尽量依赖组件的声明属性。尽可能依赖组件的声明属性。

```js
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

在使用任何动画子类时，尤其是嵌套的动画子类，如 SequentialAnimation 等嵌套类时，请尽量减少一行中的属性数量。
如果在同一行中有超过 2-3 个赋值，就会变得难以推理。之后就很难推理了。或者，您可以将单行赋值保持在您为项目设置的任何行长 约定。

由于动画很难在脑海中想象出来，因此尽可能简单的动画会让您受益匪浅。让动画尽可能简单。

```js
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

### CS-5: Specifying IDs for Objects

如果不需要访问某个对象的功能，则应避免设置 id 属性。这样就不容易出现 id 重复的问题。此外，为对象设置 id 会给我们带来额外的认知压力，因为这意味着我们需要照顾额外的关系。

如果想用描述符标记类型，但又不打算引用该类型，可以使用 objectName 代替，或者使用普通的注释。

确保文件中最顶部的组件始终以 root 作为其 id。
Qt 将在 QML 3 中淘汰非限定名称查找，因此最好现在就开始给组件 ID 并使用限定查找。所以最好现在就开始为组件赋予 ID 并使用限定查找。

详见 [QTBUG-71578](https://bugreports.qt.io/browse/QTBUG-71578)  和[QTBUG-76016](https://bugreports.qt.io/browse/QTBUG-76016) 

### CS-6: Property Assignments

在分配分组属性时，如果只更改一个属性，则应使用点符号。否则，请务必使用分
组符号。

```js
Image {
    anchors.left: parent.left // Dot notation
    sourceSize { // Group notation
        width: 32
        height: 32
    }
}
```

当您在同一文件的不同地方为 Loader 的 sourceComponent 分配组件时，请考虑使用相同的实现。例如，在 有两个相同组件的实例。如果这两个 这些 SomeSpecialComponent 都是相同的，那么最好的办法就是 将 SomeSpecialComponent 封装在一个 Component 中。

```js
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

这样就可以确保每当您对 specialComponent 进行更改时，它都会在所有加载器中生效。而在糟糕的示例中，您必须复制 相同的更改。

在不使用 Loader 的类似情况下，您可以使用内联components。

```js
component SomeSpecialComponent: Rectangle {

}
```

### CS-7: Import Statements

一般来说，在进行繁重的工作时，C++ 比 JavaScript 更受青睐。如果在某些情况下需要单独的 JavaScript 文件，请记住以下几点。

如果要导入 JavaScript 文件，请确保在 QML 文件和 JavaScript 文件中不包含相同的 模块。JavaScript 文件共享 从 QML 文件导入，因此您可以利用这一点。如果 JavaScript 文件是作为一个库，这一点就不适用。

如果您没有在 QML 文件中使用导入的模块，请考虑将 导入语句到 JavaScript 文件中。但请注意，一旦您在 JavaScript 文件中导入了 在 JavaScript 文件中，导入将不再共享。有关完整的 规则，请参阅[此处](https://doc.qt.io/qt-5/qtqml-javascript-imports.html#imports-within-javascript-resources)。 

Qt.include()` 已被[弃用](https://doc.qt.io/qt-6/qml-qtqml-qt-obsolete.html#include-method))，不应使用

一般来说，应避免使用未使用的导入语句。

#### Import Order

导入其他模块时，请使用以下顺序；

- Qt 模块
- 第三方m欧克
- 本地C++ 模块
- QML 目录

### Full Example

```js
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

如果使用得当，绑定是一种强大的工具。 每当绑定所依赖的属性发生变化时，绑定就会被评估，这可能会导致性能不佳或意外行为。或意外行为。即使是简单的绑定，其结果也可能是昂贵的。例如，绑定会导致一个项目的位置发生变化 而依赖于该项目位置或锚定到该项目上的其他项目也会更新其位置。

因此，在使用绑定时，请考虑以下规则。

## B-1: Prefer Bindings over Imperative Assignments

优先选择绑定而非强制指定。

参见相关章节 [Qt Documentation](https://doc.qt.io/qt-5/qtquick-bestpractices.html#prefer-declarative-bindings-over-imperative-assignments).

官方文档解释得很清楚，但同样重要的是，要了解绑定在性能上的复杂性，并知道瓶颈在哪里。
瓶颈所在。

如果你怀疑你遇到的性能问题与过多的绑定评估有关，那么请使用 QML 剖析器来确认你的怀疑，然后选择使用必要选项。

参考 [official documentation](https://doc.qt.io/qtcreator/creator-qml-performance-monitor.html)了解如何使用 QML 剖析器。

## B-2: Making `Connections`

连接对象用于处理来自 QML 中任意 QObject 派生类的信号。使用连接时需要注意的一点是 连接的 target 属性的默认值是它的父对象，如果没有显式地 设置为其他值。如果您在动态创建 QML 对象后设置目标 您可能希望将目标设置为空，否则您可能会 收到不应该被处理的信号。

还要注意的是，使用 Connections 对象会带来轻微的性能/内存损耗，因为它需要进行另一次分配。如果您担心这个问题，可以使用 QtObject.connect方法，但要[注意]()这种方法的 此解决方案的隐患。 

```js
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

绑定 的 when 属性可用于根据条件启用或禁用绑定表达式。如果您使用的绑定很复杂，不需要在每次属性变化时都执行绑定，那么这是减少绑定执行次数的好主意。

使用上面的同一个示例，我们可以使用绑定对象将其重写如下。

```js
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

再说一遍，这只是一个非常简单的例子，目的是为了说明问题。在现实生活中 情况下，在这种情况下使用绑定对象不会带来更多好处。除非绑定表达式的代价很高（例如，改变项目的 锚点，从而引起连锁反应，导致其他项目重新定位。

## B-4: KISS It

已删除。不再适用。保留记录是为了不乱用项目编号。

### 删除理由

自从我第一次编写本指南以来，QML 引擎发生了很大变化。虽然保持绑定简单仍然是个好主意（即不要在绑定中调用任何昂贵的函数），但如果说 会有某些优化。关于避免使用 var 属性的建议仍然适用，您应该尽量使用最精确的类型。

### 以前的内容

~您可能已经熟悉 [KISS 原则](https://en.wikipedia.org/wiki/KISS_principle) 
QML 支持优化绑定表达式。优化后的绑定不需要 JavaScript 环境，因此运行速度更快。优化绑定的基本要求是，编译时必须知道访问的每个符号的类型信息。

~因此，**应避免访问 var 属性**。你可以在[Performance Considerations And Suggestions](https://doc.qt.io/qt-5/qtquick-performance.html#bindings)查看优化绑定的全部前提条件。

## B-5: Be Lazy

已删除。不再适用。保留记录是为了不乱用项目编号。

### 删除理由

在 QML 中，声明式比命令式更好。这促进了命令式方法的发展，并没有 提供很大价值。如果您需要禁用或启用绑定参考
[Binding](#b-1-prefer-bindings-over-imperative-assignments) 或使用布尔标志 来启用绑定，例如 `visible: privates.bindingEnabled ? root.count > 0 : false`.

### 以前的内容

有些情况下，您可能不需要立即绑定，而是在满足某个 条件满足时才需要绑定。通过延迟地创建绑定，可以避免不必要的执行。要在运行时创建绑定，可以使用 `Qt.binding()`.~

```js
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

~您还可以使用 Qt.callLater 来减少对函数的冗余调用。

## B-6: Avoid Unnecessary Re-Evaluations

避免不必要的重新赋值？

如果您有一个循环或流程需要更新属性值，您可能需要使用临时局部变量来累积这些变化。然后只
报告属性的最后一个值。这样就可以避免在累加的中间阶段触发对绑定表达式的重新评估。

下面是一个直接来自 Qt 文档的糟糕示例：

```js
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

下面是合适的做法

```js
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

还要注意的是，列表类型没有与添加/移动/删除元素相关的更改信号。如果要使用列表类型存储顺序数据，请确保使用该属性的地方不会执行昂贵的操作（例如填充视图）。

# C++ Integration

QML 可用 C++ 扩展，方法是使用 Q_OBJECT 宏公开 QObject 类，或使用 Q_GADGET 宏公开自定义数据类型。使用 C++ 为 QML 应用程序添加功能总是首选。
但重要的是要知道哪种是公开 C++ 类的最佳方式，这取决于您的使用情况。

## CI-1: Avoid Context Properties 避免使用上下文属性

避免使用上下文属性

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

上下文属性的访问成本很高，而且难以推理。当你编写 QML 代码时，应努力减少使用上下文变量（不存在于直接作用域中的变量，而存在于其上的变量）和全局状态。只要设置了所需的属性，每个 QML 文档都应能与 QML 场景一起运行。

见 [QTBUG-73064](https://bugreports.qt.io/browse/QTBUG-73064).

## CI-2: Use Singleton for Common API Access

使用单例来访问通用应用程序接口

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

你应该尽量不使用单例来访问共享数据。可重用组件尤其不是访问单例的好地方。理想情况下，所有QML文档都应该依赖于通过属性进行的定制来更改其内容。

让我们想象一个场景，我们正在创建一个油漆应用程序，我们可以改变调色板上当前选定的颜色。我们只有一个调色板实例，并且在整个C++代码中都可以访问该实例中的数据。因此，我们决定将其作为单例暴露给QML方是有意义的。

应尽量避免使用单例来访问共享数据。可重用组件尤其 是访问单例的坏地方。理想情况下，所有 QML 文档都应依靠自定义 属性来改变其内容。

让我们设想一个场景，我们正在创建一个油漆应用程序，我们可以改变调色板上当前选择的颜色。选择的颜色。我们只有一个调色板实例，在整个 C++ 代码中都可以访问其中的数据。
我们的整个 C++ 代码都要访问其中的数据。因此，我们决定将它作为一个单例暴露给 QML 端。

```js
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

通过这段代码，我们将组件绑定到了 `Palette` 单例。如果有人想使用我们的 `ColorViewer` ，他们就无法更改它，从而无法显示其他选定的颜色。

```js
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

这将允许该组件的用户从外部设置颜色和名称，但我们仍然依赖于单例。

```js
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

该版本允许您从单例解耦，使其可在任何想要显示所选颜色的上下文中重新使用，而且您可以轻松地通过 `qmlscene` 运行该版本并检查其行为。

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

```js
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

上面的代码是完全有效的 QML 代码。我们将从单例中获取模型，并用我们在 CI-2 中创建的可重用组件来显示它。然而，这里仍有一个问题。“ColorsWindow “现在绑定到了 ”Palette "单例中的模型。如果我想让用户选择两组不同的颜色，我就需要创建另一个内容相同的文件并使用它。现在我们有两个组件在做基本相同的事情，而这两个组件需要维护。

这也使得它很难进行原型设计。如果我想同时看到这个窗口的两个不同颜色版本，我做不到，因为我使用的是单例。或者，如果我想要弹出一个新窗口，向用户显示颜色集的变体，我也做不到，因为数据绑定在单例中。

更好的办法是使用实例化类型或将模型作为属性。

```js
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

现在，我可以让同一个窗口在同一时间显示不同的颜色集，因为它们没有绑定到一个单例。在原型开发过程中，我可以通过在模型中添加
’PaletteColorElement‘ 类型，或者通过请求测试数据集来提供虚拟数据：

```js
PaletteColorsModel {
    testData: "prototype_1"
}
```

测试数据可以自动生成，也可以由 JSON 文件提供。这样做的好处是，我不再受单例的约束，可以自由地实例化任意数量的窗口。

在某些情况下，您可能真的希望数据在任何地方都是一样的。在这种情况下，你仍然应该提供一个实例化类型，而不是一个单例。您仍然可以在模型的 C++ 实现中访问相同的资源，并将其提供给 QML。您仍然可以在不同的上下文中自由地使用您的数据，增加代码的可重用性。

```cpp
class PaletteColorsModel
{
    explicit PaletteColorsModel(QObject* parent = nullptr)
    {
        initializeModel(MyColorPaletteSingleton::instance().selectedColors());
    }
};
```

## CI-4: Watch Out for Object Ownership Rules 注意对象所有权规则

当您从 C++ 向 QML 公开数据时，很可能也会传递自定义的数据类型。否则，您最终可能会抓耳挠腮，想不出应用程序崩溃的原因。

如果您要公开自定义数据类型，最好将数据的父类设置为向 QML 传输数据的 C++ 类。这样，当 C++ 类被销毁时，自定义数据类型也会被销毁，您就不必担心手动释放内存了。

也可能出现这样的情况：你从一个没有父类的单例类暴露数据，而数据会被销毁，因为接收它的 QML 对象会取得所有权并销毁它。最终，你将访问不存在的数据。所有权不会因为属性访问而转移。有关数据
所有权规则，请参阅[这里](https://doc.qt.io/qt-5/qtqml-cppintegration-data.html#data-ownership).

了解更多，阅读这篇[文章](https://www.embeddeduse.com/2018/04/02/qml-engine-deletes-c-objects-still-in-use/).

# Performance and Memory

大多数应用程序都不会有内存限制。但如果你在内存有限的硬件上工作，或者你真的很在意内存分配、请按照以下步骤减少内存使用量。

## PM-1: Reduce the Number of Implicit Types

如果一个类型定义了自定义属性，该类型就会成为 JS 引擎的隐式类型，必须存储额外的类型信息。

```js
Rectangle { } // Explicit type because it doesn't contain any custom properties

Rectangle {
    // The deceleration of this property makes this Rectangle an implicit type.
    property int meaningOfLife: 42
}
```

应该参考官方文档的建议 [官方文档](http://doc.qt.io/qt-5/qtquick-performance.html#avoid-defining-multiple-identical-implicit-types)，如果该类型在多个地方使用，则将其拆分成独立的组件。
但有时，这对您的情况可能没有意义。如果您在 QML 文件中使用了大量自定义属性，可以考虑用 `QtObject`封装类型的自定义属性。显然，JS 引擎仍需要为这些类型分配内存，但由于避免了隐式类型，您已经获得了内存效率。此外，将属性封装在 `QtObject` 中比将这些属性分散到不同的类型中占用更少的内存。

示例：

```js
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

在本例中，自定义属性的引入增加了 64b 内存和 2 次分配。加上 “privates”，内存使用量增加到 256b。总内存使用量为 320b。

你可以使用 QML profiler 查看每种类型的分配和内存使用情况。如果我们把这个例子改成下面这个，你会发现内存使用量和分配次数都减少了。

```js
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

在第二个示例中，总内存使用量为 288b。在这种情况下，这确实只是微小的差别，但当项目中的组件数量增加，硬件内存受限时，差别就会开始显现。

# Signal Handling

信号是 Qt/QML 中非常强大的机制。事实上，您可以从 C++ 连接到信号，这让它变得更加出色。但在某些情况下，如果处理不当，您可能会抓耳挠腮。

## SH-1: Try to Avoid Using connect Function in Models

您可以在 QML 端和 C++ 端设置信号。下面是一个两种情况的例子。

QML 端.

```js
// MyButton.qml
import QtQuick.Controls 2.3

Button {
    id: root

    signal rightClicked()
}
```

C++ 端:

```cpp
class MyButton
{
    Q_OBJECT

signals:
    void rightClicked();
};
```

连接信号使用的语法

```js
item.somethingChanged.connect(function() {})
```

使用此方法时，您将创建一个与`somethingChanged` 信号相连的函数。

示例:

```js
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

这是完全合法的代码。而且在大多数情况下都能正常工作。但是，如果 `MyItem` 中没有管理 `customObject` 的生命周期，也就是说
如果当 `MyItem` 实例被销毁时，`customObject` 还能继续存在、
就会出现问题。

连接是在 `MyItem`的上下文中创建的，函数自然也就可以访问其外层上下文。因此，只要我们有
”MyItem“的实例，每当 ”somethingChanged "发出时，我们就会收到一条日志，上面写着
my_item_is_alive`。

下面内容来自Qt文档 [Qt documentation](https://doc.qt.io/qt-5/qml-qtquick-listview.html):

> Delegates are instantiated as needed and may be destroyed at any time. They
> are parented to `ListView`'s `contentItem`, not to the view itself. State
> should never be stored in a delegate.

> Delegates代理根据需要实例化，可随时销毁。它们的父节点是 `ListView` 的 `contentItem`，而不是view视图本身。绝不应将state状态存储在delegate委托中。

So you might be making use of an external object to store state. But what If
`MyItem` is used in a `ListView`, and it went out of view and it was destroyed
by `ListView`?

因此，您可能会使用外部对象来存储状态。但如果`MyItem`在`ListView`中使用，而它离开了视图并被`ListView`销毁了怎么办？

让我们用一个更具体的例子来看看会发生什么。

```js
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

在此示例中，一旦其中一个按钮被销毁，我们仍然拥有该对象的实例。然后，对象实例仍然包含我们在
`Component.onCompleted` 中建立的连接。因此，当我们点击 `btn` 时，就会出现错误：
`TypeError: Type error`。但当我们展开窗口，再次创建按钮时。但之前的连接仍然存在，并且仍然会导致
错误。但现在又创建了一个新的连接，我们就会在同一个对象上出现两个连接。

这种情况显然不理想，应该避免。但该怎么做呢？

最简单、最优雅的解决方案（我发现）是简单地使用一个连接 "对象并在其中处理信号。因此，如果我们将代码改为
这样：

```js
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

现在，只要委托被销毁，连接也会被销毁。这种方法甚至可以用于多个对象。您只需将 `Connections` 放在一个 `Component` 中，然后使用 `createObject` 为特定对象实例化即可。

```js
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

在命令式编程中，使用与函数非常相似的信号可能很有诱惑力。请抵制这种诱惑。尤其是在应用程序的 C++ 层之间进行通信时，错误使用信号可能会造成严重的混淆。

首先，让我们明确定义信号的作用。下面是[Qt](https://doc.qt.io/qt-5/signalsandslots.html#signals) 是如何定义的。

> Signals are emitted by an object when its internal state has changed in some
> way that might be interesting to the object's client or owner. 

> 当对象的内部状态发生某种变化，而对象的客户端或所有者可能对此感兴趣时，对象就会发出信号。



这意味着信号处理器中发生的任何事情都是对对象内部状态变化的反应。信号处理程序不应改变同一对象中的其他内容。

请看下面的示例。我们有一个 `ColorPicker` 组件，当颜色被选中时，我们想用它来向用户显示一条信息。就组件设计而言，客户看到一条消息并不是` ColorPicker` 的工作，它的工作是显示一个对话框并更改它所代表的颜色。

```js
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

上面的示例非常简单，信号处理程序只对变化做出反应，并对信息进行处理，之后 `ColorPicker` 对象就不受影响了。

```js
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



在这个示例中，信号处理器不仅对内部状态做出反应，而且还改变了内部状态。这是一个非常简单的示例，很容易发现错误。无论您的应用程序有多复杂，明确区分总是有益的。否则，你乍一看以为是函数的东西最终可能会变成信号，从而失去内部状态变化的语义。

Here's a general principle to follow:

1. When communicating up, use signals.

2. When communicating down, use functions.

    这里有一个总的原则可以遵循：

    1. 向上交流时，使用信号。  
    2. 向下通信时，使用函数。

### Communicating with C++ Using Signals

When you have a model that you use in the QML side, it's very possible that you
are going to run into cases where something that happens in the QML side needs
to trigger an action in the C++ side.

In these cases, prefer not to invoke any C++ signals from QML side. Instead,
use a function call or better a property assignment. The C++ object then should
make the decision whether to fire a signal or not.

If you are using a C++ type instantiated in QML, the same rules apply. You should
not be emitting signals from QML side.

当您在 QML 端使用模型时，很可能会遇到 QML 端发生的事情需要触发 C++ 端动作的情况。

在这种情况下，最好不要从 QML 端调用任何 C++ 信号。取而代之的是使用函数调用或更好的属性赋值。然后由 C++ 对象决定是否触发信号。

如果你使用在 QML 中实例化的 C++ 类型，同样的规则也适用。你不应该从 QML 端发射信号。

# JavaScript

流行的建议是，应尽可能避免在 QML 代码中使用 JavaScript，而让 C++ 处理所有逻辑。这是一个合理的建议，应予以遵循，但在某些情况下，您无法避免在用户界面中使用 JavaScript 代码。在这种情况下，请遵循以下指南，以确保在 QML 中很好地使用 JavaScript。

## JS-1: Use Arrow Functions

Arrow functions were introduced in ES6. Its syntax is pretty close to C++ lambdas
and they have a pretty neat feature that makes them most comfortable to use
when you are using the `connect()` function to create a binding. If there's no
block within the arrow function, it has an implicit return statement.

Let's compare the arrow function version with the old way.

ES6 引入了箭头函数。它的语法与 C++ lambdas 非常接近，而且有一个相当不错的特性，使其在使用 `connect()` 函数创建绑定时使用起来更加得心应手。如果箭头函数中没有代码块，它就会有一个隐式返回语句。让我们比较一下箭头函数版本和旧版本。

```js
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

箭头函数版本更容易看懂，书写也更简洁。 有关箭头函数的更多信息，请访问 [MDN 博客](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

## JS-2: Use the Modern Way of Declaring Variables

With ES6, there are 3 ways of delcaring a variable: `var`, `let`, and `const`.

You should leverage `let` and `const` in your codebase and avoid using `var`.
`let` and `const` enables a scope based naming wheras `var` only knows about one
scope.

在 ES6 中，有 3 种删除变量的方法：var、let 和 const。您应该在代码库中使用 let 和 const，避免使用 var。

`let` 和 `const` 实现了基于作用域的命名，而 `var` 只知道一个作用域。

```js
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

和 C++ 一样，如果你不希望变量被赋值，最好使用 const。 但请记住，JavaScript 中的 const 变量并不是不可变的。这只是意味着它们不能被重新赋值，但其内容可以更改。

```javascript
const value = 32;
value = 42; // ERROR!

const obj = {value: 32};
obj.value = 42; // Valid.
```

参考MDN文章 [const](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)和 [let](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let)

# States and Transitions

状态和转换是创建动态用户界面的一种强大方法。以下是在项目中使用它们时需要注意的一些事项。

## ST-1: Don't Define Top Level States

Defining states at the top-level of a reusable component can cause breakages if the user of your
components also define their own states for their specific use case. 

如果组件的用户也为他们的特定用例定义自己的状态，在可重用组件的顶层定义状态就会造成中断。

```js
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

```js
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

可视化项是 QML 的核心，您在窗口中看到的任何东西（或由于透明而看不到的东西）都是可视化项。充分了解可视化项、它们之间的关系、大小和位置，将有助于为您的应用程序创建更强大的用户界面。

## VI-1: Distinguish Between Different Types of Sizes

在考虑几何图形时，我们会考虑 x、y、宽度和高度。这定义了物品在场景中的位置和大小。x 和 y 非常简单，但 QML 中的尺寸信息却不是这样。

您可以从各种视觉项目中获得两种不同的尺寸信息：

1. Explicit size: `width`, `height`
2. Implicit size: `implicitWidth`, `implicitHeight`

充分了解这些不同类型对于建立一个可重复使用的组件库非常重要。

### Explicit Size

顾名思义。这是显式分配给项的大小。默认情况下，Items 没有显式大小，其大小始终为 Qt.size(0,0)。

```js
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

隐含尺寸指的是一个 `Item` 为正确显示自己而默认占用的尺寸。 这个尺寸不会自动为任何 `Item` 设置。作为组件设计者，您需要决定这个尺寸，并将其设置到您的组件中。

还需注意的是，Qt 内部 [Qt internally
knows](https://github.com/qt/qtdeclarative/blob/dev/src/quick/items/qquickitem.h#L418)知道是否有显式大小。因此，如果没有设置显式大小，它将使用隐式大小。

```js
// Even though there's no explicit size, it will have a size of Qt.size(100, 100)
Rectangle {
    implicitWidth: 100
    implicitHeight: 100
    color: "red"
}
```

-----

无论何时构建可重用组件，都不要在组件中设置显式尺寸，而是选择提供合理的隐式尺寸。这样，组件的用户就可以自由操作组件的大小，当他们需要返回默认大小时，就可以始终默认使用隐式大小，这样他们就不必为组件存储不同的默认大小。如果您想实现调整大小的功能，这一功能也非常有用。

当用户使用您的组件时，他们可能懒得为其设置尺寸

```js
CheckBox {
    text: "Check Me Out"
}
```

In the example above, the check box would only be visible If there was a sensible implicit size for
it. This implicit size needs to take into account its visual components (the box, the label etc.) so
that we can see the component properly. If this is not provided, it's difficult for the user of your
component to set a proper size for it.

在上面的示例中，复选框只有在有合理的隐含尺寸的情况下才会可见。这个隐含尺寸需要考虑到它的视觉组件（方框、标签等），这样我们才能正确地看到组件。如果不提供隐含尺寸，组件的用户就很难为其设置合适的尺寸。

## VI-2: Be Careful with a Transparent `Rectangle`

`Rectangle` should never be used with a transparent color except when you need to draw a border.
This is especially true if you are using a `Rectangle` as part of a delegate that's supposed to be
created in a batch.

Drawing transparent/translucent content takes more time because translucency requires blending.
Opaque content is optimized better by the renderer.

In order to avoid paying the penalty, look for ways that you can defer the use of a transparent
`Rectangle`. Maybe you can show it on hover, or during certain events and set it to invisible when
it's no longer needed. Alternatively, you can put the `Rectangles` in an asynchronous `Loader`.

Here's a sample QML code to demonstrate the difference between using an opaque rectangle and a
transparent one when it comes to the creation time of these components.

除非您需要绘制边框，否则不应将 `Rectangle` 与透明色一起使用。 如果您将 Rectangle 用作需要批量创建的委托的一部分，情况尤其如此。

绘制透明/半透明内容需要花费更多时间，因为半透明内容需要混合。 不透明内容可以通过渲染器得到更好的优化。

为了避免付出代价，请寻找可以推迟使用透明矩形的方法。也许你可以在悬停时或某些事件中显示它，然后在不再需要它时将其设置为不可见。或者，您也可以将矩形放在异步加载器中。

下面是一段 QML 示例代码，用于演示在创建这些组件时，使用不透明矩形和透明矩形的区别。

```js
Window {
    visible: true

    Row {
        Button {
            text: "Rect"
            onClicked: {
                console.time("Rect")
                rprect.model = rprect.model + 10000
                console.timeEnd("Rect")
            }
        }

        Button {
            text: "Transparent"
            onClicked: {
                console.time("Transparent")
                rptrans.model = rptrans.model + 10000
                console.timeEnd("Transparent")
            }
        }

        Button {
            text: "Transparent Loader"
            onClicked: {
                console.time("Transparent Loader")
                rploader.model = rploader.model + 10000
                console.timeEnd("Transparent Loader")
            }
        }

        Button {
            text: "Translucent"
            onClicked: {
                console.time("Translucent")
                rptransl.model = rptransl.model + 10000
                console.timeEnd("Translucent")
            }
        }

        Button {
            text: "Reset"
            onClicked: {
                rprect.model = 0
                rptrans.model = 0
                rptransl.model = 0
                rploader.model = 0
            }
        }
    }

    Repeater {
        id: rptrans
        model: 0
        delegate: Rectangle {
            width: 10
            height: 10
            color: "transparent"
        }
    }

    Repeater {
        id: rptransl
        model: 0
        delegate: Rectangle {
            width: 10
            height: 10
            opacity: 0.5
            color: "red"
        }
    }

    Repeater {
        id: rprect
        model: 0
        delegate: Rectangle {
            width: 10
            height: 10
            color: "red"
        }
    }

    Repeater {
        id: rploader
        model: 0
        // This will speed things up. You can defer the creation of the rectangle to when it makes
        // sense and since it's asynchronous it won't block the UI thread.
        delegate: Loader {
            asynchronous: true
            sourceComponent: Rectangle {
                width: 10
                height: 10
                opacity: 0.5
                color: "transparent"
            }
        }
    }
}
```

When you run this example for the first time and create solid rectangles, you'll notice that the
creation is pretty fast. If you close it and run it again, but this time create transparent or
translucent ones you'll see that the time reported does not actually differ that much from the
solid rectangle.

The real problem starts presenting itself when you are creating new transparent items when there's
already rectangles on the scene. Try creating first the solid ones and then the transparent ones.
You'll see that the time difference is very noticeable.

Please note that this will not matter that much when you are drawing a few rectangles here and
there. The problem will present itself when you are using translucency in the context of a delegate
because there can potentially be creating thousands of these rectangles.

当你第一次运行这个示例并创建实心矩形时，你会发现创建速度非常快。如果关闭并再次运行，但这次创建的是透明或半透明的矩形，你会发现所报告的时间实际上与创建实心矩形的时间相差不大。

真正的问题出现在创建新的透明项目时，而此时场景中已经有了矩形。试着先创建实心的，然后再创建透明的，你会发现时间差非常明显。

请注意，当您在这里或那里绘制几个矩形时，这并不重要。当您在委托的上下文中使用半透明效果时，问题就会出现，因为有可能会创建成千上万个这样的矩形。

参见: [Translucent vs Opaque](https://doc.qt.io/qt-5/qtquick-performance.html#translucent-vs-opaque)
