## 头文件与类的声明

1. 防卫式声明
```c++
// complex.h
#ifndef __COMPLEX__
#define __COMPLEX__

// ...

#endif
```

## 构造函数

1. 构造函数可以重载

2. 构造函数写法：
```c++
// class complex
double re, im;

// 好的写法
complex(double r = 0, double i = 0)
: re(r), im(i) // 设置初始值，效率高
{ }

// 不好的写法
complex(double r = 0, double i = 0) {
    // 重新赋值
    re = r;
    im = i;
}
```


3. inline 函数

函数若在 class body 内定义完成，则自动成为 inline 候选人

4. 访问级别

`public` 外界可以访问；`private` 只有类自己才能访问