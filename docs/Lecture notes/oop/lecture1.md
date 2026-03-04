## 第一个C++程序：Hello, World!


```cpp
// 包含输入输出流库
#include <iostream>
// 使用标准命名空间
using namespace std;//std 是 Standard（标准）的缩写，声明使用标准库命名空间，可直接写 cout 替代 std::cout，简化代码


// 程序入口点 main 函数
int main() {
    cout << "Hello, World!" << endl;
    cout << "欢迎来到C++的世界！" << endl;
    cout << "Welcome to the world of C++!" << endl;//endl：(end line) 是一个特殊的操作符，表示输出一个换行符，并刷新输出缓冲区，确保内容立即显示在屏幕上。也可以用'\n'来表示换行，但'\n'不会刷新缓冲区。

    return 0;
}

```

![image-20260304141644175](./assets/image-20260304141644175-1772605007191-1.png)

* 列表初始化不允许缩窄转换，即使值在目标类型范围内。
