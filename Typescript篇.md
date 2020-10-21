## Typescript

1.interface: ts里面有个核心原则就是检查变量的形状，这个工作主要是由interface来完成的，我们可以用 interface 来表示对象，函数，索引数据，类和各种属性，并且interface、类之间还可以互相extends

2.type aliases: 类型别名只是为一个类型增加了一个名字，没有创立新的类型。它也可以支持泛型，并且支持 interface 的几乎所有功能，除了 interface 在声明之后可以继续向里面加属性，而类型别名一旦声明就不能往里面加属性了。

3.generics: 软件工程里面有一个词是可重用性，泛型就是这种以类型为变量，方便用户创建多种类型的东西。
