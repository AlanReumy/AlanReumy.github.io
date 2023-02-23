# 协变和逆变

我们先来介绍一下子类型的概念，来看如下例子：

```ts
interface Animal {
  name: string;
}

interface Dog extends Animal {
  wang(): void;
}
```

对于上面这个例子来说，父类型是 `Animal`，子类型是 `Dog`。在类型系统中，属性更多的类型是子类型，在集合论中，属性更少的类型是子类型

总之记住一点，**子类型比父类型具体**

## assignable

在类型系统中，如果一个变量赋值给另外一个变量时，需要检查两个变量的类型是否可以相互赋值，这就是 `assignable` 可赋值性

```ts
let animal: Animal;
let dog: Dog;

animal = dog; // √
dog = animal; // ×
```

在上面这个例子中，`animal`是一个属性少的类型，它看起来更加宽泛，所以更具体的子类型是能够赋值给它的，因为 `animal` 变量上只有 `name` 这个属性，而如果将 `animal` 变量赋值给 `dog` 变量的时候，由于 `animal` 变量本身不具备 `say` 方法，所以会出错

因此，从可赋值性的角度上来说，父类型是能够赋值给子类型的，因为子类型本身覆盖了父类型的所有属性

## 回到概念

在维基百科上，对于协变和逆变的定义是这样的：

> 在一门程序设计语言的类型系统中，一个类型规则或者类型构造器是：
>
> - 协变（covariant），如果它保持了子类型序关系 ≦。该序关系是：子类型 ≦ 基类型。
> - 逆变（contravariant），如果它逆转了子类型序关系。
> - 不变（invariant），如果上述两种均不适用。

```ts
let testAnimal = (animal: Animal) => {
  animal.name;
};

let testDog = (dog: Dog) => {
  dog.wang();
};

let ani = { name: "ani" };

testAnimal = testDog; // ×
testAnimal(ani); // × 不存在 wang 方法
```

在最开始的例子中 `animal = dog` 是正确的，这里怎么不可以呢？因为对于 `testDog` 这个函数类型来说，它的参数类型期望是一个更加具体的类型

在这个情况下，**父子类型发生了转换，这其实就是逆变**

而实际情况下，由于灵活性等权衡，TS 对于函数参数默认的处理是 **双向协变** 的。也就是既可以 `testDog = testAnimal`，也可以 `testAnimal = testDog`。在开启了 `tsconfig` 中的 `strictFunctionType` 后才会严格按照 逆变 来约束赋值关系。
