---
title: 基于Jest和Enzyme的React测试
date: 2018-08-09 12:04:46
tags:
---

在项目的开发过程中，随着多国家的项目的逐步启动，在开发过程中在代码中添加新的需求有时会出现影响之前已有国家部分的情况发生，这给开发和测试人员都增加了工作成本。而我们作为新一代的开发人员，为了提高效率我们不断探索。于是针对前端项目，我们逐步的引入了基于Jest和Enzyme的单元及集成测试。我们可以将其视为我们对于项目的第一层“武装”，能够让我们在开发过程中实时的监测自己的代码对整个项目的影响，从而能够第一时间作出修改。

本文将简单介绍Jest框架和Enzyme框架，并且将两者结合起来对React项目进行单元测试和集成测试的尝试，以及这段时间使用过程中的一些心得。

Jest
Jest是Facebook开源的一款Javascript测试框架。Jest对React的支持非常的友好，所以React项目很自然的选到了Jest作为测试框架。Jest主要包括几个主要的特性，如测试结构，断言语句，展示和观察测试结果，snapshot测试和mock测试。

1. 测试结构
测试结构是一个测试框架的基础特性，清晰的测试结构能让测试代码更清晰，易于定位问题。Jest的测试结构是继承了Jasmine(也是一款测试框架)，基本上是由describe和test函数组成的层次结构。describe代表一个test suite，其中可包含深层的describe和多个test函数，并且通过beforeEach，afterEach等钩子函数为当前test suite下的test服务。有一点需要注意的是，在运行一个describe时，所有的test语句都是先被cache起来，之后再统一按顺序执行，所以在describe中手动设置和修改一些变量时要多留心。
![Test Structure](/images/Jest-Enzyme/test-structure.png)

2. 断言语句
Jest提供的断言语句非常丰富。除了我们常见的是非，字符串，数组类的判断外，还有为支持Snapshot的判断和Mock函数的判断。
![Assertion 1](/images/Jest-Enzyme/assertions-1.png)
![Assertion 2](/images/Jest-Enzyme/assertions-2.png)
3. 展示和观察测试结果
在显示结果时能够清晰的按测试结构展示，测试没通过的部分显示的也很醒目，定位很准确。在watch模式下，Jest会自动的检测文件的变化，并且只对文件修改的测试文件重新运行。这点为我们在写测试代码提供了很大的便利。
![Result 1](/images/Jest-Enzyme/results-1.png)

4. Snapshot测试
Snapshot是针对前端UI测试一种特殊设计，当确定UI渲染结果在特定情况下是不变时，我们可以将其生成Snapshot，并且以后每次进行比较。Snapshot测试可以说是对React量身打造，给React的组件化的单元测试提供了很大的便利。React自身就提供了react-test-render的渲染器，可以将React组件渲染为json对象，而非真正的dom元素。Jest可以将json对象生成snapshot，并进行比较。后面还会介绍Enzyme框架，Enzyme框架则会针对React组件提供更大的支持。
![Snapshot 1](/images/Jest-Enzyme/snapshot-1.png)
![Snapshot 2](/images/Jest-Enzyme/snapshot-2.png)

5. Mock函数
在测试的过程中，会遇到一些情况我们无法使用真实的函数或有时我们需要判断我们的函数是不是被执行，这时我们就需要用到Mock函数。Mock函数提供了以下几个特性：1. mock新的函数实现。2. 捕捉函数被执行的次数以及被执行时的传入参数。3. 获取通过new操作生成的实例对象们。4.mock函数的返回值。
![Mock 1](/images/Jest-Enzyme/mock-1.png)

Enzyme
Enzyme是airbnb开发的一款方便React测试，断言和操作组件的框架。前面我们提到了React自身提供了react-test-render，但是这个渲染器只是提供了简单的虚拟渲染，并且没有提供很多实用的api方便我们查找和操作React组件。然而Enzyme在这方面提供了良好的功能和简洁的api。Enzyme提供了三种渲染方式，分别为shallow，mount和render。本文将主要介绍shallow和mount。

在介绍两种渲染方式之前，要先介绍一下Enzyme为find函数提供的强大的selector。Enzyme的selectors有以下五种：1. CSS选择器 2. React属性选择器 3. React组件构造函数 4. React组件的displayName 5. 对象属性选择器。其中使用React组件的构造函数给React测试带来了很大的便利。

1. Shallow渲染
Shallow渲染为React组件的单元化测试提供了便利，这得益于这种渲染方式只关注于当前组件，并不对子组件进行深度渲染。
![Shallow](/images/Jest-Enzyme/shallow.png)

2. Mount渲染
Mount渲染又称全渲染(Full Rendering)。与Shallow渲染不同，Mount渲染是将组件真实的渲染到了dom上。在这里为模拟浏览器的dom，enzyme使用了jsdom库来模拟dom。
![Full-1](/images/Jest-Enzyme/full-1.png)
![Full-2](/images/Jest-Enzyme/full-2.png)

Tips
1. 当组件重新渲染后，总是要重新要wrapper.find获取最新的你需要的组件。
2. Full Rendering情况下，父组件重新渲染时，需要手动wrapper.update以重新渲染子组件。
3. 当组件中存在异步操作，最好使用async test，并且在组件渲染后，手动设置等待操作，以等待异步操作完成获取最新的组件进行测试。

可能是比较好的实践
1. 当组件为纯展示的基础组件时，可以使用Snapshot Test。
2. 当组件为带有内部状态转换的基础组件时，可以使用Shallow Rendering Test。
3. 当组件为容器组件时，时常配合有Redux，可以使用Full Rendering Test。
