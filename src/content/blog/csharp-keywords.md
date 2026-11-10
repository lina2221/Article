---
title: 'C# 知识点：预处理指令'
description: '整理 C# 常用关键字与修饰符：访问修饰符、类型与成员定义、流程控制、异常处理、类型操作、参数与方法、命名空间与 using、其他关键字。'
pubDate: 2026-11-10
---

> 本文整理 C# 常用关键字与修饰符，按访问修饰符、类型与成员定义、流程控制、异常处理、类型操作、参数与方法、命名空间与 using、其他关键字分类说明。（整理自语雀文档）

## 一、访问修饰符（Access Modifiers）

控制类型或成员的访问权限：

- `public`：完全公开，任何代码可访问。

  ```csharp
  public class MyClass { public int Value; }
  ```

- `private`：仅当前类内部可访问（默认访问修饰符）。

  ```csharp
  class MyClass { private int secret; }
  ```

- `protected`：当前类及派生类可访问。

  ```csharp
  class Parent { protected int data; }
  class Child : Parent { void UseData() { data = 1; } }
  ```

- `internal`：同一程序集（项目）内可访问。

  ```csharp
  internal class AssemblyInternal { }
  ```

- `protected internal`：同一程序集或派生类可访问（组合权限）。
- `private protected`：仅当前程序集内的派生类可访问。

## 二、类型与成员定义

### 1. 类型声明

- `class`：定义类（引用类型）。

  ```csharp
  public class Person { }
  ```

- `struct`：定义结构（值类型）。

  ```csharp
  public struct Point { public int X; public int Y; }
  ```

- `interface`：定义接口（抽象方法集合）。

  ```csharp
  public interface IFly { void Fly(); }
  ```

- `enum`：定义枚举（命名常量集合）。

  ```csharp
  public enum Color { Red, Green, Blue }
  ```

- `record`：定义记录类型（用于存储数据，默认实现值相等性）。

  ```csharp
  public record User(string Name, int Age);
  ```

- `record struct`：值类型的记录。

  ```csharp
  public record struct Vector(int X, int Y);
  ```

### 2. 成员定义

- `static`：声明静态成员（属于类型而非实例）。

  ```csharp
  public class MathUtil { public static int Add(int a, int b) => a + b; }
  ```

- `abstract`：声明抽象成员（无实现，需派生类重写）。

  ```csharp
  public abstract class Shape { public abstract double Area(); }
  ```

- `virtual`：声明虚方法（允许派生类重写）。

  ```csharp
  public class Animal { public virtual void Speak() => Console.WriteLine("..."); }
  ```

- `override`：重写基类的虚方法或抽象方法。

  ```csharp
  public class Dog : Animal { public override void Speak() => Console.WriteLine("Woof"); }
  ```

- `sealed`：密封类或方法（阻止派生或重写）。

  ```csharp
  public sealed class FinalClass { } // 不能被继承
  ```

- `readonly`：声明只读字段（只能在初始化或构造函数中赋值）。

  ```csharp
  public class Car { private readonly string model; }
  ```

- `const`：声明常量（编译时确定的值，必须初始化）。

  ```csharp
  public const int MaxSize = 100;
  ```

- `event`：声明事件（用于发布 - 订阅模式）。

  ```csharp
  public event Action OnCompleted;
  ```

- `delegate`：声明委托（方法类型的引用）。

  ```csharp
  public delegate int Calculator(int a, int b);
  ```

- `partial`：拆分类型定义到多个文件（类、结构、接口）。

  ```csharp
  // File1.cs
  public partial class MyClass { }
  // File2.cs
  public partial class MyClass { }
  ```

## 三、流程控制

### 1. 条件语句

- `if` / `else`：条件判断。

  ```csharp
  if (x > 0) { } else if (x < 0) { } else { }
  ```

- `switch` / `case` / `default`：多条件分支（C# 8+ 支持模式匹配）。

  ```csharp
  switch (color)
  {
      case Color.Red: Console.WriteLine("Red"); break;
      default: Console.WriteLine("Other"); break;
  }
  ```

- `case`：`switch` 中的分支标签。
- `default`：`switch` 中未匹配时的默认分支。

### 2. 循环语句

- `for`：固定次数循环。

  ```csharp
  for (int i = 0; i < 10; i++) { }
  ```

- `foreach`：遍历集合。

  ```csharp
  foreach (var item in list) { }
  ```

- `while`：条件循环（先判断后执行）。

  ```csharp
  while (x > 0) { x--; }
  ```

- `do`：条件循环（先执行后判断）。

  ```csharp
  do { x--; } while (x > 0);
  ```

- `break`：退出当前循环或 `switch`。
- `continue`：跳过当前循环的剩余部分，进入下一次迭代。

### 3. 跳转语句

- `return`：从方法返回值（或终止方法）。

  ```csharp
  public int GetValue() { return 42; }
  ```

- `goto`：跳转到标签（慎用，破坏结构化）。

  ```csharp
  goto End;
  End: Console.WriteLine("Done");
  ```

## 四、异常处理

- `try`：包裹可能抛出异常的代码。
- `catch`：捕获并处理异常。
- `finally`：无论是否发生异常，均执行的代码（如释放资源）。

  ```csharp
  try { /* 可能出错的代码 */ }
  catch (IOException ex) { /* 处理IO异常 */ }
  finally { /* 清理资源 */ }
  ```

- `throw`：抛出异常。

  ```csharp
  if (x < 0) throw new ArgumentException("x must be positive");
  ```

## 五、类型操作

- `typeof`：获取类型的 `Type` 对象。

  ```csharp
  Type type = typeof(int);
  ```

- `is`：检查对象是否为指定类型（或模式匹配）。

  ```csharp
  if (obj is string s) { Console.WriteLine(s); }
  ```

- `as`：安全地将对象转换为指定类型（失败返回 `null`）。

  ```csharp
  string s = obj as string;
  ```

- `sizeof`：获取值类型的大小（字节数，仅在 `unsafe` 代码中可用）。

  ```csharp
  unsafe { int size = sizeof(int); }
  ```

- `default`：获取类型的默认值（值类型为 `0`，引用类型为 `null`）。

  ```csharp
  int x = default; // 0
  string s = default; // null
  ```

- `nameof`：获取变量、类型或成员的名称（字符串）。

  ```csharp
  Console.WriteLine(nameof(x)); // 输出 "x"
  ```

## 六、参数与方法

- `params`：允许方法接受可变数量的参数（必须是最后一个参数）。

  ```csharp
  public int Sum(params int[] numbers) { /* 求和 */ }
  ```

- `in`：声明输入参数（按引用传递，但不允许修改）。

  ```csharp
  public void Print(in int value) { /* 不能修改 value */ }
  ```

- `ref`：声明引用参数（按引用传递，允许修改）。

  ```csharp
  public void Increment(ref int x) { x++; }
  ```

- `out`：声明输出参数（必须在方法内赋值）。

  ```csharp
  public bool TryParse(string s, out int result) { /* 赋值 result */ }
  ```

- `async` / `await`：异步编程（简化异步操作）。

  ```csharp
  public async Task<string> FetchData()
  {
      var data = await HttpClient.GetAsync("url");
      return await data.Content.ReadAsStringAsync();
  }
  ```

## 七、命名空间与 using

- `namespace`：定义命名空间（避免类型名冲突）。

  ```csharp
  namespace MyApp.Utils { public class Helper { } }
  ```

- `using`：引入命名空间，或定义资源的范围（自动释放）。

  ```csharp
  using System; // 引入命名空间
  using (var stream = new FileStream("file.txt", FileMode.Open)) { } // 自动释放资源
  ```

- `global::`：引用全局命名空间（解决命名冲突）。

  ```csharp
  global::System.Console.WriteLine("Hello"); // 明确使用系统的 Console
  ```

## 八、其他关键字

- `void`：表示方法无返回值。

  ```csharp
  public void Print() { }
  ```

- `object`：所有类型的基类（别名 `System.Object`）。
- `string`：字符串类型（别名 `System.String`）。
- `bool`：布尔类型（`true`/`false`）。
- `byte`/`sbyte`/`short`/`ushort`/`int`/`uint`/`long`/`ulong`：整数类型。
- `float`/`double`/`decimal`：浮点类型。
- `char`：字符类型（单个 Unicode 字符）。
- `null`：空引用（表示引用类型未指向任何对象）。
- `true`/`false`：布尔值。
- `this`：当前实例的引用。

  ```csharp
  public class Person { public Person(string name) { this.Name = name; } }
  ```

- `base`：访问基类的成员。

  ```csharp
  public class Child : Parent { public Child() : base() { } }
  ```

- `new`：创建实例、隐藏基类成员（方法 / 字段）。

  ```csharp
  var obj = new MyClass(); // 创建实例
  public new void Method() { } // 隐藏基类方法
  ```

- `operator`：定义运算符重载。

  ```csharp
  public static Vector operator +(Vector a, Vector b) => new(a.X + b.X, a.Y + b.Y);
  ```

- `checked`/`unchecked`：控制整数运算溢出检查。

  ```csharp
  checked { int x = int.MaxValue + 1; } // 抛出溢出异常
  ```

- `unsafe`：允许使用指针（需在项目中启用 "不安全代码"）。

  ```csharp
  unsafe { int* ptr = &x; }
  ```

- `fixed`：固定变量地址（防止垃圾回收移动，用于 `unsafe` 代码）。

  ```csharp
  fixed (char* p = str) { }
  ```

- `stackalloc`：在栈上分配内存（`unsafe` 代码）。

  ```csharp
  unsafe { int* buffer = stackalloc int[10]; }
  ```

- `yield`：在迭代器中返回序列元素（`yield return`/`yield break`）。

  ```csharp
  public IEnumerable<int> GetNumbers()
  {
      yield return 1;
      yield return 2;
  }
  ```

- `partial`：拆分类型定义（见上文 "成员定义"）。
- `required`（C# 11+）：要求对象初始化时必须赋值的属性。

  ```csharp
  public class User { public required string Name { get; set; } }
  ```

- `file`（C# 11+）：限制类型仅在当前文件中可见。

  ```csharp
  file class FileInternal { }
  ```
