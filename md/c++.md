# c++


## 编译

使用c11标准

```shell
g++ -std=c++11
```

## 指针

指针的加减法，是根据指针的类型移动指定长度地址，
例如
```
#include<iostream>

using namespace std;
int main()
{
        int a[2] = {1,3};
        int *p = a;
        cout << p << " " << *p << endl;
        p++;
        cout << p << " " << *p << endl;

}

```
输出结果如下
> 0x7ffe7de83150 1
0x7ffe7de83154 3

## 函数

### 注意事项

1. <hi>不要返回指向局部变量或临时对象的引用（指针），因为当函数执行完毕后，局部变量和临时对象将小时，引用将执行不存在的数据。</hi>
2. 函数参数可以有默认值`Klunk(int n=10){klunk_ct=n;}`



### 类自动函数

`c++`会默认为类提供一些成员函数

- 默认构造函数，如果没有定义默认构造函数
- 默认析构函数（类销毁时调用，默认析构函数不做任何操作），如果没有定义
- 复制构造函数，如果没有定义
        类的复制函数的原型通常为，它接受一个指向类对象的常量引用作为参数，
        ```c++
        Class_name(const Class_name &);）
        ```
        每当程序生成了对象的副本时，编译器都将使用复制构造函数。具体地说，当函数按值传递对象或函数返回对象时，都将使用复制构造函数。<hi>按值传递意味着创建原始变量的一个副本</hi>， 编译器生成临时对象时，也将使用复制构造函数。例如，将3个Vector对象相加时，编译器可能生成临时的Vector对象来保存中间结果。何时生成临时对象随编译器而异。
        
    ```c++
   StringBad sail = sport;
   //与下面的代码等效
    StringBad sail; 
    sail.str = sport.str;
    sail.len= sport.len;
    ```
- 赋值运算符（将一个对象赋给另外一个对象），如果没有定义
- 地址运算符，如果没有定义

### 函数符重载
函数符重载一般是以`符号`左边的对象的函数`operator<XX>`进行重写，当对象在`符号`右边时，这种方式就无效了，需要通过友元函数的方式来实现，例如重载`cout`的`<<`运算符，通过友元函数（友元函数不属于类函数）的方式去实现
```c++
#include <iostream>

using namespace std;

class Time 
{
public:
        int hour = 0;
        friend ostream& operator <<(ostream& os,const Time& t); 
        friend Time operator +(int plus,const Time& t); 
};


ostream& operator <<(ostream& os, const Time& t) 
{
        os <<  (t.hour);
        return os;
}

Time operator +(int plus,const Time& t)
{
        Time result;
        result.hour = plus + t.hour;
        return result;
}

int main(void)
{
        Time time ;
        cout << time << endl;
        time = 10+ time;
        cout << time << endl;
}
```

### 转换函数

`operator typename();`
- 转换函数必须是类方法
- 转换函数不能指定返回类型
- 转换函数不能有参数

```c++
class Stonewt
{
private:
    int stone;
public:
    Stonewt(int stone)
    {
        this -> stone = stone;
    }   
    operator int() const
    {
        return this -> stone;
    };    
};


int main()
{
    Stonewt st = 100;
    int stone = st;
}
```

## 字符串

### raw字符串

```c++
//以R来标识原始字符串，"(和)"用作界定符,界定符可以在"和括号直接添加任意数量的基本字符,例如 "+*(和)+*"
cout << R"( fuck "high",\n)" << endl;
```
## 数组

```
int arr[3] = {1,2,3};
//数组名是一个指针,以下三种方式输出的都是数组的地址
cout << arr     << endl;
cout << &arr    << endl;
cout << &arr[0] << endl;

//所有可以使用解指针的方式取得0位置的值
cout << *arr << endl;

//以下都可以表示方式第二个元素
cout << arr[1]   << endl;
cout << *(arr+1) << endl;


//对于struct结构的数组，访问成员时需要使用间接成员运算符

struct things 
{
   char  *name
};

things t[3];

things *p = t;

cout << p -> name << endl;
cout << (*p).name << endl;


```
## 标准库

### cout

当打印指针时，一般会打印指针的地址，但如果指针的类型为`char *`时，则cout将显示指向的字符串。
若要显示字符串的地址，则需将其强制转换为另一种指针类型。
```c++
char * ps = "hello";

cout << ps << " at " <<  (int *) ps << endl;

//hello at 0x400951
```

### sizeof

```c++
//当表示数组时，其表示数组的长度，当表示指针时其表示指针的长度

int arr[5] = {1,2,3,4,5} 

cout << sizeof arr << endl;

//5

void test(int arr[]){
    cout << sizeof arr << endl;
}

test(arr);
//此时方法内部返回的则为8（指针长度根据系统而异)

```


##  常见编译错误

- `error: expected initializer before ‘&’ token` 一般是因为前面的struct没加`;`结尾
- `Segmentation fault` 函数返回了一个不存在的内存地址，比如函数申明为返回引用类型，实际返回一个临时变量。



## 其他

在赋值语句中，左边必须是可修改的左值，也就是说，在赋值表达式中，左边的子表达式必须标识一个可修改的内存块。