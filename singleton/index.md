# C++设计模式（一）：单例模式


单例模式是一种创建性设计模式，能够保证一个类只有一个实例，并提供一个访问该实例的全局节点。

### 适用场景
- 程序的某个类对于所有客户端只有一个可访问的实例。如操作系统中的文件系统；
- 创建一个对象需要消耗的资源过多，可以使用单例模式。如程序与数据库的连接。

### 实现方式
- 默认的构造函数，拷贝构造函数，赋值构造函数声明为私有，禁止再类外部创建该对象；
- 在类中添加一个私有静态成员变量用于保存单例实例；
- 声明一个公有静态方法来获取单例实例。

### 优缺点
| 优点 | 缺点|
| :--- | :---|
| <li> 保证一个类只有一个实例和一个全局访问点，减少了内存开销；<li> 避免了对资源的多重占用。|<li> 违反了单一职责原则，该模式同时解决了两个问题；<li> 没有接口，无法继承。

### 实现
- 懒汉模式：直到第一次用到类的实例时才实例化。

```C++
// 线程安全的懒汉模式实现 
  
class Singleton {  
private:  
    static Singleton* instance;  
    static std::mutex mutex_;  
  
    Singleton() {}  
    Singleton(const Singleton&) = delete;  
    Singleton& operator=(const Singleton&) = delete;  
  
public:  
    static Singleton* getInstance() {  
        std::lock_guard<std::mutex> lock(mutex_);  
  
        if (instance == nullptr) {  
            instance = new Singleton();  
        }  
  
        return instance;  
    }  
};  
  
Singleton* Singleton::instance = nullptr;  
std::mutex Singleton::mutex_;
```
```C++
// 智能指针实现
  
class Singleton {  
public:  
    static std::shared_ptr<Singleton> getInstance() { 
        // 确保初始化函数只会被调用一次，即使是多线程环境
        std::call_once(initInstanceFlag, &Singleton::initSingleton);  
        return instance;  
    }  
  
private:  
    Singleton() {}  
    ~Singleton() {}  
  
    static void initSingleton() {  
        instance = std::make_shared<Singleton>();  
    }  
  
    static std::shared_ptr<Singleton> instance;  
    // std::once_flag变量用于标识初始化函数是否已经被调用过
    static std::once_flag initInstanceFlag;  
};  
  
std::shared_ptr<Singleton> Singleton::instance = nullptr;  
std::once_flag Singleton::initInstanceFlag;
```
- 饿汉模式：类定义时就初始化
``` C++
// 线程安全的饿汉模式实现，线程安全不用加锁
class Singleton {  
private:  
    static Singleton instance;  
  
    Singleton() {}  
    Singleton(const Singleton&) = delete;  
    Singleton& operator=(const Singleton&) = delete;  
  
public:  
    static Singleton& getInstance() {  
        return instance;  
    }  
};  
  
Singleton Singleton::instance;
```
```C++
// 智能指针实现
class Singleton {
public:
    static std::shared_ptr<Singleton> getInstance() {
        return instance;
    }

private:
    Singleton() {}
    Singleton(const Singleton&) = delete;  
    Singleton& operator=(const Singleton&) = delete;  
    static std::shared_ptr<Singleton> instance; // 静态智能指针

    // 在类内部初始化单例对象
    static std::shared_ptr<Singleton> initializeInstance() {
        return std::make_shared<Singleton>();
    }
};

// 在类外部初始化静态成员变量
std::shared_ptr<Singleton> Singleton::instance = Singleton::initializeInstance();
```
