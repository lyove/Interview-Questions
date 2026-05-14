# Typescript

## 一、接口（Interface）和类型（Type）
### 接口（Interface）
```ts
interface Animal {
  name: string;
}
interface Animal {        // ✅ 同名自动合并
  age: number;
}
interface Dog extends Animal {
  breed: string;
}
// Dog = { name: string; age: number; breed: string }
```

### 类型（Type）
```ts
type Animal = {
  name: string;
};
// 联合类型
type Status = 'pending' | 'success' | 'error';
// 元组
type Point = [number, number];
```

### 接口（interface）和类型别名（type）的区别？

| 特性 | interface | type |
| :--- | :--- | :--- |
| 可扩展性 | 支持声明合并，比如：<br />`interface Animal {name: string;}`和`interface Animal { age: number;}`同名会自动合并 | 不支持 |
| 支持继承 | 支持 extends | 支持交叉类型 &，例如：`type Dog = Animal & { breed: string; };` |
| 支持基本类型 | ❌	 | ✅（如 `type ID = string`） |
| 使用场景 | 对象结构建模 | 复杂类型组合 |


## 二、如何表示联合类型：
使用 | 表示联合类型

## 三、泛型
简单来说，泛型就像是“类型的变量”或“类型占位符”。它允许你在编写函数、接口或类时，不预先指定具体的类型，而是在使用时再确定类型。  
例如：
```jsx
// T 是一个类型变量，代表调用时传入的具体类型
function getFirst<T>(arr: T[]): T {
  return arr[0];
}

const num = getFirst([1, 2, 3]); // TypeScript 自动推断 T 为 number，返回值也是 number
const str = getFirst(['a', 'b']); // 自动推断 T 为 string，返回值也是 string
```
### 🛠️ 泛型的四大核心应用场景
1. 泛型函数
在函数名后使用 <T> 定义类型参数，它可以在参数、返回值或函数内部使用。
```jsx
function identity<T>(arg: T): T {
  return arg;
}
let output = identity<string>("hello"); // 显式指定 T 为 string
let output2 = identity(100); // 自动推断 T 为 number
```
具体含义  
```jsx
function identity<T>(arg: T): T {
//           ^^^^    ^^^    ^
//           声明    使用    使用
  return arg;
}
```

2. 泛型接口
当接口的某些属性或方法需要支持多种类型时，可以使用泛型接口。  
```jsx
interface ApiResponse<T> {
  code: number;
  message: string;
  data: T; // data 的类型由外部传入
}

const userRes: ApiResponse<{ id: number; name: string }> = {
  code: 200,
  message: "成功",
  data: { id: 1, name: "张三" }
};
```

3. 泛型类
泛型类可以确保类的实例方法在处理不同类型时保持类型一致。  
```jsx
class Box<T> {
  private content: T;
  constructor(initial: T) {
    this.content = initial;
  }
  get(): T {
    return this.content;
  }
}

const numberBox = new Box<number>(100);
// numberBox.set("hello"); // 报错！这里只能传入 number
```

4. 泛型类型别名
常用于定义一些通用的数据结构，比如树形结构或包装类型。  
```jsx
type Wrapped<T> = { value: T };
const wrappedNum: Wrapped<number> = { value: 42 };
```


## 四、什么是类型断言？如何使用？
使用「尖括号」`<类型>` 或 as `as 类型`。
TypeScript 类型断言（Type Assertion）是一种告诉编译器"我知道这个值的具体类型，请相信我"的机制。它不会进行任何运行时转换，仅在编译时移除类型检查。

```ts
// 尖括号语法（不推荐在 JSX 中使用，会与标签语法冲突）
let someValue: any = "this is a string";
let strLength: number = (<string>someValue).length;

// as 语法（推荐，通用性更好）
let strLength2: number = (someValue as string).length;
```
### 常见使用场景
1. 将宽泛类型收窄为具体类型
```ts
function getData(): any {
  return { name: "Tom", age: 25 };
}

const data = getData();
// data 是 any，但我们知道它应该是 User
const user = data as { name: string; age: number };
console.log(user.name); // OK
```

2. DOM 元素类型断言
```ts
const input = document.getElementById("username") as HTMLInputElement;
console.log(input.value); // 如果不断言，input 是 HTMLElement，没有 value 属性

const canvas = document.querySelector("canvas") as HTMLCanvasElement;
```

3. 处理联合类型
```ts
type Shape = 
  |{ kind: "circle"; radius: number }
  |{ kind: "square"; side: number };

function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    // TypeScript 已自动收窄，无需断言
    return Math.PI * shape.radius ** 2;
  }
  
  // 这里 shape 被推断为 square，但如果逻辑复杂可能需要断言
  return (shape as { kind: "square"; side: number }).side ** 2;
}
```

4. 非空断言（与类型断言相关）
```ts
// ! 是非空断言运算符
const el = document.getElementById("app")!; // 告诉 TS：这不会是 null
el.innerHTML = "Hello"; // 无需检查 null
```

### 双重断言（更激进的断言）
当类型之间完全不兼容时，普通断言会报错，需要先断言为 unknown，再断言为目标类型： 
```ts
type Cat = { meow: () => void };
type Dog = { bark: () => void };

const cat: Cat = { meow: () => {} };

// ❌ 直接断言报错：类型不兼容
// const dog: Dog = cat as Dog;

// ✅ 双重断言（绕过类型检查，风险自负）
const dog: Dog = cat as unknown as Dog;
dog.bark(); // 运行时错误！cat 没有 bark 方法
```


## 五、元组（Tuple）的作用是什么？如何定义？
`let user: [string, number];`

## 六、类型推断？
TypeScript 会根据赋值自动推断类型


## 七、类型守卫
TypeScript 类型守卫（Type Guard）是一种在运行时检查值的类型，并据此在代码块内收窄（Narrow）类型范围的机制。与类型断言"强制告诉编译器"不同，类型守卫是让编译器根据逻辑自动推断出更精确的类型。
示例：
```ts
function process(value: string | number) {
  // 此时 value 是联合类型 string | number
  value.toFixed(); // ❌ 报错：string 没有 toFixed
  
  if (typeof value === "number") {
    // ✅ 在这个代码块内，value 被自动收窄为 number
    value.toFixed(2);
  } else {
    // ✅ 这里 value 被推断为 string
    value.toUpperCase();
  }
}
```
### 四种类型守卫方式
1. typeof 类型守卫. 
适用于基础类型（string、number、boolean、bigint、symbol、undefined、object、function）。
```ts
function printValue(value: string | number | boolean) {
  if (typeof value === "string") {
    console.log(value.toUpperCase()); // value: string
  } else if (typeof value === "number") {
    console.log(value.toFixed(2));    // value: number
  } else {
    console.log(value.valueOf());     // value: boolean
  }
}
```
⚠️ **typeof null === "object" 是 JavaScript 历史 bug，需额外注意**。


2. instanceof 类型守卫
适用于类（Class）的实例检查。
```ts
class Dog {
  bark() { return "Woof!"; }
}

class Cat {
  meow() { return "Meow!"; }
}

function makeSound(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    console.log(animal.bark()); // animal: Dog
  } else {
    console.log(animal.meow()); // animal: Cat
  }
}
```

3. in 类型守卫
通过检查**属性是否存在**来区分类型。  
```ts
type Circle = { kind: "circle"; radius: number };
type Square = { kind: "square"; side: number };
type Rectangle = { kind: "rect"; width: number; height: number };

type Shape = Circle | Square | Rectangle;

function getArea(shape: Shape) {
  if ("radius" in shape) {
    return Math.PI * shape.radius ** 2; // shape: Circle
  } else if ("side" in shape) {
    return shape.side ** 2;             // shape: Square
  } else {
    return shape.width * shape.height;  // shape: Rectangle
  }
}
```
💡 **比 typeof 更适合检查对象结构，但需注意属性名冲突。**

4. 自定义类型守卫（最强大）
使用 value is Type 谓词，让函数"返回一个类型判断"。  
```ts
// 定义类型守卫函数
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function isNumber(value: unknown): value is number {
  return typeof value === "number" && !isNaN(value);
}

// 使用
function process(value: unknown) {
  if (isString(value)) {
    console.log(value.toUpperCase()); // ✅ value: string
  } else if (isNumber(value)) {
    console.log(value.toFixed(2));    // ✅ value: number
  }
}
```

## 八、any和unknown的区别？
`any`：绕过所有类型检查。
`unknown`：必须先做类型检查后才能操作。

## readonly
```jsx
const person: { readonly name: string } = { name: "Alice" };
person.name = "Bob"; // ❌ 报错：无法分配到只读属性
```
