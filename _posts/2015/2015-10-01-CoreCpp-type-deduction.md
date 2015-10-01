---
layout: post
title: Core C++ series: type deduction
categories:
- C++
tags:
- C++template
---

#Core C++ series: type deduction�������Ƶ�/���

[TOC]

 

##1. �����Ƶ�����

```C++

/*Ex1*/

template <typename E, int N>

void f(E(&)[N]);

bool b[42];

f(b); // P=E(&)[N], A=bool [42] �Ƚ�P��A, �Ƶ���E=bool, N=42��P�е��������α������Ƶ��ķ��������鲻���˻�ת�ͳ�ָ�룩

 

/*Ex2*/

template<typename T1, typename T2, typename T3>

void g(T1 (T2::*)(T3*));// ָ����T2��Ա������ָ�룬���������T3, ����ֵ����T1

class S {

    public: void f(double*);

}

g(&S::f); // P=T1 (T2::*)(T3*)��A=(void)(S::*)(double*), �Զ����±ȶԸ������죬�Ƶ����:T1=void, T2=S, T3=doule

```

��������������C++11��ǰ��֧�ֵ������Ƶ�����C++11���Ժ�ı�׼�����������Ҫ�����Ƶ�����ʽ��

> C++98:  template type deduction(T&/T*, T)

> C++11:  template type deduction(T&&), auto object, decltype, lambda implicit return, lambda capture

> C++14:  auto return type, lambda auto parameter, decltype(auto), lambda init capture

 

##2. �����Ƶ��ķ���

###2.1 template type deduction

```C++

// һ�㻯�����

template<typename T>

void f(ParamType param);

f(expr); // expr��������ExprType

```

type deduction Ҫ���ľ��ǶԱ�ParamType ��ExprType���Ƶ���ģ�����T�����ͣ������ó�ParamType����ֵ��ע��ĵط����ڣ�`ԭ����ExprType������ֱ������ʹ�õģ�������Ҫ�����ģ����ҵ�����ʽ��ParamType����ʽ�й�`��

 

```C++

// �����const���ԣ�����volatile����Ӧ�Ľ���ͬ������

int val;

int& expr1 = val;

const vector<string> &  expr2;

const char* const expr3;

const char* const & expr4;

const char expr5[10];

f(string()); // expr6: string()����ֵ

```

ͨ��ParamType������������ʽ��

- ParamType is a reference or pointer, but not a universal refrence

- ParamType is a universal reference 

- ParamType is neither reference nor pointer

 

����������ʽ��ParamType��ExprType���������Զ�Ҫ��ȥ, ����ExprType1��int& Ҫ������int ��

####2.1.1 ParamType�����û�ָ�루��universal reference��

```C++

template<typename T>

void f(T& param);

f(expr);

 

// ����ԭ���ͣ�

ExprType1 => int

ExprType2 => const vector<string>

ExprType3 => const char* const

ExprType4 => const char* const

ExprType5 => const char [10]

ExprType6 // failed to compile for non-const reference bind to a rvalue

 

// ģ�����������T = ExprType

// ���������void f(const T& param), ֻ�����Ƶ�T��ȥ��ExprType��top-level const���Σ���ΪParamType���Ѿ�����const�����ˣ��������ϵ�T3 = const char*

```

��������£��ᱣ��ExprType��const/volatile���ԣ�Ҳ���ᷢ������/������ָ���ת�͡���ָ����������΢����㣬����ԭ����ExprType��const char* const p�����������const char*��top-level��const���λ��ǻᱻ�������뿴��������ӣ�

```C++

/*��ָ������const���Ա�������top-level��const���Ի�ȥ��*/

template<typename T>

void f(T* param);

f(expr);

 

// ����ԭ���ͣ�

ExprType1 => // failed to compile for no matching function

ExprType2 => // failed to compile for no matching function

ExprType3 => const char*  // T = const char

ExprType4 => const char*  // T = const char

ExprType5 => const char*  // T = const char

ExprType6 // failed to deduction

 

// ģ����������ͣ�T = (Exprtype ȥ��ָ�����)

```

####2.1.2 ParamType��universal reference

����ExprType�ĵ�����ʽ��T&�ĵ���һ����ֻ������Ҫ���⿼��expr����/��ֵ���ԣ�`���expr����ֵ����ParamType�Ƶ��������ֵ���ã����expr����ֵ����ParamType�Ƶ��������ֵ����`��Ϊ�˴ﵽ���Ч������Ҫ�������Ƶ�T��ʱ���������������ԣ�Ȼ������reference collapse����Ƶ��������ϸ�ڿ��Բο�[universal reference][1]/forward reference��

```C++

template <typename T>

void f(T&& param); // const T&&����ʽ����universal reference

f(expr);

 

// ����ԭ���ͣ�

ExprType1 => int

ExprType2 => const vector<string>

ExprType3 => const char* const

ExprType4 => const char* const

ExprType5 => const char [10]

ExprType6 => std::basic_string<char>

 

// ģ����������ͣ�T = ExprType & || T = ExprType ��

T1 => int & // expr����ֵ�����������������ԣ���T = ExprType &

T2 => const vector<string>& // ͬ��

T3 => const char* const &   // ͬ��

T4 => const char* const &   // ͬ��

T5 => const char(&)[10]     // ͬ��

T6 => std::basic_string<char> // expr ����ֵ��������������

```

####2.1.3  ParamType�Ǵ�ֵ����ʽ

������ʽ���Ƶ�������ȥ��ԭ����ExprType��const/volatile���Σ��������Ƶ�֮ǰ���ᷢ������/������ָ���ת�͡�

```C++

template<typename T>

void f(T param);

f(expr);

 

// ����ԭ���ͣ�const����ȥ�����ᷢ������/������ָ���ת��

ExprType1 => int

ExprType2 => vector<string>

ExprType3 => const char*

ExprType4 => const char*

ExprType5 => const char*

ExprType6 => std::basic_string<char>

 

// ģ����������ͣ�T = ExprType

```

**С��**���������T&/T&&���򾡿��ܱ������ʽ�������ԣ�const/volatile������ά�ȵȣ���������T��ʱ��ᾡ����ȥ����������(const/volatile, ����/������ת��)��universal reference�ῼ�Ǳ��ʽ����/��ֵ���ԣ���������������Ρ�����ο�[C++ Type Deduction and Why You Care CppCon2014][4]. auto type deduction��template type deduction�ķ������ƣ�Ҳ��������ʽ��`auto, auto&, auto&&`�������߻��Ǵ�������Ĳ�𣬱���template��auto����initializer_listʱ��

```C++

template<typename T>

void f(T param);

f({1, 2, 3, 4}); // error�����������Ƶ���T=initializer_list<int>����C++��׼����������ô����

 

auto x = {1, 2, 3, 4}; // auto �����Ƶ���x������Ϊinitializer_list<int>

auto x {1, 2, 3, 4}; // error

auto x {1}; // ok��x��������int

// ��ʵ�ϣ���initilizer_list��ʼ��һ��auto����ʱ������ÿ����������ʽ�����������Ƶ��������initializer_list<T>����Ҳ��Clang����������ʹ�õ���ʽ��

```

###2.2 decltype�������Ƶ�

decltype����˼������Ƶ���������ʱ�����ͣ�����`int& x = y; `decltype(x)�͸�Ϊint&��������ԭ���͵����ԣ�������C++11��׼�Ĺ涨��

> 1) If the argument is an unparenthesized id-expression or an

> unparenthesized class member access, then decltype yields the type of

> the entity named by this expression. If there is no such entity, or if

> the argument names a set of overloaded functions, the program is

> ill-formed. 

2) If the argument is any other expression of type T, and

>     - a) if the value category of expression is xvalue, then decltype yields T&&;

>     - b) if the value category of expression is lvalue, then decltype yields T&;

>     - c) if the value category of expression is prvalue, then decltype yields T.

 

��׼���Խ���Ϊ���ȿ����ʽ�ǲ���ĳ��id������x, classa.member_var��, ����ǣ���ֱ��ȡ����ʱ�����ͣ�������ǣ�����Ҫ����[lvalue, xvalue, pvalue][2]�������������͡�

```C++

//** use stantard item 1

int y = 1;

const int& x = y;

decltype(x) xType; // xType��������const int&

 

//** use stantard item 2-a

decltype(std::move(x)) xType; // xType��������const int&&

 

//** use stantard item 2-b

const vector<int> v(1);

decltype(v[0]) xType = 1; // operator[] ���ص�����ֵ��������, xType��const int&

decltype((y)) yType; // "(y)"����������ֵ����, yType��������int&, ע��decltype(y)�Ľ����int

 

//** use standard item 2-c

decltype(0) xType; // 0��pvalue, xType��������int

```

###2.3 ��������ֵ�����Ƶ�

```C++

auto lookupValue() // C++11��Ҫ��trialing return type

{

    static std::vector<int> values = {1, 2, 3, 4, 5};

    int idx = 3;

    return values[idx]; // lvalue reference

}

```

auto�������Ƶ���template by value��ʽ���Ƶ���ʽһ�£�ԭ���͵�reference/const����Ҫ�������˴���������ֵ������int����Ȼ��������ʽ��������Ϊauto&, auto&&���Ƶ���ʽ�ֱ���template���Ƶ���ʽһ�¡�T&&��auto &&���ܸ���ԭ�������/��ֵ�����������Ƶ��������/��ֵ���ã�����ԭ�����reference����ȴ�Ƕ�ʧ�ģ��������£�

```C++

// auto������������ֵ���͵�ȱ��

template<class Fun, class... Args>

auto fun_wrapper(Fun fun, Args&&... args)

{

    return fun(std::forward<Args>(args)...);

}

```

���fun(...)���ص�T&������auto���Ƶ�����fun_wrapper(...)�ķ���ֵ������T�����ܴﵽ����ת����Ŀ�ġ����������auto& fun_wrapper(...)����fun(...)�ķ���ֵ������T��fun_wrapper(...)�ķ�������ȴ��T&�����ɲ�������ת����

Ϊ�˿˷����ȱ�㣬C++14 ����decltype(auto)����ҪӦ��decltype�Ĺ������Ƶ�����֤fun_wrapper(...)�ķ���������fun(...)һ�¡�ע�⣬C++��׼ֻ������decltype(auto)�Ĺ���decltype(auto&)��decltype(auto&&)���ǷǷ���

```C++

// decltype(auto)

template<class Fun, class... Args>

decltype(auto) fun_perfect_wrapper(Fun fun, Args&&... args)

{

    return fun(std::forward<Args>(args)...);

}

 

// һ���Ƿ�ʹ�õ�����

decltype(auto) lookupValue()

{

    static std::vector<int> values = {1, 2, 3, 4, 5};

    int idx = 1;

    auto x = values[idx]; // auto �Ƶ��Ľ����int

    return (x);// "(x)"��������int&

}

// decltype(auto)�Ƶ��Ľ����int&

lookupValue() = 12; // ������Թ����������������ڴ��󣬶Ժ����ľֲ�����xд����

```

###2.4 lambda ���ʽ�е������Ƶ�

lambda�б�������������Ƶ����ӽ��������εķ�ʽ��������template/auto�������Ƶ���ʽ����Ҫע���һ���Ǳ�������ʱ�ᱣ��ԭ���͵�const/volatile���Ρ�

``` C++

// C++11���ô������ķ�ʽ������

const int cx = 0;

auto lambda_byval = [cx]() {}; // by value, ����ԭ���͵�cont/volatile����

auto lambda_byref = [cx&]() {}; // by reference

 

// C++14

auto lambda_init = [cy = cx]() {}; // by value ��ʼ������cy��������int��û�б���ԭ�����͵�const����

auto lambda_init_ref = [&ref = x](){};// by reference ��ʼ������x������const��������벻�������Ⲷ���в�����const& д����

 

auto lambda = [](auto x, auto y){return x+y; };//generic lambda������ʹ��auto�������Ƶ�

```

##3. ģ����������Ƶ�������

֮ǰ�����ݶ��������۵�һ(ParamType, ExprType)���Ƶ�����ʵ���о������������(ParamTypeN,  ExprTypeN)���Ƶ������⻹�����������޷������Ƶ����������Щ���漰��`�Ƶ�������`��

###3.1 ����Ƶ�

-  ÿ��(ParamType, ExprType)���ֱ�����Ƶ�������Ƶ����������г�ͻ������Ƶ�ʧ��

-  ֻ�в���ģ����������Ƶ�������ģ�����ʹ���Ѿ�ָ��/�Ƶ������ͣ����û�п�ʹ�õ����ͣ���ģ���Ƶ���ʧ�ܵġ�

```C++

/*Ex1 ����ì��*/

template <typename T> void f(T t1, T t2);

f(1, 1.2); // t1 �Ƶ���T = int��t2 �Ƶ���T = double������ì�ܣ��Ƶ�ʧ��

 

/*Ex2 ����ì��*/

template <typename T>

T const& max(T const& a, T const& b);

// ���max("Apple", "Pear"), �ͻᷢ������. ��Apple����ԭʼ������char[6]����Pear��ԭʼ������char[5], �β������������Σ����ܷ������鵽ָ��ת��(decay)��(A1,P1)(A2,P2)���������Ƶ��Ľ���ǳ�ͻ�ģ����Ƶ�ʧ��

 

/*Ex3 ʹ�����Ƶ��Ľ��*/

template <int N>

class X{

                public:

                                typedef int I;

                                void f (int) {}

};

 

template<int N>

void f_wrap(void (X<N>::*p)(typename X<N>::I )) {} // X<N>::I �޶�����߲��ֵĲ������Ƶ���

 

int main(){

f_wrap(&X<33>::f); // ok��X<N>::*p �Ƶ���N=33��Ȼ����뵽X<N>::I��

}

```

### 3.2 �Ƶ�������

��С�ڸ��������޷��Ƶ��������ģ�����Ĺ��򣬲ο�[Template argument deduction: Non-deduced contexts][3]

```C++

/*Ex1:*/

// ģ����������޶���::��ߣ��޷����������Ƶ����Ǹ��޷��Ƶ���������

template <typename T> struct identity { typedef T type; };


template <typename T>

void bad(std::vector<T> x, T value = 1);


template <typename T>

void good(std::vector<T> x, typename identity<T>::type value = 1);


std::vector<std::complex<double>> x;

bad(x, 1.2); // error, ExrpType1��ExrpType2���μ������Ƶ�����ExrpType1�Ƶ���T=std::complex<double>��

             // ExrpType2�Ƶ���T=double�����߳�ͻ�������Ƶ�ʧ�ܡ�

good(x, 1.2); // ok��ExrpType2����non-deduced contexts��ֻExrpType1�������Ƶ���

 

/*Ex2:*/

// �ӱ��ʽ������ģ�����

template<std::size_t N> void f(std::array<int, 2*N> a);

std::array<int, 10> a;

f(a); // error, std::array<int, 2*N>�е�2*N����std::array�ķ�����ģ����������ӱ��ʽ������f��ģ�����N��Υ����C++��׼��ֱ���Ͽ������ǿ����Ƶ���2*N=10����һ���Ƶ���N=5�������ź���׼�涨2*N���ڲ����Ƶ��������ġ�

 

/*Ex3*/

//����β����Ͳ�������/ָ�룬���˻���ָ�룬��һά�ȵ���Ϣ��ʧ���޷������Ƶ���

template<int i> void f1(int a[10][i]);

template<int i> void f2(int a[i][20]);

template<int i> void f3(int (&a)[i][20]); // &���Σ��ᱣ��ʵ�����͵��������Ͳ��䣨����ת��ָ�룩

void g() {

  int v[10][20];

  f1(v);            // ok: i deduced to be 20

  f1<20>(v);        // ok

  f2(v);            // error: cannot deduce template-argument i

  f2<10>(v);        // ok

  f3(v);            // ok: i deduced to be 10

}

 

/*Ex4*/

template<typename T> void f(T=5, T=7);

void g(){

f(1); //ok: call f<int>(1, 7)

f(); // error, Ĭ�ϲ��������������Ƶ�

f<int>(); // ok

}

```

����Ҳ�����ܹ��γ��Ƶ��������ģ����ֱȽ���������ӣ�

```C++

/*Ex1 ת�������ģ��*/

// user-defined conversion template based on return type

template<typename T>

using sid=T;

 

class S{

public:

    template<typename T, int N>

    operator sid<T(&)[N]> (); // // ���ܵ���д��operator T(&)[N]()�����벻��

};

 

void f(int(&)[20]) {}

void g(S s) { cout << typeid(f(s)).name(); }

 

/*Ex2 ���ൽ�����ת��*/

template <class T> struct B { };

template <class T> struct D : public B<T> {};


template <class T> void f(B<T>&){}


void f() {

    D<int> d;

    f(d); // ParamType��B<T>, ExprType��D<int>���Ƶ���T=int��D<int>��B<int>�ǳ�����

}

```

  [1]: https://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers

  [2]: http://en.cppreference.com/w/cpp/language/value_category

  [3]: http://en.cppreference.com/w/cpp/language/template_argument_deduction

  [4]: http://www.aristeia.com/TalkNotes/C++TypeDeductionandWhyYouCareCppCon2014.pdf