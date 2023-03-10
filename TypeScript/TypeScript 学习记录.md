# TypeScript 学习记录

插件：[TypeScript Importer](https://link.juejin.cn/?target=https%3A%2F%2Fmarketplace.visualstudio.com%2Fitems%3FitemName%3Dpmneo.tsimporter) 、Move TS、ErrorLens

## 原始类型和对象类型

## 字面量类型和枚举类型

## 函数和 Class 的类型

## any 、unknown 、never 和类型断言

**类型断言**

在 TypeScript 类型分析不正确或不符合预期时，将其断言为此处的正确类型

## TypeScript 类型工具(上)

类型别名

联合类型和交叉类型

索引类型

索引签名类型

[key: string]: 类型

其中的[ ]属于索引签名类型

索引类型查询

keyof 关键字

可以将对象中的所有键转换为对应字面量类型，然后再组合成联合类型。

keyof 的产物一定是联合类型

索引类型访问

obj[类型]

`'propA'`和`'propB'`都是**字符串字面量类型**，**而不是一个 JavaScript 字符串值**。

通过键的字面量类型来访问对应这个键的键值的类型

```TypeScript
interface Foo {
    propA: number
    propB: string
}

type PropAType = Foo['propA']// number
type PropBType = Foo['propB']// string
```

映射类型

语法：in

```TypeScript
type Stringify<T> = {
    [K in keyof T]: string
}

interface Foo {
    prop1: string;
    prop2: number;
    prop3: boolean;
    prop4: () => void;
}

type StringifiedFoo = Stringify<Foo>;

// 等价于
interface _StringifiedFoo {
    prop1: string;
    prop2: string;
    prop3: string;
    prop4: string;
}

type Clone<T>={
    [K in keyof T]:T[K]
}
```

in 为映射类型，keyof 获取 T 的字面量类型组合的联合类型，K 存储了每个键的字面量类型，T[K] 可获取键对应的键值的类型

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=MGQ5OTIxMWU3ZTQ1MTNmYjVkY2U3MjdmNWVhZWUxMmVfbVNuclZxZ1VHYXFiaENFdEdGbFJLTmtna2F5bFoxdTdfVG9rZW46Ym94Y25sSWVCTG1Ua3ZwWWNhdnFGNGR0M1lmXzE2NzgyODE3OTE6MTY3ODI4NTM5MV9WNA)

## TypeScript 类型工具(下)

类型守卫

常用

```TypeScript
export type Falsy = false | "" | 0 | null | undefined;

export const isFalsy = (val: unknown): val is Falsy => !val;

// 不包括不常用的 symbol 和 bigint
export type Primitive = string | number | boolean | undefined;

export const isPrimitive = (val: unknown): val is Primitive => ['string', 'number', 'boolean' , 'undefined'].includes(typeof val);
```

## 泛型

```TypeScript
type Factory<T>=T | number | boolean
```

上面这个类型别名的本质就是一个函数，T 就是它的变量，返回值则是一个包含 T 的联合类型

## 关键字

infer

## 联合类型

type1 | type2 | type3

## 接口

接口不会编译为 js 代码，是 ts 的一部分

接口作为函数参数时，传的值不一定就为该接口类型，只要能满足大于这个接口要求即可

**可选属性**

**只读属性**

readonly 可以设置属性为只读

readonlyArray 可以设置只读的数组

**接口定义函数类型**

```TypeScript
interface SearchFn{
    (source:string,substring:string):boolean;
}
```

## 泛型

我们需要返回的变量类型与传入的变量类型一致，所以我们使用一个类型变量来存储类型

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=MGMwMDk1ODlmZmExM2UzNzE3NjhlMzQ0NTNlNmMwNjdfaDdlNUJiOXNaN3p6Rkd5bGxlWW1QYnVFMWV1c2RtMFhfVG9rZW46Ym94Y25kUks3NzJCSEU1U0l3SUFpZVFlSkEzXzE2NzgyODE3OTE6MTY3ODI4NTM5MV9WNA)

### **泛型变量**

```TypeScript
function loggingIdentity<T>(arg: T[]): T[] {
    console.log(arg.length);
    // Array has a .length, so no more error
    return arg;
}
```

泛型函数`loggingIdentity`，接收类型参数`T`和参数`arg`，它是个元素类型是`T`的数组，并返回元素类型是`T`的数组。 泛型变量当作类型的一部分进行使用

### **泛型类型**

**箭头函数的泛型表示**

```TypeScript
const foo = <T,>(x: T): T => x;
```

**原始函数**

```TypeScript
function identity<T>(arg: T): T {
    return arg;
}
let myIdentity: <T>(arg: T) => T = identity;
let myIdentity: {<T>(arg: T): T} = identity;

interface GenericIdentityFn {
    <T>(arg: T): T;
}
let myIdentity: GenericIdentityFn = identity;
```

**一点改动**

```TypeScript
interface GenericIdentityFn<T> {
    (arg: T): T;
}
function identity<T>(arg: T): T {
    return arg;
}
let myIdentity: GenericIdentityFn<number> = identity;
```

当我们使用`GenericIdentityFn`的时候，还得传入一个类型参数来指定泛型类型（这里是：`number`），锁定了之后代码里使用的类型。

### **泛型类**

泛型类看上去与泛型接口差不多。 泛型类使用（`<>`）括起泛型类型，跟在类名后面。

```TypeScript
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}
let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) {
    return x + y;
};
```

### **泛型约束**

创建一个接口来描述约束条件

使用 extends 关键字继承这个接口来实现约束

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=N2Y0ZGIzNDc3MTkxYzI4ZWQxNmMzOTZhNjMyYzZjMWRfNzFmckZleXgzWGtFNEhvY3NMaElMRzdJeEMzT25EUmVfVG9rZW46Ym94Y25RVWFSZURTNzRjNHVsUFJLdDR1ejZiXzE2NzgyODE3OTE6MTY3ODI4NTM5MV9WNA)

### **在泛型约束里使用类型参数**

你可以声明一个类型参数，且它被另一个类型参数所约束。 比如，现在我们想要用属性名从对象里获取这个属性。 并且我们想要确保这个属性存在于对象`obj`上，因此我们需要在这两个类型之间使用约束。

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=YTExYmU2ZTY0MDA2NTg0Y2YxMjI2NGQ1YWY2NWQwZWVfVVhjREFPYmM5UHcwRDkxYjF2OEttZ0M3Z3FDYVRvV3NfVG9rZW46Ym94Y242TzJCcWJ1U0dWbzNVOVJQSE5hZ2tkXzE2NzgyODE3OTE6MTY3ODI4NTM5MV9WNA)

### **在泛型约束中使用类类型**

```TypeScript
class BeeKeeper {
    hasMask: boolean;
}
class ZooKeeper {
    nametag: string;
}
class Animal {
    numLegs: number;
}
class Bee extends Animal {
    keeper: BeeKeeper;
}
class Lion extends Animal {
    keeper: ZooKeeper;
}
function createInstance<A extends Animal>(c: new () => A): A {
    return new c();
}
createInstance(Lion).keeper.nametag;  // typechecks!
createInstance(Bee).keeper.hasMask;   // typechecks!
```

## type 和 interface 的区别

> https://juejin.cn/post/6844903749501059085

都可以描述一个对象或函数

都可以拓展，但语法不一样

**总结**

用 interface 描述数据结构，用 type 描述类型关系

能用 interface 实现就用 interface，不能再用 type

## unknown 和 any 的区别

any 类型类似于纯 JavaScript 的工作方式。我们有时可能需要描述一个我们根本不知道类型的变量。

在 TypeScript 中，任何东西可以赋值给 any 。它通常被称为 top type 。但也可以把 any 赋给所有类型

any 和 unknown 的最大区别是, unknown 是 top type (任何类型都是它的 subtype) , 而 any 即是 top type, 又是 bottom type (它是任何类型的 subtype ) ,这导致 any 基本上就是放弃了任何类型检查

与 any 一样，所有类型都可以分配给 unknown。

但只能将 unknown 类型的变量赋值给 any 和 unknown。

unknown 类型要安全得多，因为它迫使我们执行额外的类型检查来对变量执行操作。

## 高级类型

**ReturnType**

会返回一个函数类型中返回值位置的类型

**Partial**

将一个对象定义的所有属性都变为可选属性

**Require**

将一个对象的所有属性变为必须属性

**Pick**

选择对象中的一些属性

**Omit**

删除对象中的一些属性
