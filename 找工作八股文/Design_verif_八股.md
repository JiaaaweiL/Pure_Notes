# C++/OOP 常识：
### 1. 如何不用temp去swap两个veriable？ 
方法1：加减法    
```CPP
A = A + B;		
B = A - B;  # B = A + B - B => B = A;		
A = A - B;	# A = A + B - A => A = B;
```
方法2：位运算
```cpp
#性质是 X ^ X = 0; X ^ 0 = X;
A = A ^ B;
B = A ^ B; # A ^ B ^ B = A; Thus B = A;
A = A ^ B; A ^ A ^ B; A = B;
```
### 2. Pass by value 和 Pass by reference有什么区别？
Pass by value: 函数接收的是实参的副本（即变量的拷贝）。在函数内部对参数的修改不会影响实参, 实参与形参在内存中是两个独立的变量。对于大型数据结构（如数组、对象等），拷贝会占用更多内存并增加执行时间。     
Pass by reference: 函数接收的是实参的引用，形参和实参指向同一块内存。在函数内部对参数的修改直接影响实参。使用引用或指针实现。不需要执行拷贝， 不会占用更多内存并增加执行时间。    
Pass by value:
```cpp
void func(int x) {
    x = 10;  // 修改形参
}
int main() {
    int a = 5;
    func(a);
    // a 仍然是 5，因为修改的是 x 的副本
    return 0;
}
```
Pass by reference:
```cpp
void func(int& x) {
    x = 10;  // 修改实参的值
}
int main() {
    int a = 5;
    func(a);  // 传入引用
    // a 变成了 10，因为直接修改了实参123
    return 0;
}
```   


