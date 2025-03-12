+++
title = 'C++ 模拟 Rust Trait'
date = 2025-03-12T22:11:24+08:00
draft = false
tags = ['编程']

+++

接触 `rust` 倒是挺早了，看着它的生态慢慢好起来。  
第一次用上 `trait` 就在想，这么好的东西，该怎么在 `c++` 里用上呢。  
不过不算刚需，就一直搁置着，最近尝试简单实现了一下。  

## Rust Trait
先来看看 rust trait 的例子。  
简单来说，就是 "定义接口，实现接口，接口多态"。  
其实 `c++` 这边已经有一套完整的继承范式来实现上述需求了，"虚接口，继承实现，虚多态"。  
不过虚函数会要求强制动态分发，然后给类实例添加虚表指针。  
所以要尝试不用虚继承来实现 `trait`。  

### keywords
- trait → Defines a trait.  
  ```rust
  trait Speak {
      fn speak(&self) -> String;
  }
  ```
- impl → Implements a trait for a type.  
  ```rust
  struct Dog;
  impl Speak for Dog {
      fn speak(&self) -> String {
          "Woof!".to_string()
      }
  }
  ```
- where → Adds trait bounds in a structured way.  
  ```rust
  fn make_speak<T>(animal: T) 
  where 
      T: Speak 
  {
      println!("{}", animal.speak());
  }
  ```
- dyn → Used for dynamic dispatch.  
  ```rust
  fn speak_dyn(animal: &dyn Speak) {
      println!("{}", animal.speak());
  }
  ```

## C++ 实现

[完整的代码](https://github.com/hypengw/rstd/blob/master/src/core/trait.cppm)  

### 接口定义

> 在没有反射的代码生成时，接口需要同时带有 delegate 功能

`requires` 只能用作约束，它无法落地成具体的函数，所以需要定义具体的接口类。  
现在我们可以定义任意接口类，然后写一套函数的声明。  
然后该怎么通过接口去调用实现呢，如果是具体的实例，它应该会具体的实现函数，可以正常调用。但是考虑 `dyn` 的实现，以及**C++没法通过模板去生成变量/函数的名字**，我想的是让接口类自己带有 `delegate` 的实现，需要情况的不同的去调用不同的实现。
```c++
template<typename T>
struct Speak {
    auto speak() -> std::string { M::template call<0>(this); }

private:
    using M = TraitMeta<MyTrait, T>;
    friend M;
    template<typename F>
    static consteval auto collect() {
        return TraitApi { &F::speak };
    }
};
```
- `template call<X>`: 查找 `vtable` 里的函数，然后调用。  
  对于 `vtable`，静态分发是引用 `constexpr static` 变量，动态分发是通过指针引用。  
- `collect`: 在没有反射的情况下，帮助获取接口的类型和地址。由于是模板，所以也可以用于验证是否被正确实现。  
- `TraitApi`: 内部是一个简单 `tuple`，存储类型信息和地址。  

### 接口实现

`c++` 里常见的 `Customization Points` 有 重载，模板特化，Policy 以及 ADL。  
这里我们选择模板特化，`rust` 其实也是类似的实现。  
比如，`Orphan Rule` 即对 `trait` 的孤儿原则：  
- 为自己的类型实现外部 Trait
- 为外部类型实现自己的 Trait

在 `c++` 的视角下，非常好理解，这两条规则都是为了让 Trait 的实现（即模板特化）对引用它的编译单元（即引用 `.h` 的 `.cpp/.cc`）一定可见。  
这样就能避免在不同的编译单元产生不同的实现。  

```c++
struct Dog;

template<>
struct Impl<Speak, Dog> {
    static auto speak(TraitPtr self) -> std::string;
};

struct Dog : Speak<Dog>  { std::string voice {"Woof!"}; };

auto Impl<Speak, Point>::speak(TraitPtr self) -> std::string {
    return self.as_ref<Point>().voice;
}
...
```
- `Impl` 接受一个 Trait 模板和具体类型。  
- `: Speak<Dog>` 非虚继承，来让 `Dog` 类拥有具体的接口函数，即 `Dog().speak()`。  
- `Impl<Speak, Dog>::reset(Dog())` 直接调用实现。  

这里把 `Dog` 的 `fields` 分开定义会好一些。  
这样可以在 `Dog` 还未定义的时候，直接操作 `Dog fileds`。
```c++
struct Dog;
struct DogFields {
    std::string voice {"Woof!"};
};
// Impl Speak<Dog> ...
struct Dog : DogFields, Speak<Dog> {}
```

### 静态分发
用 `std::semiregular` 来判断 `Impl` 是否有完整定义，当然也可以自己写模板判断 `size`。  
``` c++
template<typename A, template<typename> class... T>
concept Implemented = (std::semiregular<Impl<T, A>> && ...);
// ...

template<typename T>
    requires Implemented<T, Speak>
void make_speak(T& animal) {
    std::printf("{}", animal.speak());
}
```

### dyn 胖指针
即存储一个 `vtable` 指针和 `self` 指针。  
然后利用接口类的分发功能来实现调用。  

```c++
template<template<typename> class Tr, ConstNess Cn>
class Dyn : public Tr<DynImpl<Tr>> {
    using M = TraitMeta<Tr, DynImpl<Tr>>;
    friend M;
    using ptr_t = std::conditional_t<Cn == ConstNess::Const, const TraitPtr, TraitPtr>;

    const decltype(M::apis)* const apis;
    ptr_t                          self;
    ...
}
```
- `Tr`: 一个 `Trait` 接口
- `Tr<DynImpl<Tr>>`： 即用 `DynImpl` 标签标记给 `Tr`，生成具体的调用函数
- `apis`: `vtable` 指针
- `Cn`: `Tr` 无法拥有 `const` 标记，所有需要额外的参数来标记 `ConstNess`