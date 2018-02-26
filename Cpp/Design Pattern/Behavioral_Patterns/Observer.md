[UP](../index.md)

# Observer

### Subject
```cpp
#include <iostream>
#include <vector>
using namespace std;

class Subject
{
private:
    vector<Observer*> m_observers;

public:
    Subject();
    virtual ~Subject();

public:
    virtual void Notify()
    {
        int count = m_observers.size();
        for(int i = 0; i < count; i++)
        {
            (m_observers[i])->Update(this);
        }
    }

    virtual void Attach(Observer* item)
    {
        views.push_back(item);
    }

    virtual void Detach(Observer* item)
    {
        int count = m_observer.size();
        for(int i = 0; i < count; i++)
        {
            if (m_observers[i] == item)
                break;
        }
    }
};
```

### Observer
```cpp
#include <iostream>
#include <vector>

using namespace std;

class Observer
{
public:
    Observer() {}
    virtual ~Observer(){}
    virtual void Update(Subject* target) = 0;
};

```