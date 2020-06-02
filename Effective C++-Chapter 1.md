Chapter 1. 让自己习惯C++

## list 01: 视C++为一个语言联邦

 C++主要次语言：
 - C

- Object-Oriented C++----classes、封装、继承、多态、virtual函数（动态绑定）

- Template C++-----泛型编程

- STL------容器、迭代器、适配器、算法、函数对象...

## list 02 : 尽量以const、enum、inline 替换 #define 

----宁可以编译器替换预处理器

```c++
#define ASPECT_RATIO 1.653
const double AspectRatio = 1.653;
```

1. 单纯常量const、enum替换#defines:

- 定义常量指针

常量定义式通常放在头文件内

```C++
const std::string authorName("Scott Meyers")
```

- class专属常量

将常量作用域限制于Class内，必须使其成为class的member

```C++
  class GamePlayer{
  private:
      static const int NumTurns = 5;  //常量声明式，头文件内
     // enum { NumTurns = 5 };		//另NumTurns成为5的一个记号名称,不能获得
      //pointer或者reference指向整数常量     
      int scores[NumTurns];
  };

  const int GamePlayer::NumTurns;	//定义式，实现文件内  
```

2. 对于形式函数的宏，inline替换#defines

```C++
#define CALL_WITH_MAX(a,b) f((a) > (b) ? (a) : (b))	//所有实参加上(),容易产生错误
```

```C++
template<typename T>
inline void callWithMax (const T& a, const T& b)
{
    f(a > b ? a : b);
}
```

## list 03 ： 尽可能使用const

1. const可被施加于任何作用域内的**对象、函数参数、返回类型、成员函数本体**

- 指针

```C++
char greeting[] = "greeting";
char* p = greeting;
const char* p = greeting;	//const data
char* const p = greeting;	//const pointer
const char* const p = greeting;
```

‘**’左边是对象，*‘*’右边修饰指针

- 迭代器 (T* 指针)

```
c++ const
std::vector<int>::iterator iter;	//T* const,迭代器不可变，不能指向不同的东西
*iter = 10;		//指向值改变
++iter;			//error,改变迭代器
std::vector<int>::const_iterator Citer;	//迭代器所指的东西值不可改变
*Citer = 10;		//error,const ++Citer;			//改变Citer 
```

- 函数声明
```c++
class Rational{...};
const Rational			//防止赋值操作(a * b = c)
    operator*(const Rational& lhs, const Rational& rhs);

```

### const 成员函数

2. 可“操作const对象”，2个成员函数常量性不同，可被重载，使用概念性的常量性

```C++
class TextBlock{
public:
    ...
    const char& operator[](std::size_t position) const
    {
        return text[position];
    }
    char& operator[](std::size_t position)	//返回ref_to_char,pass by ref
    {
        return text[position];
    }
private:
    std::string text;    
};
const TextBlock ctb;
ctb[0] = 'x';		//error
```

> bitwise constness: const成员函数不可以更改**对象内的non-static成员变量**

> logical constness : const成员可以修改对象内bits，但只在客户端侦查不到情形下

```c++
class CTextBlock{
public:
	...
	std::size_t length() const;
private:
	char* pText;
	(mutable) std::size_t textLength;		//（2） mutable释放掉non-static成员变量bitwise 
	(mutable) bool lengthIsValid;			//constness 约束
};
std::size_t cTextBlock::length() const
{
    if(!lengthIsValid)
    {
        textLength = std::strlen(pText);		//（1）error!const成员函数内不能赋值修				LengthIsValid = true;					//	改对象内的non-static成员变量
    }
    return textLength;
}
```

3. 实现operator[]的机能一次并使用它2次，**用non-const调用另一个const**，通过进行**常量性转除**。
- 在const和non-const成员函数中避免重复

```c++
class TextBlock{
public:
    ...
    const char& operator[](std::size_t position) const
    { 
        ...
        return text[position];
    };
    char& operator[](std::size_t position)
    {
        return
            const_cast<char&>(							//op[]返回值const转除
            	static_cast<const TextBlock&>(*this)	//为*this(non-const对象)加上const
            		[position]);						//调用const op[]
    };  
};
```

## List 04 ：确定对象被使用前己先被初始化

- 为内置型对象进行手工初始化
- 构造函数最好使用成员初值列，而不要在构造函数本体内使用赋值操作

```c++
Triang( int len = 1, int bp =1 );

Member Initialiization list
Triang::Triang( const Triang &rhs )
    : _len( rhs._len ),				//_len = rhs._len 为赋值操作
      _beg( ths._beg ), _next( rhs._beg )
{ }
```

- 为避免“跨编译单元执之初始化次序”问题，请以local static 对象替换 non-local static对象

```c++
FileSystem& tfs()				//tfs()代替tfs
{
	static FileSystem fs;
	return fs;
}
```

