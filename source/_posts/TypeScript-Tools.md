---
title: TS 工具类型，开发事半功倍
date: 2020-03-21 08:46:52
tags: ['ts','js']
categories: 前端
keywords: ['ts','js']
top_img: https://goss1.cfp.cn/creative/vcg/veer/400/new/VCG41N680280554.jpg?x-oss-process=image/format,webp
cover : https://goss1.cfp.cn/creative/vcg/nowater800/new/1d6e8b76d174484fa491959c72623178.jpg?x-oss-process=image/format,webp
---
 ## TS 工具类型，开发事半功倍

  ### 一、类型别名 

  TypeScript 提供了为类型注解设置别名的便捷语法，你可以使用 `type SomeName = someValidTypeAnnotation` 来创建别名，比如：

  ```
  type Pet = 'cat' | 'dog';
  let pet: Pet;
  
  pet = 'cat'; // Ok
  pet = 'dog'; // Ok
  pet = 'zebra'; // Compiler error
  ```

  ### 二、基础知识

  为了让大家能更好地理解并掌握 TypeScript 内置类型别名，我们先来介绍一下相关的一些基础知识。

  #### 2.1 typeof

  在 TypeScript 中，`typeof` 操作符可以用来获取一个变量声明或对象的类型。

  ```
  interface Person {
    name: string;
    age: number;
  }
  
  const sem: Person = { name: 'semlinker', age: 30 };
  type Sem= typeof sem; // -> Person
  
  function toArray(x: number): Array<number> {
    return [x];
  }
  
  type Func = typeof toArray; // -> (x: number) => number[]
  ```


  #### 2.2 keyof

  `keyof` 操作符可以用来一个对象中的所有 key 值：

  ```
  interface Person {
      name: string;
      age: number;
  }
  
  type K1 = keyof Person; // "name" | "age"
  type K2 = keyof Person[]; // "length" | "toString" | "pop" | "push" | "concat" | "join"
  type K3 = keyof { [x: string]: Person };  // string | number
  ```


  #### 2.3 in

  `in` 用来遍历枚举类型：

  ```
  type Keys = "a" | "b" | "c"
  
  type Obj =  {
    [p in Keys]: any
  } // -> { a: any, b: any, c: any }
  ```



  #### 2.4 infer

  在条件类型语句中，可以用 `infer` 声明一个类型变量并且对它进行使用。

  ```
  type ReturnType<T> = T extends (
    ...args: any[]
  ) => infer R ? R : any;
  ```

  以上代码中 `infer R` 就是声明一个变量来承载传入函数签名的返回值类型，简单说就是用它取到函数返回值的类型方便之后使用。

  

  #### 2.5 extends

  有时候我们定义的泛型不想过于灵活或者说想继承某些类等，可以通过 extends 关键字添加泛型约束。

  ```
  interface ILengthwise {
    length: number;
  }
  
  function loggingIdentity<T extends ILengthwise>(arg: T): T {
    console.log(arg.length);
    return arg;
  }
  ```

  现在这个泛型函数被定义了约束，因此它不再是适用于任意类型：

  ```
  loggingIdentity(3);  // Error, number doesn't have a .length property
  ```

  这时我们需要传入符合约束类型的值，必须包含必须的属性：

  ```
  loggingIdentity({length: 10, value: 3});
  ```

  ### 三、内置类型别名

  #### 3.1 Partial

  `Partial` 的作用就是将某个类型里的属性全部变为可选项 `?`。

  **定义：**

  ```
  // node_modules/typescript/lib/lib.es5.d.ts
  
  /**
   * Make all properties in T optional
   */
  type Partial<T> = {
      [P in keyof T]?: T[P];
  };
  ```

  在以上代码中，首先通过 `keyof T` 拿到 `T` 的所有属性名，然后使用 `in` 进行遍历，将值赋给 `P`，最后通过 `T[P]` 取得相应的属性值。中间的 `?`，用于将所有属性变为可选。

  **示例：**

  ```
  interface Todo {
    title: string;
    description: string;
  }
  
  function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>) {
    return { ...todo, ...fieldsToUpdate };
  }
  
  const todo1 = {
    title: "organize desk",
    description: "clear clutter"
  };
  
  const todo2 = updateTodo(todo1, {
    description: "throw out trash"
  });
  ```

  #### 3.2 Required

  `Required` 的作用就是将某个类型里的属性全部变为必选项。

  **定义：**

  ```
  // node_modules/typescript/lib/lib.es5.d.ts
  
  /**
   * Make all properties in T required
   */
  type Required<T> = {
      [P in keyof T]-?: T[P];
  };
  ```

  以上代码中，`-?` 的作用就是移除可选项 `?`。

  **示例：**

  ```
  interface Props {
    a?: number;
    b?: string;
  }
  
  const obj: Props = { a: 5 }; // OK
  const obj2: Required<Props> = { a: 5 }; // Error: property 'b' missing
  ```

  #### 3.3 Readonly

  `Readonly` 的作用是将某个类型所有属性变为只读属性，也就意味着这些属性不能被重新赋值。

  **定义：**

  ```
  // node_modules/typescript/lib/lib.es5.d.ts
  
  /**
   * Make all properties in T readonly
   */
  type Readonly<T> = {
      readonly [P in keyof T]: T[P];
  };
  ```

  如果将上面的 `readonly` 改成 `-readonly`， 就是移除子属性的 `readonly` 标识。

  **示例：**

  ```
  interface Todo {
    title: string;
  }
  
  const todo: Readonly<Todo> = {
    title: "Delete inactive users"
  };
  
  todo.title = "Hello"; // Error: cannot reassign a readonly property
  ```

  `Readonly` 对于表示在运行时将赋值失败的表达式很有用（比如，当尝试重新赋值冻结对象的属性时）。

  ```
  function freeze<T>(obj: T): Readonly<T>;
  ```

  #### 3.4 Record

  `Record` 的作用是将 `K` 中所有的属性的值转化为 `T`类型。

  **定义：**

  ```
  // node_modules/typescript/lib/lib.es5.d.ts
  
  /**
   * Construct a type with a set of properties K of type T
   */
  type Record<K extends keyof any, T> = {
      [P in K]: T;
  };
  ```

  **示例：**

  ```
  interface PageInfo {
    title: string;
  }
  
  type Page = "home" | "about" | "contact";
  
  const x: Record<Page, PageInfo> = {
    about: { title: "about" },
    contact: { title: "contact" },
    home: { title: "home" }
  };
  ```

  #### 3.5 Pick

  `Pick` 的作用是将某个类型中的子属性挑出来，变成包含这个类型部分属性的子类型。

  **定义：**

  ```
  // node_modules/typescript/lib/lib.es5.d.ts
  
  /**
   * From T, pick a set of properties whose keys are in the union K
   */
  type Pick<T, K extends keyof T> = {
      [P in K]: T[P];
  };
  ```

  **示例：**

  ```
  interface Todo {
    title: string;
    description: string;
    completed: boolean;
  }
  
  type TodoPreview = Pick<Todo, "title" | "completed">;
  
  const todo: TodoPreview = {
    title: "Clean room",
    completed: false
  };
  ```

  #### 3.6 Exclude

  `Exclude` 的作用是将某个类型中属于另一个的类型移除掉。

  **定义：**

  ```
  // node_modules/typescript/lib/lib.es5.d.ts
  
  /**
   * Exclude from T those types that are assignable to U
   */
  type Exclude<T, U> = T extends U ? never : T;
  ```

  如果 `T` 能赋值给 `U` 类型的话，那么就会返回 `never` 类型，否则返回 `T` 类型。最终实现的效果就是将 `T` 中某些属于 `U` 的类型移除掉。

  **示例：**

  ```
  type T0 = Exclude<"a" | "b" | "c", "a">; // "b" | "c"
  type T1 = Exclude<"a" | "b" | "c", "a" | "b">; // "c"
  type T2 = Exclude<string | number | (() => void), Function>; // string | number
  ```

  #### 3.7 Extract

  `Extract` 的作用是从 `T` 中提取出 `U`。

  **定义：**

  ```
  // node_modules/typescript/lib/lib.es5.d.ts
  
  /**
   * Extract from T those types that are assignable to U
   */
  type Extract<T, U> = T extends U ? T : never;
  ```

  如果 `T` 能赋值给 `U` 类型的话，那么就会返回 `T` 类型，否则返回 `never` 类型。

  **示例：**

  ```
  type T0 = Extract<"a" | "b" | "c", "a" | "f">; // "a"
  type T1 = Extract<string | number | (() => void), Function>; // () => void
  ```

  #### 3.8 Omit

  `Omit` 的作用是使用 `T` 类型中除了 `K` 类型的所有属性，来构造一个新的类型。

  **定义：**

  ```
  // node_modules/typescript/lib/lib.es5.d.ts
  
  /**
   * Construct a type with the properties of T except for those in type K.
   */
  type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
  ```

  **示例：**

  ```
  interface Todo {
    title: string;
    description: string;
    completed: boolean;
  }
  
  type TodoPreview = Omit<Todo, "description">;
  
  const todo: TodoPreview = {
    title: "Clean room",
    completed: false
  };
  ```

  #### 3.9 NonNullable

  `NonNullable` 的作用是用来过滤类型中的 `null` 及 `undefined` 类型。

  **定义：**

  ```
  // node_modules/typescript/lib/lib.es5.d.ts
  
  /**
   * Exclude null and undefined from T
   */
  type NonNullable<T> = T extends null | undefined ? never : T;
  ```

  **示例：**

  ```
  type T0 = NonNullable<string | number | undefined>; // string | number
  type T1 = NonNullable<string[] | null | undefined>; // string[]
  ```

  #### 3.10 ReturnType

  `ReturnType` 的作用是用于获取函数 `T` 的返回类型。

  **定义：**

  ```
  // node_modules/typescript/lib/lib.es5.d.ts
  
  /**
   * Obtain the return type of a function type
   */
  type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
  ```

  **示例：**

  ```
  type T0 = ReturnType<() => string>; // string
  type T1 = ReturnType<(s: string) => void>; // void
  type T2 = ReturnType<<T>() => T>; // {}
  type T3 = ReturnType<<T extends U, U extends number[]>() => T>; // number[]
  type T4 = ReturnType<any>; // any
  type T5 = ReturnType<never>; // any
  type T6 = ReturnType<string>; // Error
  type T7 = ReturnType<Function>; // Error
  ```


  #### 3.11 InstanceType

  `InstanceType` 的作用是获取构造函数类型的实例类型。

  **定义：**

  ```
  // node_modules/typescript/lib/lib.es5.d.ts
  
  /**
   * Obtain the return type of a constructor function type
   */
  type InstanceType<T extends new (...args: any) => any> = T extends new (...args: any) => infer R ? R : any;
  ```

  **示例：**

  ```
  class C {
    x = 0;
    y = 0;
  }
  
  type T0 = InstanceType<typeof C>; // C
  type T1 = InstanceType<any>; // any
  type T2 = InstanceType<never>; // any
  type T3 = InstanceType<string>; // Error
  type T4 = InstanceType<Function>; // Error
  ```

  #### 3.12 ThisType

  `ThisType` 的作用是用于指定上下文对象的类型。

  **定义：**

  ```
  // node_modules/typescript/lib/lib.es5.d.ts
  
  /**
   * Marker for contextual 'this' type
   */
  interface ThisType<T> { }
  ```

  > 注意：使用 `ThisType` 时，必须确保 `--noImplicitThis` 标志设置为 true。

  **示例：**

  ```
  interface Person {
      name: string;
      age: number;
  }
  
  const obj: ThisType<Person> = {
    dosth() {
      this.name // string
    }
  }
  ```


  #### 3.13 Parameters

  `Parameters` 的作用是用于获得函数的参数类型组成的元组类型。

  **定义：**

  ```
  // node_modules/typescript/lib/lib.es5.d.ts
  
  /**
   * Obtain the parameters of a function type in a tuple
   */
  type Parameters<T extends (...args: any) => any> = T extends (...args: infer P) => any 
    ? P : never;
  ```

  **示例：**

  ```
  type A = Parameters<() => void>; // []
  type B = Parameters<typeof Array.isArray>; // [any]
  type C = Parameters<typeof parseInt>; // [string, (number | undefined)?]
  type D = Parameters<typeof Math.max>; // number[]
  ```

  #### 3.14 ConstructorParameters

  `ConstructorParameters` 的作用是提取构造函数类型的所有参数类型。它会生成具有所有参数类型的元组类型（如果 T 不是函数，则返回的是 never 类型）。

  **定义：**

  ```
  // node_modules/typescript/lib/lib.es5.d.ts
  
  /**
   * Obtain the parameters of a constructor function type in a tuple
   */
  type ConstructorParameters<T extends new (...args: any) => any> = T extends new (...args: infer P) => any ? P : never;
  ```

  **示例：**

  ```
  type A = ConstructorParameters<ErrorConstructor>; // [(string | undefined)?]
  type B = ConstructorParameters<FunctionConstructor>; // string[]
  type C = ConstructorParameters<RegExpConstructor>; // [string, (string | undefined)?]
  ```

  ### 四、参考资源

  - TypeScript 强大的类型别名
  - typescript-3-7-utility-types-printable-pdf-cheat-sheet
  - 深入理解 TypeScript