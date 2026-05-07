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

## 三、什么是类型断言？如何使用？
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
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };

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


## 四、元组（Tuple）的作用是什么？如何定义？
`let user: [string, number];`

## 五、类型推断？
TypeScript 会根据赋值自动推断类型


## 六、类型守卫
就是一种告诉 TypeScript “在这个 if 语句/函数内部，这个变量一定是某个特定类型” 的方式。
是一种在运行时检查变量类型的技术，它允许你在特定代码块中缩小（narrow） 一个变量的类型
`typeof`类型守卫、`instanceof`


## 七、any和unknown的区别？
`any`：绕过所有类型检查。
`unknown`：必须先做类型检查后才能操作。
