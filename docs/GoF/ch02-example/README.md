# 实例研究:设计一个文档编辑器

- task

  设计一个称为 Lexi 的"所见即所得"(或"WYSIWYG")的文档编辑器

- 设计问题

  - 文档结构

    对文档内部表示的选择

    编辑、格式安排、显示和文本分析会受到`表示`的影响, 如何考虑

  - 格式化

    怎样将文本和图形安排到行和列上的

    有多种格式化策略

    策略如何作用`表示`

  - 修饰用户界面

    包括滚动条、边界和文档界面的阴影

    随着用户界面的演化而发生变化

    能自由增加和去除这些修饰

  - 多种 UI 风格(支持多种视感)

    不需作较大修改就能适应不同的视感标准

  - 跨平台(支持多种窗口系统)

    尽可能的独立于特定平台

  - 用户操作

    不同的用户界面控制 Lexi, 如按钮和下拉菜单都有复制命令

    提供一个统一的机制, 既可以访问这些分散的功能, 又可以对操作进行撤消(undo)

  - 文本分析(拼写检查和连字符)

    支持检查拼写错误和决定连字符的连字点分析操作

    添加一个新的分析操作时，尽量少修改相关的类

- 针对每个问题分析套路

  - 一组相关联的目标集合

  - 怎样达到这些目标的限制条件集合

  - 一个或多个设计模式

  - 问题的讨论

## 文档结构

- 两种不同的看待方式

  - 对字符、线、多边形和其他图形元素的一种安排

  - 并非图形项，而是看作文档的物理结构一行、列、图形、表和其他子结构

- 效果

  - 用户界面应该让用户直接操纵这些子结构

    用户应该能够将一个图表当作一个单元，而不是个别图形原语的一组集合

    用户应该能够对表进行整体引用，而不是将表作为非结构化的一堆文本和图形

为实现效果, 选择能匹配文档物理结构的内部表示, 即第二种看待方式

- 特性

  - 保持文档的物理结构

    即将文本和图形安排到行、列、表等

  - 可视化生成和显示文档

  - 根据显示位置来映射文档内部表示的元素

    用户在可视化表示中所点击的某个东西来决定用户所引用的文档元素

- 限制条件

  - 一致对待文本和图形

    应用界面允许用户在图形中自由的嵌入文本，反之亦然

  - 不过分强调内部表示中单个元素和元素组之间的差别

    可以插入字符的地方也可以插入多元素组成的图标

  - 需要对文本进行分析

    与第二个限制条件有冲突, 图形无法文本分析

### 结构安排方式

递归组合

- 层次结构信息的表述

  通常由递归组合(Recursive Composition)来实现的

  递归组合可以由较简单的元素逐渐建立复杂的元素

- 使用递归组合的对象结构作`内部表示`

  字符和图形从左到右排列形成文档的一行，然后由多行形成一列，再由多列形成一页

  ```mermaid
  graph TD
      A(列) -->|组合| B(行)
      B -->|组合| C(字符)
      B -->|组合| D(空格)
      B -->|组合| E(fa:fa-car Car)
      A -->|组合| F(行)
      F -->|组合| I(...)
      A -->|组合| G(...)
  ```

### 实现结构

- 方式

  每个重要元素当作一个对象处理

- 要求

  可以描述物理结构

  不仅包括字符、图形等可见元素, 也包括不可见的、结构化的元素, 如行和列

  在显示、格式化和互相嵌入等方面一致对待图形和文本

- 设计

  文档结构中的所有对象抽象为一个抽象类图元(Glyph)

  Glyph 包含了基本图形元素(字符和图片)和结构化元素(行、列、表)

- Glyph 类层次

  ```mermaid
  classDiagram
  class Glyph{
      <<abstract>>
      +draw(Window)*
      +intersects(Point)*
      +insert(Glyph, int)*
  }

  class Character{
      char: c
      +draw(Window)
      +intersects(Point)
      +insert(Glyph, int)
  }

  class Row{
      children
      +draw(Window)
      +intersects(Point)
      +insert(Glyph, int)
  }

  Glyph <|-- Character
  Glyph <|-- Rectangle
  Glyph <|-- Polygon
  Glyph <|-- Row
  Glyph --o Row: children
  ```

- 基于 Glyph 接口

<div style="text-align:center">
  <table style="width: 100%;margin:auto">
      <tr align="center">
          <td><b>Responsibility</b></td>
          <td><b>Operations</b></td>
      </tr>
      <tr align="center">
          <td rowspan = "2">Appearance</td>
          <td>Virtual void Draw(Window*)</td>
          </tr>
      <tr align="center">
          <td>Virtual Void Bounds (Rect&)</td>
          </tr>
      <tr align="center">
          <td>hit detection</td>
          <td>Virtual bool Intersects (Const Point&)</td>
      </tr>
      <tr align="center">
          <td rowspan = "4">Structure</td>
          <td>Virtual Void Insert (Glyph*, int)</td>
      </tr>
      <tr align="center">
          <td>Virtual Void Remove (Glyph*)</td>
          </tr>
      <tr align="center">
          <td>Virtual Glyph*Child (int)</td>
      </tr>
      <tr align="center">
          <td>Virtual Glyph*Parent（）</td>
      </tr>
  </table>
</div>

- 接口三个基本责任

  - 怎样画出自己

  - 它们占用多大空间

    父图元需要知道子图元空间大小以便确定自身大小

    Intersects 操作判断一个指定的点是否与图元相交

  - 一个图元的父图元和子图元是什么

### 模式

Composite 模式

- 核心

  递归组合

  不仅可用来表示文档，也可表示任何潜在复杂的、层次式的结构

  Composite 模式描述了面向对象的递归组合的本质

## 格式化

- task

  - 构造一个特殊物理结构

    该结构对应于一个恰当格式化了的文档

记录文档物理结构的能力并没有告诉我们怎样得到一个特殊格式化结构

- 限制

  讨论特殊情形

  "格式化"含义限制为将一个图元集合分解为若干行

### 封装概念

- 问题

  `格式化的质量`和`格式化的速度`的权衡

- 方案

  备选多种不同格式化算法, 根据环境选取

- 问题转化

  问题转化为如何支持不同的格式化算法

- 效果

  - 算法和 Gyph 独立

    自由地增加一个 Gyph 子类而不用考虑格式算法

    增加一个格式算法不应要求修改已有的图元类

- 实现

  定义一个封装格式化算法的对象的类层次结构, 称为`Compositor`

  类层次结构的根结点将定义支持许多格式化算法的接口

  每个子类实现这个接口以执行特定的算法

  Glyph 子类对象`Composition`(类似文档)自动使用给定算法对象来排列其子图元

### 实现

- Compositor 和 Composition

  - Compositor

    封装算法

  - Composition

    储存图元, 以格式化

  - 基本 Compositor 接口

    | Responsibility |             Operations              |
    | :------------: | :---------------------------------: |
    |   格式化内容   | `void setComposition(Composition*)` |
    |   何时格式化   |      `virtual void compose()`       |

- Composition 和 Compositor 类间的关系

  ```mermaid
  classDiagram
  class Glyph{
      insert(Glyph, int)*
  }
  class Composition{
      children
      compositor
      insert(Glyph, int)
  }
  class Compositor{
      composition
      setComposition(Composition*)
      compose()
  }
  class ArrayCompositor{
      compose()
  }
  class TextCompositor{
      compose()
  }
  class SimpleCompositor{
      compose()
  }

  Glyph <|-- Composition
  Glyph --o Composition:children
  Composition <-- Compositor:composition
  Composition o-- Compositor:compositor
  Compositor <|-- ArrayCompositor
  Compositor <|-- TextCompositor
  Compositor <|-- SimpleCompositor
  ```

- 过程

  - 未格式化

    Composition 只包图形元素, 不包含行和列等结构元素

  - 格式化

    Composition 调用它的 Compositor 的 Compose 操作

    Compositor 依次遍历 Composition 的各个子图元，根据分行算法插入新的行和列图元

  ```mermaid
  graph TD
      A(列 insert) -->|组合| B(行 insert)
      B -->|组合| C(字符)
      B -->|组合| D(空格)
      B -->|组合| E(fa:fa-car Car)
      A -->|组合| F(行 insert)
      F -->|组合| I(...)
      A -->|组合| G(...)
      J(composition include compositor) --> A
  ```

### 模式

Strategy 模式

- 参与者

  - Strategy

    Compositor 即 Strategy, 封装了不同的格式算法

  - Strategy 的操作环境

    Composition 即 Compositor 策略的环境

- 核心

  为 Strategy 和它的环境设计足够通用的接口

  因此, 无需为支持一个新的算法而改变 Strategy 或它的环境。

- 运用

  Composition 的接口 Insert 足够通用, 满足了 Compositor 子类任何算法对文档的物理结构的修改的需求

  Compositor 接口也足以支持 Composition 启动格式化操作

## 修饰用户界面

- 任务

  - 为 Lexi 添加修饰

    - 边界

      界定文本页

    - 滚动条

      让用户能看到同一页的不同部分

- 效果

  - 便于增加和去除这些修饰

- 问题转化

  如果其他用户界面对象不知道存在这些修饰, 那么我们就获得了最大的灵活性

  这使我们无需改变其他的类就能增加和移去这些修饰

  > 换而言之, 透明的
  >
  > 想到生成器模式的问题

### 封装概念

Transparent Enclosure

- 两个组合候选对象

  图元(Glyph)

  边界(Border)

- 谁来进一步修饰谁

  - 边界修饰图元

    边界在屏幕上包围了图元的感觉

  - 图元修饰边界

    对相应的 Glyph 子类作修改以使边界对所有子类有效

- border

  - 边界有形

    说明它应该是图元,即 Border 类应该是 Glyph 的子类

  - 图元一致性

    客户应该一致地对待图元，而不应关心图元是否有边界

    当客户画一个简单的、无边界的图元时，就不必对它作修饰。如果那个图元包含于一个边界对象中，客户应该以画出前面简单图元同样的方法画出这个边界对象，而不应该特殊对待该边界对象

    暗示了 Border 接口是与 Glyph 接口匹配的, 即使用相同的接口

得出了透明围栏(Transparent Enclosure)的概念

- 它结合了两个概念

  - 单子女(单组件)组合

  - 兼容的接口

  客户通常分辨不出它们是在处理组件还是组件的围栏(即，这个组件的父组件)，特别是当围栏只是代理组件的所有操作时更是如此。但是围栏也能通过在代理操作之前或之后添加一些自己的操作来修改组件的行为。围栏也能有效地为组件添加状态。

### MonoGlyph

透明围栏的概念可以用于所有的修饰其他图元的图元

- 概念具体化

  定义修饰作用的图元的抽象为 MonoGlyph

  ```mermaid
  classDiagram
  class Glyph{
    draw(Window)*
  }
  class MonoGlyph{
    component: Glyph*
    draw(Window)
  }
  class Border{
    - component: Glyph*
    + Border(Glyph*)
    + draw(Window)
    - drawBorder(Window)
  }
  class Scroller{
    - component: Glyph*
    + Scroller(Glyph*)
    + draw(Window)
  }
  Glyph <|-- MonoGlyph
  Glyph --o MonoGlyph:component
  MonoGlyph <|-- Border
  MonoGlyph <|-- Scroller
  ```

  - Border::draw 的实现

    - 先调用父类 MonoGlyph::draw

    - 调用私有操作 DrawBorder 来画出边界

- 使用效果

  ```JavaScript
  doc = new Composition()
  doc_with_scroller = new Scroller(doc)
  doc_with_scroller_and_border = new Border(doc_with_scroller)
  ```

  > 一步一步进行修饰, 形成一个 stack 结构

现在准备给 Lexi 文本编辑区增加边界和滚动界面

- 在一个 Scroller 实例中组合已存在的 Composition 实例以增加滚动界面，然后再把它组合到 Border 实例中

  ```mermaid
  graph TD
      A(列 insert) -->|组合| B(行 insert)
      B -->|组合| C(字符)
      B -->|组合| D(空格)
      B -->|组合| E(fa:fa-car Car)
      A -->|组合| F(行 insert)
      F -->|组合| I(...)
      A -->|组合| G(...)
      J(composition include compositor) --> A
      K(scroller) --> J
      L(border) --> K
  ```

  也可以交换组合顺序，把一个带有边界的组合放在 Scroller 实例中。这样边界可以和文本一起滚动

  无论何种，透明围栏使得试验不同的选择变得很容易，使得客户和修饰代码无关

- 注意

  Border:是怎样组合一个而不是两个或多个 Glyph 对象的。

  这里讲给某物加上边界暗示了"某物"是单个的

### 模式

Decorator 模式

- Decorator 模式

  描述了以透明围栏来支持修饰的类和对象的关系

  修饰指给一个对象增加职责的事物

  想到用语义动作修饰抽象语法树、用新的转换修饰有穷状态自动机或者以属性标签修饰持久对象网等例子

## 多种 UI 风格

- 可移植性

  - 障碍

    不同视感标准之间的差异性

- 目的

  拥有多个不同的 UI 风格

  容易增加新的风格

  运行时刻可以改变 Lexi 的 UI 风格

### 实现

对对象创建进行抽象

- 不同风格抽象为同一的概念

  ```mermaid
  classDiagram
  class ScrollBar{
    scrollTo(int)
  }
  class MotifScrollBar{
    scrollTo(int)
  }
  class PMScrollBar{
    scrollTo(int)
  }
  class MacScrollBar{
    scrollTo(int)
  }
  class Button{
    click()
  }
  class MotifButton{
    click()
  }
  class PMButton{
    click()
  }
  class MacButton{
    click()
  }
  class Menu{
    popUp()
  }
  class MotifMenu{
    popUp()
  }
  class PMMenu{
    popUp()
  }
  class MacMenu{
    popUp()
  }
  Glyph <|-- ScrollBar
  Glyph <|-- Button
  Glyph <|-- Menu
  ScrollBar <|-- MotifScrollBar
  ScrollBar <|-- PMScrollBar
  ScrollBar <|-- MacScrollBar
  Button <|-- MotifButton
  Button <|-- PMButton
  Button <|-- MacButton
  Menu <|-- MotifMenu
  Menu <|-- PMMenu
  Menu <|-- MacMenu
  ```

- 产品与产品之间构成系列

  ```mermaid
  classDiagram
  class GUIFactory{
    createScrollBar()
    createButton()
    createMenu()
  }
  class MotifFactory{
    createScrollBar()
    createButton()
    createMenu()
  }
  class PMFactory{
    createScrollBar()
    createButton()
    createMenu()
  }
  class MacFactory{
    createScrollBar()
    createButton()
    createMenu()
  }
  GUIFactory <|-- MotifFactory
  GUIFactory <|-- PMFactory
  GUIFactory <|-- MacFactory
  ```

  > 保持风格一致

### 模式

Abstract Factory 模式

- 参与者

  - Factory

  - Product

- 描述

  不直接实例化类的情况下创建一系列相关的产品对象

- 适用

  产品对象的数目和种类不变，而具体产品系列之间存在不同的情况

  实例化特定的具体工厂对象来选择产品系列

  用不同的具体工厂实例来替换原来的工厂对象以改变整个产品系列

- 对比

  抽象工厂模式对产品系列的强调使它区别于其他只与一种产品对象有关的创建性模式

## 跨平台

- 移植性

  - 障碍

    Lexi 所运行的窗口环境

### 分析

- 是否可用 Abstract Factory 模式

  > 考虑之前的抽象工厂模式, 对于 ScrollBar, 不同 UI 风格, ScrollBar 都是有同一的认识, 同一的概念, 都有滚动功能, 都能显示出形状, 所以可以抽象出一个 ScrollBar
  >
  > 若同等地对待操作系统, 则意味需要抽象出整个操作系统,然后去实现多个不同的操作系统. 而操作系统的实现, 功能, API 有很大差别, 使用 Abstract Factory 相当困难

  系统移植的限制条件与 UI 风格的独立性条件是有极大不同的

- 背景

  > 重点关心的是窗口运行的功能, 而不关系具体的操作系统是什么
  >
  > 不要在操作系统的层面上思考, 转而考虑高级的抽象, 思考窗口
  >
  > 也就是实现与抽象分离

- 解决方法

  对不同的窗口系统作一个统一的抽象, 在对各窗口系统的实现作一些调整, 使之符合公共的接口

### 封装概念

实现依赖关系

- Window 类封装了窗口要各窗口系统都要做的一些事情

  - 提供画基本几何图形的操作

  - 能变成图标或还原成窗口

  - 能改变自己的大小

  - 它们能够根据需要画出(或重画出)窗口内容

    例如, 当它们由图标还原为窗口时, 或它们在屏幕空间上重叠、出界的部分重新显示时, 都要重画

- Window 接口

<div style="text-align:center">
  <table style="width: 100%;margin:auto">
      <tr align="center">
          <td><b>Responsibility</b></td>
          <td><b>Operations</b></td>
      </tr>
      <tr align="center">
          <td rowspan = "6">窗口管理</td>
          <td>virtual void redraw()</td>
      </tr>
      <tr align="center">
          <td>virtual Void rasise()</td>
      </tr>
      <tr align="center">
          <td>virtual Void lower()</td>
      </tr>
      <tr align="center">
          <td>virtual Void iconify()</td>
      </tr>
      <tr align="center">
          <td>virtual Void deiconify()</td>
      </tr>
      <tr align="center">
          <td>...</td>
      </tr>
      <tr align="center">
          <td rowspan = "5">图形</td>
          <td>virtual void drawLine()</td>
      </tr>
      <tr align="center">
          <td>virtual Void drawRect()</td>
      </tr>
      <tr align="center">
          <td>virtual Void drawPolygon()</td>
      </tr>
      <tr align="center">
          <td>virtual Void drawText()</td>
      </tr>
      <tr align="center">
          <td>...</td>
      </tr>
  </table>
</div>

- Window 跨平台功能

  - 功能的交集

    类似于最小功能的窗口系统

    对一些即使是大多数窗口系统都支持的高级特征, 也无法利用

  - 功能并集

    创建一个合并了所有存在系统的功能的接口

    接口势必规模巨大,并且存在不一致的地方

    当某个厂商修改它的窗口系统时, 我们不得不修改这个接口和 Lexi, 因为 Lexi 依赖于它

  采取折中的方法

Window 是一个抽象类, 针对的是 Lexi 的应用, 而非面向操作系统

- 实现方案

  - 实现 Window 类和它的子类的多个版本,每个版本对应一个窗口平台

  - 为每一个窗口层次结构中类创建特定实现的子类, 容易子类数目爆炸

  上述都不可靠

- 对变化的概念进行封装

  变化的是窗口系统实现

  将`窗口系统的实现`封装为一个对象 WindowImp, WindowImp 可以采取 Abstract Factory 模式完成跨平台

  而 Window 和它的子类根据 WindowImp 实现针对应用端的功能

### 实现

- Window 和 WindowImp

  ```mermaid
  classDiagram
  class Window{
    - imp
    raise()
    drawRect()
  }
  class WindowImp{
    deviceOperate()
  }
  Window <|-- ApplicationWindow
  Window <|-- IconWindow
  Window <|-- DialogWindow
  Window o-- WindowImp : imp
  WindowImp <|-- MacWindowImp
  WindowImp <|-- PMWindowImp
  WindowImp <|-- XWWindowImp
  ```

- 要点

  Window 是针对应用程序, 而 WindowImp 是针对窗口系统的

  WindowImp 稳定的接口隐藏了各个窗口系统接口的差异, 它的设计是受不同于 Window 接口的限制条件驱动的, 实现角度可以不必和 Window 的客观世界视图一致

  Window 子类将更多的精力放在窗口的抽象上,而不是窗口系统的细节

### 模式

Bridge 模式

- 目的

  允许分离的类层次一起工作,即使它们是独立演化的

  允许保持和加强我们对窗口的逻辑抽象, 而不触及窗口系统相关的代码. 反之也一样

- 核心

  创建了两个分离的类层次,一个支持窗口的逻辑概念,另一个描述了窗口的不同实现

## 用户操作

- 背景

  - 复制可以用快捷键, 也可以鼠标右键点击, 还可以点击菜单

  - 支持撤销(undo)和重做(redo) 操作

### 封装概念

封装一个请求

- 设计者的角度

  下拉菜单为会对鼠标点击做出反应的又一种图元

  于是, 可设计 MenuItem 为 Glyph 的子类

- 问题

  如果用继承方式实现, 继承`请求`和`形状`来实现, 则子类数目爆炸, `子类数目 = 形状的数目 * 请求的数目`

  同时, 子类不稳定, `请求`和`形状`紧密绑定了

可以从问题中看出, 需要一种允许用菜单项所执行的请求对菜单项进行参数化的机制

- 考察函数进行参数化

  - 缺点

    - 没有强调撤销/重做操作

    - 很难将状态和函数联系起来

      例如,一个改变字体的函数需要知道是哪一种字体

    - 很难扩充, 很难部分地复用它们

- 用对象来参数化 MenuItem

  - 通过继承扩充和复用请求实现
  - 可以保存状态
  - 实现撤销/重做功能

这里是另一个封装变化概念的例子, 即封装请求

### 实现

Command 类及其子类

- 定义一个 Command 抽象类

  提供发送请求的接口'execute'

- 子类实现 execute 方法

  满足不同的请求

  可以将部分或全部工作委托给其他对象

  可以完全由自己来满足请求

  ```mermaid
  classDiagram
  class Command {
    execute()*
  }
  class PasteCommand{
    buffer
    execute()
  }
  class FontCommand{
    execute()
  }
  class SaveCommand{
    execute()
  }
  class QuitCommand{
    execute()
  }
  Command <|-- PasteCommand
  Command <|-- FontCommand
  Command <|-- SaveCommand
  Command <|-- QuitCommand
  SaveCommand <-- QuitCommand
  ```

请求者看到的只是 Command 对象

- 实例化

  给每个菜单项一个适合该菜单项的 Command 子类实例, 就像每个菜单项指定一个文本字符串

  ```mermaid
  classDiagram
  class MenuItem{
    click()
  }
  class Command{
    execute()
  }
  Glyph <|-- MenuItem
  MenuItem o-- Command
  ```

### 撤销和重做

- 参考:[命令行模式](https://refactoringguru.cn/design-patterns/command)

- 考虑逆操作

  Command 接口中增加 Unexecute 操作

  使用上一次 execute 操作所保存的取消信息来消除 execute 操作的影响

  - 例如

    在 FontCommand 的例子中, execute 操作会保存改变字体的文本区域和以前的字体。 FontCommand 的 Unexecute 操作将把这个区域的文本回复为以前的字体

- 注意

  如果用户多次重复无意义的字体改变操作,他应该不必执行相同数目的撤销操作才可以返回到上一次有意义的操作。如果执行一个命令不产生任何影响,那么就不需要相应的撤销操作

  因此为了决定一个命令是否可以撤销,我们给 Command 接口增加了一个抽象的 Reversible 操作, 它返回 Boolean 值

### 命令历史记录

- 支持任意层次的撤销和重做命令的最后一步是定义一个 命令历史记录
- 或已执行的命令列表

### 模式

Command 模式

## 文本分析

拼写检查和断字处理

- 目的

  - 支持权衡

    支持多个算法, 一组不同算法的集合能够提供时间/空间/质量选择时的权衡

  - 支持多种分析功能

    可能会加入查找、字数统计、计算表格总值的设施、语法检查等等

    引入这种新功能不改变 Glyph 类及其子类

- 深层问题

  - 访问需要分析的信息, 信息分散在文档结构

  - 分析这些信息

### 问题一

访问分散的信息

- 屏蔽访问对象的实现

  不需要知道图元中底层数据结构

  只有图元自己知道它所使用的数据结构

  说明:图元接口不应该偏重于某个数据结构

- 支持不同方式访问信息

  访问机制必须能容纳不同的数据结构, 并且我们还必须支持不同的遍历方法, 如前序、后序和中序

### 封装概念

访问和遍历

- 方法

  将多个访问和遍历功能直接放到图元类中, 并提供一种选择方式, 这可能是通过增加一个枚举常量作为参数

  类在遍历过程中传递该参数以确保所用的是同一种遍历方式, 它们必须传递遍历过程中积累的任何信息

- Glyph 增加的接口

  - `void first(Traversal kind)`

    初始化遍历过程

    根据枚举类型 Traversal 的参数值确定执行何种遍历,其值可以是 CHILDREN (只遍历图元的直接子图元) 、PREORDER (以先序方式遍历整个结构) 、POSTORDER 和 INORDER

  - `void next()`

    遍历时前进到下一个图元

  - `bool isDone()`

    报告遍历是否完成

  - `Glyph* getCurrent()`

    代替 Child 操作,它访问遍历的当前图元

  - `void insert(Glyph* )`

    代替了以前的操作, 在当前位置插入给定的图元

此时已经放弃了图元接口的数字索引方式, 这样不会偏重于某种数据结构

- 问题

  在不扩展枚举值或增加新的操作的条件下, 不能支持新的遍历方式

  比方说, 修改一下先序遍历,使它能自动跳过非文本图元

  届时, 不得不改变枚举类型 Traversal,使它包含 TEXTUAL_PREORDER 这样的值

- 深层问题

  把遍历机制完全放到 Glyph 类层次中,将会导致修改和扩充时不得不改变一些类

- 封装变化的概念

  - 指的是访问和遍历机制, 定义为 iterators

- 目的

  定义这些机制的不同集合

- 实现

  通过继承来统一访问不同的数据结构和支持新的遍历方式, 同时不改变图元接口或打乱已有的图元实现

### 实现

Iterator 类及其子类

使用抽象类 Iterator 为访问和遍历定义一个通用的接口

- 设计

  ```mermaid
  classDiagram
  class Iterator{
    + first(Traversal kind)*
    + next()
    + isDone()
    + getCurrent()
    + insert(Glyph* )
  }
  class PreorderIterator{
    - root
    - iterators
    + first(Traversal kind)
    + next()
    + isDone()
    + getCurrent()
    + insert(Glyph* )
  }
  class ArrayIterator{
    - currentItem
    + first(Traversal kind)
    + next()
    + isDone()
    + getCurrent()
    + insert(Glyph* )
  }
  class ListIterator{
    + first(Traversal kind)
    + next()
    + isDone()
    + getCurrent()
    + insert(Glyph* )
  }
  class NullIterator{
    + first(Traversal kind)
    + next()
    + isDone()
    + getCurrent()
    + insert(Glyph* )
  }
  class Glyph{
    + createIterator()*
  }
  Iterator <|-- PreorderIterator
  Iterator <|-- ArrayIterator
  Iterator <|-- ListIterator
  Iterator <|-- NullIterator
  Iterator <-- PreorderIterator:iterators
  PreorderIterator --> Glyph:root
  ArrayIterator --> Glyph
  ListIterator --> Glyph
  ```

  在缺省情况下 createIterator 返回一个 NullIterator

  NullIterator 是一个退化的 Iterator, 它适用于叶子图元, 即没有子图元的图元

  一个有子女的图元 Glyph 子类将重载 CreateIterator, 返回不同 Iterator 子类的一个实例,这依赖于保存图元子女所用的结构

  ```JavaScript
  preorder = new PreorderIterator(glyphObj);
  ```

- 注意 Iterator 类层次结构

  允许不改变图元类而增加新的遍历方式

### 模式

Iterator 模式

抽象了遍历算法, 对客户隐藏了它所遍历对象的内部结构

Iterator 模式再一次说明了怎样封装变化的概念, 有助于获得灵活性和复用性

- 应用

  可用于组合结构也可用于集合

### 问题二

遍历和遍历过程中的动作

- 背景

  分析过程涉及到遍历过程中的信息积累

- 深层问题

  将分析的责任放在什么位置上

- 解决方法

  - 可以在遍历的时候分析, 但显然将遍历和分离比较合适

  - 可以放在 Glyph 类中

    不同的分析过程必然是分析不同的图元, 因而一个给定的分析必须能区别不同种类的图元

    很明显的一种做法是将分析能力放到图元类本身

    但问题是每增加一种分析, 都必须改变每一个图元的类

- 方案的问题

  随着新的分析功能的增加, Glyph 的接口会越来越大

  分析操作增多会逐渐模糊基本的 Glyph 接口, 从而很难看出图元的主要目的是定义和结构化有外观和形状的对象

### 封装概念

封装分析

- 在一个独立对象中封装分析方法

- 基本想法

  - 封装分析对象 Visitor, 结合合适的 Iterator

  - Iterator 负责将 Visitor 携带到所遍历结构的每一个图元中

  - Visitor 在遍历过程中, 积累它所感兴趣的信息

- 问题

  遍历的时候, 看到的都是 Glyph 类, 而不是它的子类

  分析对象怎样才能不使用类型测试或强制类型转换也能正确对待各种不同的图元

- 解决方法

  - 用 `dynamic_cast<>()`

    将 Glyph 转化为具体的子类

    但是难以扩展

  - Glyph 添加 Check 接口

    Glyph 的子类通过这个接口反过来调用针对子类的分析方法

    ```cpp
    class rect: public Glyph
    {
      public:
      void checkMe(Visitor& v) override
      {
        v.visitRect(this);
      }
    }
    ```

    但是每增加一种新的分析, Glyph 及其子类增加类似于 checkMe(Visitor& v) 的操作

- 深层问题

  用有通用参数的与分析无关的操作来替代像 checkMe(Visitor& v) 这样表示特定分析的操作

### 实现

- 参考[访问者模式](https://refactoringguru.cn/design-patterns/visitor)和[访问者和双分派](https://refactoringguru.cn/design-patterns/visitor-double-dispatch)

Visitor 类及其子类

### 模式

Visitor 模式

- 核心

  哪一个类层次变化得最厉害

- 场景

  对一个稳定类结构的对象做许多不同的事情的情况

  增加一种新的访问者而不需要改变类结构,这对于很大的类结构是尤其重要的。

- 优势

  访问者所能访问的类之间无需通过一个公共父类关联起来 访问者能跨越类层次结构

- 缺陷

  类结构增加了一个子类, 得更新所有访问者类的接口以包含针对那个子类的 Visit 操作
