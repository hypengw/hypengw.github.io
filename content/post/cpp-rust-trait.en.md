+++
title = 'Simulating Rust Traits in C++'
date = 2025-03-12T22:11:24+08:00
draft = false
tags = ['Coding']
+++

I got in touch with `rust` quite early and watched its ecosystem gradually improve.  
When I first used `trait`, I wondered how to implement such a great feature in `c++`.  
Though it wasn't an urgent need, I put it aside. Recently, I tried to implement it in a simple way.

## Rust Trait
Let's first look at an example of rust traits.  
Simply put, it's about *defining interfaces, implementing interfaces, and interface polymorphism*.  
Actually, `c++` already has a complete inheritance paradigm to implement these requirements - *virtual interface, inheritance implementation, virtual polymorphism*.  
However, virtual functions require forced dynamic dispatch and add virtual table pointers to class instances.  
So let's try to implement `trait` without using virtual inheritance.  

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

## C++ Implementation

[Complete code](https://github.com/hypengw/rstd/blob/master/src/core/trait.cppm)    
The specific implementation may change, refer to the documentation in the repo

### Interface Definition

> When there's no reflection for code generation, the interface needs to include delegate functionality

`requires` can only be used as constraints; it cannot be materialized into concrete functions, so we need to define concrete interface classes.  
Now we can define any interface class and write a set of function declarations.  
Then how do we call the implementation through the interface? For concrete instances, they should have concrete implementation functions that can be called normally. But considering the implementation of `dyn`, and that **C++ cannot generate variable/function names through templates**, I thought about having the interface class carry its own `delegate` implementation, which calls different implementations based on different situations.  
```c++
template<typename T>
struct Speak {
    auto speak() -> std::string { return M::template call<0>(this); }

private:
    using M = TraitMeta<Speak, T>;
    friend M;
    template<typename F>
    static consteval auto collect() {
        return TraitApi { &F::speak };
    }
};
```
- `template call<X>`: Looks up the function in the `vtable` and then calls it.
  For `vtable`, static dispatch references `constexpr static` variables, while dynamic dispatch references through pointers.
- `collect`: Helps to get the type and address of the interface without reflection. Since it's a template, it can also be used to verify if the interface is correctly implemented.
- `TraitApi`: Internally, it's a simple `tuple` that stores type information and addresses.

### Interface Implementation

Common `Customization Points` in `c++` include overloading, template specialization, Policy, and ADL.  
Here we choose template specialization, which is similar to how `rust` implements it.  
For example, the `Orphan Rule`, which is the orphan principle for `trait`:  
- Implement external Traits for your own types  
- Implement your own Traits for external types

From the perspective of `c++`, it's easy to understand that these two rules ensure that the implementation of the Trait (i.e., template specialization) is visible to the compilation unit that references it (i.e., the `.cpp/.cc` that references the `.h`).  
This avoids generating different implementations in different compilation units.  

```c++
struct Dog;

template<>
struct Impl<Speak, Dog> {
    static auto speak(TraitPtr self) -> std::string;
};

struct Dog : Speak<Dog>  { std::string voice {"Woof!"}; };

auto Impl<Speak, Dog>::speak(TraitPtr self) -> std::string {
    return self.as_ref<Dog>().voice;
}
...
```
- `Impl` accepts a Trait template and a concrete type.
- `: Speak<Dog>` non-virtual inheritance allows the `Dog` class to have concrete interface functions, i.e., `Dog().speak()`.
- `Impl<Speak, Dog>::speak(Dog())` directly calls the implementation.

It's better to separate the `fields` definition of `Dog`.  
This way, you can directly operate on `Dog fields` even when `Dog` is not yet defined.  
```c++
struct Dog;
struct DogFields {
    std::string voice {"Woof!"};
};
// Impl Speak<Dog> ...
struct Dog : DogFields, Speak<Dog> {}
```

### Static Dispatch
Use `std::semiregular` to determine if `Impl` is fully defined, or you can write a template to check the `size`.  
``` c++
template<typename A, template<typename> class... T>
concept Implemented = (std::semiregular<Impl<T, A>> && ...);
// ...

template<typename T>
    requires Implemented<T, Speak>
void make_speak(T& animal) {
    std::print("{}", animal.speak());
}
```

### dyn Fat Pointer
It stores a `vtable` pointer and a `self` pointer.  
Then use the dispatch functionality of the interface class to make the call.  

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
// ...

Dog dog;
auto dyn = make_dyn<Speak>(dog);
std::print("{}", dyn.speak());
```
- `Tr`: A `Trait` interface
- `Tr<DynImpl<Tr>>`: Uses the `DynImpl` tag to mark `Tr`, generating concrete call functions
- `apis`: `vtable` pointer
- `Cn`: `Tr` cannot have a `const` tag, so an additional parameter is needed to mark `ConstNess`

### Box dyn
TODO