## 设计模式
设计模式的目的是为了可重用代码，提高代码的可扩展性和可维护性。
设计模式的主要工作是找到变化点，然后封装并隔离变化点，让变化的另一侧保持稳定，从而将变化带来的影响降到最低。

### 8个设计原则
八大设计原则分别是：依赖倒置、开放封闭、单一职责、里氏替换、接口隔离、优先使用组合而不是继承、封装变化点、针对接口编程。

* 依赖倒置原则
  - 高层模块不应该依赖底层模块，二者都应该依赖于抽象。
  - 抽象不应该依赖细节，实现细节应该依赖于抽象。
  - 稳定不应该依赖变化。
* 开放封闭原则
  - 对扩展开放，对更改封闭。
  - 类是可扩展的，但是不可修改。(通过组合或者继承的方式进行功能扩展)
* 单一职责原则
  - 一个类应该只有一个引起它变化的原因，否则类应该被拆分。(可以降低类的复杂性，提高可读性，降低变更引起的风险)
  - 变化的方向隐含着类的职责。
* 里氏替换原则
  - 阐述继承应该满足"is-a"的关系。
  - 子类可以扩展父类的功能，但不能改变父类原有的功能。(尽量不要重写父类的方法)
* 接口隔离原则
  - 接口应该小而完备
  - 粒度太小导致接口很多，粒度太大降低灵活性，无法提供定制服务。
* 优先使用组合而不是继承
  - 类继承称为"白箱复用"，对象组合称为"黑箱复用"。
  - 继承某种程度上破坏了封装性，子类和父类的耦合度高。
  - 对象组合只要求被组合对象具有良好的接口，耦合度低。
* 封装变化点
  - 使用封装来创建对象之间的分界层，让设计者可以在一侧进行修改，而不会对另一个产生不良影响，从而实现层次间的松耦合。
  - 具体而言，变化点利用抽象找出不变点，变化的都依赖于这个抽象，且变化的还可以基于抽象而不断派生，变化。
* 针对接口而不是实现编程
  - 不将变量类型声明为某个特定类具体类，而是声明为某个接口。
  - 客户程序无需获知对象的具体类型，只需要知道对象所具有的接口。
  - 具体而言，是基于基类多态接口编程，而不是某个特定子类类型。这样可以忽略类型，专注实现和调用接口。


### SOLID原则
SOLID分别指单一功能(SRP)、开闭原则(OCP)、里氏替换(LSP)、接口隔离(ISP)以及依赖翻转(DIP)。









