## TypeScript Tips

### 可选属性

带有可选属性的接口与普通的接口定义差不多，只是在可选属性名字定义的后面加 一个 ? 符号。

可选属性的好处之一是可以对可能存在的属性进行预定义，好处之二是可以捕获引 用了不存在的属性时的错误。

```javascript
interface SquareConfig {
  color?: string;
  width?: number;
}
```

### 只读属性

一些对象属性只能在对象刚刚创建的时候修改其值。 你可以在属性名前
用 readonly 来指定只读属性:

```javascript
interface Point {
  readonly x: number;
  readonly y: number;
}
```

你可以通过赋值一个对象字面量来构造一个 Point 。 赋值后， x 和 y 再也不能 被改变了。

```javascript
let p1: Point = { x: 10, y: 20 };
p1.x = 5; // error!
```

TypeScript具有 ReadonlyArray<T> 类型，它与 Array<T> 相似，只是把所有可 变方法去掉了，因此可以确保数组创建后再也不能被修改:

```javascript
let a: number[] = [1, 2, 3, 4];
let ro: ReadonlyArray<number> = a;
ro[0] = 12; // error!
ro.push(5); // error!
ro.length = 100; // error!
a = ro; // error!
```

上面代码的最后一行，可以看到就算把整个 ReadonlyArray 赋值到一个普通数组 也是不可以的。 但是你可以用类型断言重写:

```javascript
a = ro as number[];
```

**readonly vs const**

最简单判断该用 readonly 还是 的方法是看要把它做为变量使用还是做为
一个属性。 做为变量使用的话用 ，若做为属性则使用 readonly 。

### 连续添加属性

有些人可能会因为代码美观性而喜欢先创建一个对象然后立即添加属性:

```javascript
var options = {};
options.color = "red";
options.volume = 11;
```

TypeScript会提示你不能给 color 和 volumn 赋值，因为先前指定 options 的 类型为 {} 并不带有任何属性。 如果你将声明变成对象字面量的形式将不会产生 错误:

```javascript
let options = {
  color: "red",
  volume: 11
};
```

你还可以定义 options 的类型并且添加类型断言到对象字面量上。

```javascript
interface Options { color: string; volume: number }
let options = {} as Options;
options.color = "red";
options.volume = 11;
```

### 类型断言
有时候你会遇到这样的情况，你会比TypeScript更了解某个值的详细信息。 通常这 会发生在你清楚地知道一个实体具有比它现有类型更确切的类型。

通过类型断言这种方式可以告诉编译器，“相信我，我知道自己在干什么”。 类型断 言好比其它语言里的类型转换，但是不进行特殊的数据检查和解构。 它没有运行时 的影响，只是在编译阶段起作用。 TypeScript会假设你，程序员，已经进行了必须 的检查。

类型断言有两种形式。 其一是“尖括号”语法:

```javascript
let someValue: any = "this is a string";
let strLength: number = (<string>someValue).length;
```

另一个为` as `语法:

```javascript
let someValue: any = "this is a string";
let strLength: number = (someValue as string).length;
```

两种形式是等价的。 至于使用哪个大多数情况下是凭个人喜好;然而，当你在 TypeScript里使用JSX时，只有 as 语法断言是被允许的。

### 可索引的类型

与使用接口描述函数类型差不多，我们也可以描述那些能够“通过索引得到”的类 型，比如 a[10] 或 ageMap["daniel"] 。 可索引类型具有一个索引签名，它描 述了对象索引的类型，还有相应的索引返回值类型。 让我们看一个例子:

```javascript
interface StringArray {
  [index: number]: string;
  length: number; // 可以，length是number类型
  name: string // 错误，`name`的类型与索引类型返回值的类型不匹配
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];
let myStr: string = myArray[0];
```

最后，你可以将索引签名设置为只读，这样就防止了给索引赋值:

```javascript
interface ReadonlyStringArray {
  readonly [index: number]: string;
}
let myArray: ReadonlyStringArray = ["Alice", "Bob"];
myArray[2] = "Mallory"; // error!
```

### 联合类型(Union Types)

联合类型与交叉类型很有关联，但是使用上却完全不同。 偶尔你会遇到这种情况，
一个代码库希望传入 number 或 string 类型的参数。 例如下面的函数:

```javascript
function padLeft(value: string, padding: string | number) {
    // ...
}
let indentedString = padLeft("Hello world", true); // errors dur
ing compilation
```

### 类型守卫和类型断言

如果编译器不能够去除`null`或`undefined`，你可以使用类型断言手动去除。语法是添加`!`后缀: `identifier!`从`identifier `的类型里去除了`null`和`undefined `:

```javascript
function broken(name: string | null): string {
  function postfix(epithet: string) {
    return name.charAt(0) + '.  the ' + epithet; // error, 'name' is possibly null
  }
  name = name || "Bob";
  return postfix("great");
}
function fixed(name: string | null): string {
  function postfix(epithet: string) {
    return name!.charAt(0) + '.  the ' + epithet; // ok
  }
  name = name || "Bob";
  return postfix("great");
}
```

### 类型别名

类型别名会给一个类型起个新名字。 类型别名有时和接口很像，但是可以作用于原 始值，联合类型，元组以及其它任何你需要手写的类型。

```javascript
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;
function getName(n: NameOrResolver): Name {
  if (typeof n === 'string') {
    return n;
  } else {
    return n(); 
  }
}
```

起别名不会新建一个类型 - 它创建了一个新名字来引用那个类型。 给原始类型起别 名通常没什么用，尽管可以做为文档的一种形式使用。
同接口一样，类型别名也可以是泛型 - 我们可以添加类型参数并且在别名声明的右 侧传入:

```javascript
type Container<T> = { value: T };
```

### **重要:** 索引类型(Index types) 

使用索引类型，编译器就能够检查使用了动态属性名的代码。 例如，一个常见的
JavaScript模式是从对象中选取属性的子集。

```javascript
function pluck(o, names) {
  return names.map(n => o[n]);
}
```

下面是如何在TypeScript里使用此函数，通过索引类型查询和索引访问操作符:

```javascript
function pluck<T, K extends keyof T>(o: T, names: K[]): T[K][] {
  return names.map(n => o[n]);
}
interface Person {
  name: string;
  age: number;
}
let person: Person = {
  name: 'Jarid',
  age: 35
};
let strings: string[] = pluck(person, ['name']); // ok, string[]
```

### 映射类型

一个常见的任务是将一个已知的类型每个属性都变为可选的:

```javascript
interface PersonPartial {
  name?: string;
  age?: number;
}
```

或者我们想要一个只读版本:

```javascript
interface PersonReadonly {
  readonly name: string;
  readonly age: number;
}
```

这在JavaScript里经常出现，TypeScript提供了从旧类型中创建新类型的一种方式 — 映射类型。 在映射类型里，新类型以相同的形式去转换旧类型里每个属性。 例 如，你可以令每个属性成为 readonly 类型或可选的。 下面是一些例子:

```javascript
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
}
type Partial<T> = {
  [P in keyof T]?: T[P];
}
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
}
type Record<K extends keyof any, T> = {
  [P in K]: T;
}
```

像下面这样使用:

```javascript
type PersonPartial = Partial<Person>;
type ReadonlyPerson = Readonly<Person>;
```

需要注意的是这个语法描述的是类型而非成员。 若想添加额外的成员，则可以使用 交叉类型:

```javascript
// 这样使用
type PartialWithNewMember<T> = {
  [P in keyof T]?: T[P];
} & { newMember: boolean }
// 不要这样使用
// 这会报错!
type PartialWithNewMember<T> = {
  [P in keyof T]?: T[P];
  newMember: boolean;
}
```

### 预定义的有条件类型

TypeScript 2.8在 lib.d.ts 里增加了一些预定义的有条件类型:

* `Exclude<T, U>` -- 从`T`中剔除可以赋值给`U`的类型。
* `Extract<T, U>` -- 提取`T`中可以赋值给`U`的类型。
* `NonNullable<T>` --从`T`中剔除`null`和`undefined`。
* `ReturnType<T>` -- 获取函数返回值类型。
* `InstanceType<T>` -- 获取构造函数类型的实例类型。

Example

```javascript
type T00 = Exclude<"a" | "b" | "c" | "d", "a" | "c" | "f">;  //
"b" | "d"
type T01 = Extract<"a" | "b" | "c" | "d", "a" | "c" | "f">;  //
"a" | "c"
type T02 = Exclude<string | number | (() => void), Function>;  /
/ string | number
type T03 = Extract<string | number | (() => void), Function>;  /
/ () => void
type T04 = NonNullable<string | number | undefined>;  // string
| number
type T05 = NonNullable<(() => string) | string[] | null | undefi
ned>;  // (() => string) | string[]

function f1(s: string) {
  return { a: 1, b: s };
}
class C {
  x = 0;
  y = 0;
}

type T10 = ReturnType<() => string>;  // string
type T11 = ReturnType<(s: string) => void>;  // void
type T12 = ReturnType<(<T>() => T)>;  // {}
type T13 = ReturnType<(<T extends U, U extends number[]>() => T)
>;  // number[]
type T14 = ReturnType<typeof f1>;  // { a: number, b: string }
type T15 = ReturnType<any>;  // any
type T16 = ReturnType<never>;  // never
type T17 = ReturnType<string>;  // Error
type T18 = ReturnType<Function>;  // Error
type T20 = InstanceType<typeof C>;  // C
type T21 = InstanceType<any>;  // any
type T22 = InstanceType<never>;  // never
type T23 = InstanceType<string>;  // Error
type T24 = InstanceType<Function>;  // Error
```

### Typescript derive union type from tuple/array values


#### Typescript 3.0

It looks like, starting with TypeScript 3.0, it will be possible for TypeScript to automatically infer tuple types. Once is released, the tuple() function you need can be succinctly written as:

```javascript
export type Lit = string | number | boolean | undefined | null | void | {};
export const tuple = <T extends Lit[]>(...args: T) => args;
```

And then you can use it like this:

```javascript
const list = tuple('a','b','c');  // type is ['a','b','c']
type NeededUnionType = typeof list[number]; // 'a'|'b'|'c'
```

#### Typescript 3.4

```javascript
const list = ['a', 'b', 'c'] as const; // TS3.4 syntax
type NeededUnionType = typeof list[number]; // 'a'|'b'|'c';
```