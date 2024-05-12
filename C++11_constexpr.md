# constexpr

constexpr 用于验证是否是常量表达式

## 用法：

### 修饰常量：

`constexpr int a = 1 + 2`;

constexpr 和 const 不同的区别在于，constexpr 修饰的一定是常量，其次修饰的是表达式

### 修饰函数: 

constexpr 修饰函数必须满足以下 4 个条件：

1. 函数体内除了 using、typedef 以及 static_assert 式子外只能有 return 返回式存在

   例如：

   ```c++
   constexpr int sum_10(int &r){
     int sum = 10 + r;
     return sum;
   }
   ```

   这种写法会报错，正确的写法是：

   ```c++
   constexpr int sum_10(int &r){
     return 10 + r;
   }
   ```

2. 函数的返回值不能为 void

3. 函数返回的必须是一个常量表达式

4. 函数必须在使用前就被定义，否则无法通过编译

   > 一般的函数只要在使用前存在声明就可以了，因为函数的寻址是在运行时进行的（函数的定义必须存在，但是可以在任何地方）。

   <font color=purple> 在使用了 constexpr 修饰函数后报错时，错误信息中会提到 inline 关键字，而在 C++ 中 inline 关键字修饰的函数在编译阶段会直接将函数体放在使用了该内联函数的地方，这种特征是符合 constexpr 的，所以 constexpr 在这种情况下是在编译阶段执行的 </font>

   例如：

   ```c++
   int ges_();
   constexpr int ges();
   int main(){
       ges_();
       ges();
       return 0;
   }
   ```

   错误信息：

   ```c++
   newfile.cpp:7:15: warning: inline function 'ges' is not defined [-Wundefined-inline]
   constexpr int ges();
                 ^
   newfile.cpp:10:5: note: used here
       ges();
       ^
   1 warning generated.
   ld: Undefined symbols:
     ges(), referenced from:
         _main in newfile-a63849.o
     ges_(), referenced from:
         _main in newfile-a63849.o
   clang: error: linker command failed with exit code 1 (use -v to see invocation)
   ```

   在错误信息的第一行中可以看到 `inline function 'ges' is not defined`，这说明 constexpr 函数也是内联函数

### 修饰自定义类型

若没有对应的使用 constexpr 修饰的构造函数的话是无法成功创建 constexpr 修饰的自定义类型的对象的

在使用 constexpr 修饰构造函数时需满足以下条件：

1. 构造函数的函数成员必须通过初始化列表进行初始化

   > 这很容易理解，因为用 constexpr 修饰的对象的成员属性相当于常量，而常量的初始化必须在初始化列表中进行（这可能和构造函数分配地址的时间有关）

2. 构造函数的函数体内不能有内容除了 using、typedef、static_assert 表达式之外的内容

   > 因为构造函数首先是函数，而构造函数不能有返回值

### 修饰模版函数

constexpr 可以修饰模版函数，若给模版函数分配的类型不适配常量表达式，则 constexpr 会被无视掉，这和 inline 是一样的

## constexpr 和 const 的区别

### const

在 C++98/03 中，const 是具有二义性的，它可以表示只读，也可以表示常量

例如：

```c++
int initArray_1(const int n){
  array<int, n> a;
}
int initArray_2(){
  const int n = 10;
  array<int, n> a;
}
```

在这两个函数中，函数 initArray_1 会报错，而函数 initArray_2 可以成功运行。这种现象就是函数的二义性，因为在第一个函数中的 n 作为参数存在，参数列表里的 const int 表示 n 是只读的，而函数 2 里的 const int 表示 n 是个常量

> 究其原因，二义性其实是编译器的处理问题导致的，当 const int n 作为函数的参数时，n 的内容是在运行时由函数的调用决定的，而 const int n 作为常量时，n 就像被 #define 了意义，这种处理是在编译阶段进行的

而在 C++11 中，为了避免这种二义性，const 只具有只读的含义了（只读是可以修改的，我们可以在将这个数的地址去const后，再定义一个 non-const 的指针指向这个地址，然后就能照常更改了），而常数的含义给了 constexpr，这也意味着 constexpr 修饰的常量是一定无法修改的

注意：

constexpr 函数中的参数作为返回会的常量表达式的一部分时，编译器会将其判定为常量，因为 constexpr 修饰的内容会在编译阶段确定，所以不能更改该参数