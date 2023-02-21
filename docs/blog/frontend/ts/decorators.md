# 装饰器

它是一种特殊类型的声明，它能够被附加到类声明，方法，属性或参数上，可以修改类的行为。 通俗的讲装饰器就是一个函数/方法，可以注入到类、方法、属性参数上来扩展类、属性、方法、参数的功能。 常见的装饰器有：

- 类装饰器
- 属性装饰器
- 方法装饰器
- 参数装饰器

装饰器的写法：**普通装饰器(无法传参)、装饰器工厂(可传参)**

## 类装饰器

类装饰器在类声明之前被声明〈紧靠着类声明)。类装饰器应用于类构造函数，可以用来监视，修改或替换类定义。

### 普通装饰器

```ts
// target就是使用装饰器的那个类
const moveDecorator: ClassDecorator = (target: Function) => {
  target.prototype.name = "curry";
};

const musicDecorator: ClassDecorator = (target: Function) => {
  target.prototype.playMusic = () => {
    console.log("playMusic");
  };
};

@moveDecorator
@musicDecorator
class Tank {}
const t = new Tank()(t as any)
  .name(
    // "curry"
    t as any
  )
  .playMusic();

// 语法糖写法
moveDecorator(Tank);
```

### 装饰器工厂

```ts
const MusicDecoratorFactory = (type: string): ClassDecorator => {
  switch (type) {
    case "Tank":
      return (target: Function) => {
        target.prototype.playMusic = (): void => {
          console.log("勇敢的心");
        };
      };
    case "Player":
      return (target: Function) => {
        target.prototype.playMusic = (): void => {
          console.log("喜羊羊");
        };
      };
    default:
      return (target: Function) => {
        target.prototype.playMusic = (): void => {
          console.log("playMusic");
        };
      };
  }
};

const MoveDecorator: ClassDecorator = (target: Function) => {
  target.prototype.getPosition = () => {
    return { x: 100, y: 100 };
  };
};

@MusicDecoratorFactory("Tank")
@MoveDecorator
class Tank {}

@MusicDecoratorFactory("Player")
@MoveDecorator
class Player {}

const t = new Tank()(t as any)
  .getPosition()(t as any)
  .playMusic();

const p = new Player()(p as any)
  .getPosition()(p as any)
  .playMusic();
```

## 方法装饰器

方法装饰器被应用到方法的属性描述符上，可以用来监视，修改或者替换方法定义。方法装饰会在运行时传入下列个参数:

- 装饰的实例。对于静态成员来说是类的构造函数，对于实例成员是类的原型对象
- 方法的名字
- 方法的属性描述符

### 普通装饰器

```ts
const showDecorator: MethodDecorator = (
  target: Object,
  propertyKey: string | symbol,
  descriptor: PropertyDescriptor
) => {
  descriptor.value = () => {
    console.log("james");
  };
};

class User {
  @showDecorator
  public show() {
    console.log("curry");
  }

  @showDecorator
  public static find() {
    console.log("stephen");
  }
}

User.find();
new User().show();
```

### 装饰器工厂

```ts
const SleepDecorator = (times: number): MethodDecorator => {
  return (
    target: Object,
    propertyKey: string | symbol,
    descriptor: PropertyDescriptor
  ) => {
    const method = descriptor.value;
    descriptor.value = () => {
      setTimeout(() => {
        method();
      }, times);
    };
  };
};

class User {
  @SleepDecorator(3000)
  public show() {
    console.log("curry");
  }
}

new User().show();
```

## 属性装饰器

属性装饰器表达式会在运行时当作函数被调用，传入下列 2 个参数:

- 装饰的实例。对于静态成员来说是类的构造函数，对于实例成员是类的原型对象
- 装饰的属性名

```ts
const PropsDecorator: PropertyDecorator = (...args: any[]) => {
  // 第一个参数对于静态属性是构造函数，对于普通属性是原型对象
  // 第二个参数是方法名称
  // 第三个参数是值
  console.log(args);
};

const ParamsDecorator: PropertyDecorator = (...args: any[]) => {
  // 第一个参数对于静态属性是构造函数，对于普通属性是原型对象
  // 第二个参数是方法名称
  // 第三个参数是参数位置
  console.log(args);
};

class User {
  @PropsDecorator
  public name: string | undefined;

  public show(id: number = 1, @ParamsDecorator content: string) {}
}
```
