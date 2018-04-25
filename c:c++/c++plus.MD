#### ch 10 

````c++
class Stock{
    Stock( const std::string & company,int share,int share_val);
    ~Stock();// 析构函数
    private:
       std::string company;
       long shares;
       double share_val;
       double total_val;
       void set_hot() {
        total_val = shares * share_val;
    }
    public: 
    void acquire(cosnt std::string & co,long n ,dougle pr)
    void show();
    
}
````



* 内联方法 

   定义位于类声明中的函数都将自动成为内联函数 eg `Stock::set()`  是一个内联函数。

  编译器将使用函数的定义体来代替函数调用语句，这种行为发生在**编译阶段**不是在**运行阶段**。

* 方法使用哪个对象

   `Stock kata,joe` `kata.show() joe.show()`

  类的对象都有自己的存储空间，用于存储器内部的变量和类成员。但是同一个类的所有对象 ，共享同一组类方法。及每个方法只有一个副本。

* 构造函数 析构函数   

  `Stock stock = Stock("a",40,10)`

  `Stock  stock = Stock("a",20,10); ` 等价于 `Stock stock("a",10,10);`

  析构函数调用

  * 对象销毁前调用

   *C++11* 提供了列表初始化对象 `Stock stock = {"A",10,20};`

  `Const Stock land = Stock("wlecom  china");`

  `land.show()`

  c++ 拒绝第二行的调用，`Const` 表示对象不可变 ，那么所有改变对象的操作，都将不被允许  

  `void Stock::Show() const` 已这种方式定义的类函数-const成员函数。

* 作用域内枚举(*c++11*)

   

  ​