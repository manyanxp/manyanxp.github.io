[UP](../index.md)

# **<font color="#006e54">Singleton</font>**

```cpp

class Singleton
{
private:
    Singleton(){}
    virtual Singleton(){}

public:
    Singleton(const Singleton) = delete;
    void operator=(const Singleton&) = delete;

public:
    static Singleton& getInstance()
    {
        static Singleton instance;
        return instance;
    }    
};

```