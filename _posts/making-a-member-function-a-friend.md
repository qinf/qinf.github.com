title: 成员函数作为友元
date: 2015-01-08
categories: C++
tags: [C++]
---
令成员函数成为友元(`《C++ Primer》5th`, P281)，练习7.32：
    定义你自己的`Screen`和`Window_mgr`，其中`clear`是`Window_mgr`的成员，是`Screen`的友元。
    <!-- more -->
记录一下出错信息[github](https://github.com/Mooophy/Cpp-Primer/issues/123)
自己写的：
`Screen.h`
```C++
#ifndef _screen_h
#define _screen_h

#include <string>
#include <iostream>

class Screen {
public:
    friend void Window_mgr::clear(ScreenIndex);
    //friend class Window_mgr;
    using pos = std::string::size_type;
    Screen() = default;
    Screen(pos ht, pos wd): height(ht), width(wd), contents(ht*wd, ' ') { }
    Screen(pos ht, pos wd, char c): height(ht), width(wd), contents(ht*wd, c) { }
    char get() const {
        return contents[cursor];
    }
    inline char get(pos ht, pos wd) const;
    Screen &move(pos r, pos c);
    Screen &set(char);
    Screen &set(pos, pos, char);
    void some_member() const;
    Screen &display(std::ostream &os) {
        do_display(os);
        return *this;
    }
    const Screen &display(std::ostream &os) const {
        do_display(os);
        return *this;
    }
private:
    mutable size_t access_ctr;
    pos cursor = 0;
    pos height = 0, width = 0;
    std::string contents;
    void do_display(std::ostream &os) const {
        os << contents;
    }
};

inline Screen &Screen::set(char c) {
    contents[cursor] = c;
    return *this;
}
inline Screen &Screen::set(pos row, pos col, char ch) {
    contents[row*width+col] = ch;
    return *this;
}
#endif
```
`Window_mgr.h`
```C++
#ifndef _Window_mgr_h
#define _Window_mgr_h
#include <string>
#include <vector>
#include "Screen.h"
class Window_mgr {
public:
    using ScreenIndex = std::vector<Screen>::size_type;
    void clear(ScreenIndex);
private:
    std::vector<Screen> screens{Screen(24, 80, ' ')};
};
inline void Window_mgr::clear(ScreenIndex index) {
    Screen &s = screens[index];
    s.contents = std::string(s.height * s.width, ' ');
}

#endif
```
编译: `clang++ Window_mgr.h Screen.h -std=c++11`
错误信息:
```
clang: warning: treating 'c-header' input as 'c++-header' when in C++ mode, this behavior is deprecated
clang: warning: treating 'c-header' input as 'c++-header' when in C++ mode, this behavior is deprecated
In file included from Window_mgr.h:5:
./Screen.h:9:17: error: use of undeclared identifier 'Window_mgr'
    friend void Window_mgr::clear(ScreenIndex);
                ^
./Screen.h:9:35: error: unknown type name 'ScreenIndex'
    friend void Window_mgr::clear(ScreenIndex);
                                  ^
Window_mgr.h:15:7: error: 'contents' is a private member of 'Screen'
    s.contents = std::string(s.height * s.width, ' ');
      ^
./Screen.h:35:17: note: declared private here
    std::string contents;
                ^
Window_mgr.h:15:32: error: 'height' is a private member of 'Screen'
    s.contents = std::string(s.height * s.width, ' ');
                               ^
./Screen.h:34:9: note: declared private here
    pos height = 0, width = 0;
        ^
Window_mgr.h:15:43: error: 'width' is a private member of 'Screen'
    s.contents = std::string(s.height * s.width, ' ');
                                          ^
./Screen.h:34:21: note: declared private here
    pos height = 0, width = 0;
                    ^
5 errors generated.
Screen.h:9:17: error: use of undeclared identifier 'Window_mgr'
    friend void Window_mgr::clear(ScreenIndex);
                ^
Screen.h:9:35: error: unknown type name 'ScreenIndex'
    friend void Window_mgr::clear(ScreenIndex);
                                  ^
2 errors generated.
```
`pezy`的解答
> 编译： clang++ Window_mgr.h Screen.h -std=c++11

抛开错误，我们先看看编译器的编译顺序，按你代码的写法，编译器先找到`Window_mgr.h`，里面发现有这么一句:
```
#include "Screen.h"
```
于是二话不说，就去找`Screen.h`，这才正式进入`Screen`类，赫然发现有一个友元函数叫这个：
```
Window_mgr::clear(ScreenIndex);
```
里面有两个东西没见过，不认识`Window_mgr`和`ScreenIndex`。
注意咱们的顺序，编译器在进入`Window_mgr`类之前，先进入的是`Screen`类，所以在`Screen`类里，它并不认识`Window_mgr`和`ScreenIndex`。所以报错：
```
./Screen.h:9:17: error: use of undeclared identifier 'Window_mgr'
    friend void Window_mgr::clear(ScreenIndex);
                    ^
./Screen.h:9:35: error: unknown type name 'ScreenIndex'
    friend void Window_mgr::clear(ScreenIndex);
                                  ^
```
到这一步，编译器显然放弃将`clear`函数视为友元了，无怪乎你在`clear`函数中想调用`Screen`类的私有方法，就会报错了：
```
Window_mgr.h:15:7: error: 'contents' is a private member of 'Screen'
    s.contents = std::string(s.height * s.width, ' ');
          ^
./Screen.h:35:17: note: declared private here
    std::string contents;
                    ^
Window_mgr.h:15:32: error: 'height' is a private member of 'Screen'
    s.contents = std::string(s.height * s.width, ' ');
                                   ^
./Screen.h:34:9: note: declared private here
    pos height = 0, width = 0;
            ^
Window_mgr.h:15:43: error: 'width' is a private member of 'Screen'
    s.contents = std::string(s.height * s.width, ' ');
                                              ^
./Screen.h:34:21: note: declared private here
    pos height = 0, width = 0;
                        ^
```
**那么如何修正呢？**
很容易，调整顺序即可。
```
#ifndef _screen_h
#define _screen_h

#include <string>
#include <iostream>

#include "Window_mgr.h"

class Screen {
public:
    friend void Window_mgr::clear(ScreenIndex);
    //friend class Window_mgr;
    using pos = std::string::size_type;
    Screen() = default;
    Screen(pos ht, pos wd): height(ht), width(wd), contents(ht*wd, ' ') { }
    Screen(pos ht, pos wd, char c): height(ht), width(wd), contents(ht*wd, c) { }
    char get() const {
        return contents[cursor];
    }
inline char get(pos ht, pos wd) const;
    Screen &move(pos r, pos c);
    Screen &set(char);
    Screen &set(pos, pos, char);
    void some_member() const;
    Screen &display(std::ostream &os) {
        do_display(os);
        return *this;
    }
    const Screen &display(std::ostream &os) const {
        do_display(os);
        return *this;
    }
private:
    mutable size_t access_ctr;
    pos cursor = 0;
    pos height = 0, width = 0;
    std::string contents;
    void do_display(std::ostream &os) const {
        os << contents;
    }
};

inline Screen &Screen::set(char c) {
    contents[cursor] = c;
    return *this;
}
inline Screen &Screen::set(pos row, pos col, char ch) {
    contents[row*width+col] = ch;
    return *this;
}

inline void Window_mgr::clear(ScreenIndex index) {
    Screen &s = screens[index];
    s.contents = std::string(s.height * s.width, ' ');
}
#endif
```
`Window_mgr.h`
```
#ifndef _Window_mgr_h
#define _Window_mgr_h
#include <string>
#include <vector>

class Screen;

class Window_mgr {
public:
    using ScreenIndex = std::vector<Screen>::size_type;
    void clear(ScreenIndex);
private:
    std::vector<Screen> screens;
};

#endif
```
在调整顺序的过程中，我略加修正的地方：

- `Window_mgr`是`Screen`的友元，所以在`Screen.h`中`#include "Window_mgr.h"`
- `Window_mgr`也需要用到`Screen`类，所以增加前置声明`class Screen;`
- 去掉对`Window_mgr`私有成员`screens`的类内初始化，因为此时`Screen`类并未初始化。
- 将`Window_mgr`的`clear`方法的实现移至`Screen.h`中，因为它是`Screen`的友元函数，需要用到`Screen`类的各种私有信息，显然要等`Screen`初始化完成，故移至`Screen.h`的最后。
其余未作任何调整。
