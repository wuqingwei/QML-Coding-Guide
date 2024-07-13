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

这是因为，从心理上讲，这样会使画面更美观，因为 Component.onCompleted 会在组件构建完成时触发。


------

如果一个项目中有多个信号处理器，那么行数最少的处理器可能会被放在最前面。行数最少的处理程序可以放在最前面。随着执行行数的增加，处理程序 也会向下移动。唯一的例外是 Component.onCompleted 信号。总是放在最下面。

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

## CS-2: Property Initialization Order

第一个属性赋值必须始终是组件的 id。如果 要声明组件的自定义属性，声明总是在第一个属性赋值之上。

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

如果属性赋值与信号处理程序混在一起，通常更难查看。这就是为什么我们要把赋值放在信号 处理程序上。

### CS-3: Function Ordering

虽然 QML 中没有 private 和 public 函数，但您可以通过封装只应在 QtObject 内部使用的属性和函数来提供类似的机制。

公共函数的实现总是放在文件的最底部。尽管我们 优先将其他类型的公共声明放在文件顶部，但我还是鼓励您将公共函数放在底部。因为如果一个函数的行数变多，会大大降低 QML 文档的可读性、 会大大降低 QML 文档的可读性。理想情况下，您不应该有任何 函数，而尽量依赖组件的声明属性。尽可能依赖组件的声明属性。

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

在使用任何动画子类时，尤其是嵌套的动画子类，如 SequentialAnimation 等嵌套类时，请尽量减少一行中的属性数量。
如果在同一行中有超过 2-3 个赋值，就会变得难以推理。之后就很难推理了。或者，您可以将单行赋值保持在您为项目设置的任何行长 约定。

由于动画很难在脑海中想象出来，因此尽可能简单的动画会让您受益匪浅。让动画尽可能简单。

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

### CS-5: Specifying IDs for Objects

如果不需要访问某个对象的功能，则应避免设置 id 属性。这样就不容易出现 id 重复的问题。此外，为对象设置 id 会给我们带来额外的认知压力，因为这意味着我们需要照顾额外的关系。

如果想用描述符标记类型，但又不打算引用该类型，可以使用 objectName 代替，或者使用普通的注释。

确保文件中最顶部的组件始终以 root 作为其 id。
Qt 将在 QML 3 中淘汰非限定名称查找，因此最好现在就开始给组件 ID 并使用限定查找。所以最好现在就开始为组件赋予 ID 并使用限定查找。

详见 [QTBUG-71578](https://bugreports.qt.io/browse/QTBUG-71578)  和[QTBUG-76016](https://bugreports.qt.io/browse/QTBUG-76016) 

### CS-6: Property Assignments

在分配分组属性时，如果只更改一个属性，则应使用点符号。否则，请务必使用分
组符号。

```qml
Image {
    anchors.left: parent.left // Dot notation
    sourceSize { // Group notation
        width: 32
        height: 32
    }
}
```

当您在同一文件的不同地方为 Loader 的 sourceComponent 分配组件时，请考虑使用相同的实现。例如，在 有两个相同组件的实例。如果这两个 这些 SomeSpecialComponent 都是相同的，那么最好的办法就是 将 SomeSpecialComponent 封装在一个 Component 中。

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

这样就可以确保每当您对 specialComponent 进行更改时，它都会在所有加载器中生效。而在糟糕的示例中，您必须复制 相同的更改。

在不使用 Loader 的类似情况下，您可以使用内联components。

```qml
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

绑定 的 when 属性可用于根据条件启用或禁用绑定表达式。如果您使用的绑定很复杂，不需要在每次属性变化时都执行绑定，那么这是减少绑定执行次数的好主意。

使用上面的同一个示例，我们可以使用绑定对象将其重写如下。

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

~您还可以使用 Qt.callLater 来减少对函数的冗余调用。

## B-6: Avoid Unnecessary Re-Evaluations

避免不必要的重新赋值？

如果您有一个循环或流程需要更新属性值，您可能需要使用临时局部变量来累积这些变化。然后只
报告属性的最后一个值。这样就可以避免在累加的中间阶段触发对绑定表达式的重新评估。

下面是一个直接来自 Qt 文档的糟糕示例：

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

下面是合适的方式:

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

还要注意的是，列表类型没有与添加/移动/删除元素相关的更改信号。如果要使用列表类型存储顺序数据，请确保使用该属性的地方不会执行昂贵的操作（例如填充视图）。

# C++ Integration

QML 可用 C++ 扩展，方法是使用 Q_OBJECT 宏公开 QObject 类，或使用 Q_GADGET 宏公开自定义数据类型。使用 C++ 为 QML 应用程序添加功能总是首选。
但重要的是要知道哪种是公开 C++ 类的最佳方式，这取决于您的使用情况。

## CI-1: Avoid Context Properties 避免使用上下文属性

用法
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

见 [QTBUG-73064](https://bugreports.qt.io/browse/QTBUG-73064).

## CI-2: Use Singleton for Common API Access

使用单例来访问通用应用程序接口

在某些情况下，您必须为某个功能或公共数据访问提供单个实例。在这种情况下，请使用单例，因为它将具有更好的性能并且更容易阅读。单例也是向QML公开枚举的一个很好的选择。
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

实例化类型通过以下方式暴露给QML：

```cpp
// In main.cpp
qmlRegisterType<ColorModel>("MyNameSpace", 1, 0, "ColorModel");
```

实例化类型的好处是，在同一文档中有所有可供您理解和消化的内容。它们更容易在运行时更改而不会产生副作用，并且易于推理，因为在查看文档时，您不需要担心任何全局状态，而是需要担心手头正在处理的类型的状态。

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

## CI-4: Watch Out for Object Ownership Rules 注意对象所有权规则

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

```qml
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

See also: [Translucent vs Opaque](https://doc.qt.io/qt-5/qtquick-performance.html#translucent-vs-opaque)
