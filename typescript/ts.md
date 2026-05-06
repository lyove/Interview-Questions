前端-typescript

前端面试：Typescript部分

1. 接口（Interface）与类型别名（Type）
Interface
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

Type
type Animal = {
  name: string;
};
// 联合类型
type Status = 'pending' | 'success' | 'error';
// 元组
type Point = [number, number];




2. 如何表示联合类型：
使用 | 表示联合类型

3. 什么是类型断言？如何使用？
使用 <Type> 或 as Type

4. 接口（interface）和类型别名（type）的区别？
特性 	            interface 	      type
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
可扩展性        |	  支持声明合并    |     不支持
支持继承        |	  支持 extends    |     支持交叉类型 &
支持基本类型 |	  ❌	            |     ✅（如 `type ID = string`）
使用场景        |   对象结构建模   |     复杂类型组合


5. 元组（Tuple）的作用是什么？如何定义？
let user: [string, number];

6. 类型推断？
TypeScript 会根据赋值自动推断类型


7. 类型守卫
就是一种告诉 TypeScript “在这个 if 语句/函数内部，这个变量一定是某个特定类型” 的方式。
是一种在运行时检查变量类型的技术，它允许你在特定代码块中缩小（narrow） 一个变量的类型
typeof类型守卫、instanceof


8. any和unknown的区别？
any：绕过所有类型检查。
unknown：必须先做类型检查后才能操作。
