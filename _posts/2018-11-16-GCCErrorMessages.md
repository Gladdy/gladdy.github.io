---
layout: post
title:  "A difference in error messages between GCC and Clang"
date:   2018-11-16 21:10:32
tags: cpp
abstract: "GCC isn't always the most helpful in suggesting where you went wrong" 
categories: code
---

C++ can be fairly complicated language and it doesn't help that the compiler warnings aren't always the most helpful.
As an example, take the code below, it's a distilled summary illustrating the problem, and it took me a fair amount of time to work out why g++ was rejecting it.

{% highlight C++ %}
template <typename T> class proc {
  struct inner {
    enum class mode { A, B } m;
  };
 
  inner *impl(inner::mode m) { return nullptr; }
 
public:
  void do_stuff() { auto p = impl(inner::mode::A); }
};
 
int main(int argc, char **argv) {
  proc<int> p;
  p.do_stuff();
}
{% endhighlight %}

## The error according to GCC
{% highlight bash %}
martijn@martijn-laptop:~/Software$ g++ file.cpp 
file.cpp:7:9: error: expected ‘;’ at end of member declaration
 inner * impl(inner::mode m) {
         ^~~~
             ;
file.cpp:7:25: error: expected ‘)’ before ‘m’
 inner * impl(inner::mode m) {
             ~           ^~
                         )
file.cpp: In instantiation of ‘void proc<T>::do_stuff() [with T = int]’:
file.cpp:18:12:   required from here
file.cpp:12:6: error: expression cannot be used as a function
 auto p = impl(inner::mode::A);
      ^
{% endhighlight %}

## The error according to clang
{% highlight bash %}
martijn@martijn-laptop:~/Software$ clang++ file.cpp 
file.cpp:7:14: error: missing 'typename' prior to dependent type name 'inner::mode'
inner * impl(inner::mode m) {
             ^~~~~~~~~~~
             typename 
1 error generated.
{% endhighlight %}

Short and to the point - but most importantly, it gives a valuable suggestion as to what is wrong in the code. In this case, because the `class proc` is actually templated, the type of `inner::mode` depends on what the type of T is (hence it is called a dependent type). What threw me off here is that the `struct inner` was only kept as an inner struct as an utility in the implementation of the class and in fact, does not depend on T at all. However, as it is an inner class, when it has to assign a symbol to `struct inner` it will assign it `proc<T>::inner`, for whatever values T gets instantiated as. This means that from the compilers perspective it has to be a dependent type - and hence the typename is required to resolve the ambiguity that might otherwise arise.