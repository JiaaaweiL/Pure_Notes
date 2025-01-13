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
### 3.Memory layout
![image](https://github.com/user-attachments/assets/e199f7d0-8c3d-46dc-92f7-d98c50903b7a)

代码段Text Segment，包括二进制可执行代码；   
数据段Data Segment，包括已初始化的静态常量和全局变量；    
BSS 段Block Started by Symbol，包括未初始化的静态变量和全局变量；   
堆段Heap Segment，包括动态分配的内存，从低地址开始向上增长；   
文件映射段Memory-Mapped Segment，包括动态库、共享内存等，从低地址开始向上增长；    
栈段Stack Segment，包括局部变量和函数调用的上下文等。栈的大小是固定的，一般是8MB，当然系统也提供了参数，以便我们自定义大小；    
![image](https://github.com/user-attachments/assets/e9e96930-752d-409b-a82d-b0213cb88501)            
**Q & A:** 
- 为什么堆效率比栈低？ 堆需要动态分配和释放内存，涉及复杂的管理（如碎片整理），而栈只需调整栈指针，速度快。   
- 全局变量存在哪里？已初始化的全局变量存储在初始化数据段；未初始化的存储在BSS段。   
- 为什么代码段是只读的？ 防止修改程序的指令，提高安全性和稳定性。  
- 如何避免栈溢出？ 减少递归深度，避免定义大数组。   
- 全局数据，静态数据放那里？ 常量放哪里？ 全局数据，静态数据分初始化/未初始化。初始化的放Data segment，未初始化的放BSS。 字符串常量放test segment， constant 修饰的全局变量放text segment，const修饰的局部变量放stack

已初始化静态变量	    数据段（Data Segment）	    static int x = 10;   
未初始化静态变量	    BSS 段（BSS Segment）	    static int x;   
已初始化全局变量	    数据段（Data Segment）	    int global_x = 100;    
未初始化全局变量	    BSS 段（BSS Segment）	    int global_x;     
字符串常量	        代码段（Text Segment）	    const char* str = "Hello";     
全局常量	            代码段（Text Segment）	    const int x = 50;    
局部常量	            栈段（Stack Segment）	    const int local_x = 20;（在函数内部定义）     

### 4. 关键字static  
static修饰变量的意义是：在内存中，仅仅只有这一份。每个类也只能共享一个static变量。无论这个类有多少instances，访问到的是同一个东西。     
static修饰方法的意义是： 我们无需创建类的子类，也能直接通过 "类名::方法名字" 调用方法。 不依赖于类的具体对象，可以直接通过类名调用。静态方法不能直接访问非静态成员（因为非静态成员需要具体对象），但可以访问静态成员。    
如果需要在静态方法中访问非静态成员变量，必须通过某个对象显式访问。 示例：
```cpp
class MyClass {
private:
    int value; // 非静态成员变量
public:
    MyClass(int v) : value(v) {}
    static void displayValue(MyClass& obj) {
        std::cout << "Value: " << obj.value << std::endl;
    }
};
int main() {
    MyClass obj(10);
    MyClass::displayValue(obj); // 显式传递对象
    return 0;
}
```
静态成员变量属于类本身，和具体的对象无关，因此普通方法可以直接通过类名访问静态成员变量。 普通方法拥有 this 指针，也可以通过 this-> 间接访问静态成员。
```cpp
class MyClass {
public:
    static int count; // 静态成员变量
    void increment() {
        count++; // 普通方法直接访问静态变量
    }
};
int MyClass::count = 0;
int main() {
    MyClass obj1, obj2;
    obj1.increment();
    obj2.increment();
    std::cout << MyClass::count << std::endl; // 输出: 2
    return 0;
}

```

### 5. OOP的三要素：封装（encapsulation）， 继承（Inheritance），多态（Polymorphism）
1. 封装  将数据（变量）和操作数据的方法（函数）绑定在一起，通过访问权限控制（如 private, protected, public）对数据进行保护，避免直接访问或修改数据，从而提高代码的安全性和可维护性。     
将类的内部实现隐藏，对外只暴露必要的接口。限制外部直接访问类的内部数据，必须通过公开的方法（如 getter 和 setter）访问或修改。提高了代码的安全性，防止数据被意外或非法修改。
```cpp
#include <iostream>
using namespace std;

class BankAccount {
private:
    double balance; // 私有变量，外部不能直接访问
public:
    // 构造函数
    BankAccount(double initial_balance) : balance(initial_balance) {}

    // 公有接口（getter）
    double getBalance() const {
        return balance;
    }
    // 公有接口（setter）
    void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
};
int main() {
    BankAccount account(100.0); // 初始余额 100
    account.deposit(50.0); // 存入 50
    cout << "Current balance: " << account.getBalance() << endl; // 输出余额
    return 0;
}
```
2. 继承（Inheritance） 通过继承，子类（派生类）可以继承父类（基类）的属性和方法，从而实现代码的复用和扩展。同时，子类可以新增自己的属性和方法，或者重写父类的方法。     
代码复用：避免重复代码，子类可以直接复用父类的功能。继承是为了代码复用，同时为多态的实现打下基础。       
```cpp
class Animal {
public:
    void eat() {
        cout << "Animal is eating." << endl;
    }
};

class Dog : public Animal { // Dog 继承 Animal
public:
    void bark() {
        cout << "Dog is barking." << endl;
    }
};

int main() {
    Dog dog;
    dog.eat();  // 子类继承父类方法
    dog.bark(); // 子类自己的方法
    return 0;
}
```

3. 多态Polymorphism 多态是指同一个接口可以表现出不同的行为，通常通过**函数重写Overriding和函数重载Overloading**实现    
函数重写（Overriding）：     
子类可以重写父类的方法，在运行时根据具体对象的类型调用相应的方法（即运行时多态）。使用**虚函数virtual**实现，基于动态绑定。
```cpp
class Animal {
public:
    virtual void makeSound() { // 父类中的虚函数
        cout << "Animal makes sound." << endl;
    }
};

class Dog : public Animal {
public:
    void makeSound() override { // 子类重写父类方法 也可以不写override
        cout << "Dog barks." << endl;
    }
};

class Cat : public Animal {
public:
    void makeSound() override {
        cout << "Cat meows." << endl;
    }
};

int main() {
    Animal* animal;
    Dog dog;
    Cat cat;

    animal = &dog;
    animal->makeSound(); // 输出: Dog barks.

    animal = &cat;
    animal->makeSound(); // 输出: Cat meows.

    return 0;
}
```
函数重载（Overloading）：
同名函数可以根据参数的不同（数量或类型）执行不同的行为（即编译时多态）。
```cpp
class Calculator {
public:
    int add(int a, int b) {
        return a + b;
    }

    double add(double a, double b) {
        return a + b;
    }
};

int main() {
    Calculator calc;
    cout << calc.add(2, 3) << endl;       // 调用第一个 add，输出: 5
    cout << calc.add(2.5, 3.5) << endl;   // 调用第二个 add，输出: 6
    return 0;
}
```
**总结**
封装	    隐藏实现细节，仅暴露接口	    提高代码安全性和可维护性	用于保护敏感数据，如银行账户、用户密码管理等
继承	    子类继承父类的属性和方法	    代码复用、减少重复代码、扩展功能	定义动物类的通用行为，子类实现具体行为
多态	    同一接口表现出不同行为	    提高代码的灵活性和可扩展性	运行时根据类型选择调用函数，如画图工具支持不同形状绘制
