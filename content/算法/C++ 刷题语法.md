### vector

```cpp
std::vector<int> numbers; // 声明一个空的整数向量

std::vector<std::string> names = {"Alice", "Bob", "Charlie"};
numbers.push_back(10); // 在向量尾部添加元素 10
numbers.push_back(20); // 在向量尾部添加元素 20

int firstElement = numbers[0]; // 使用索引访问第一个元素
int lastElement = numbers.back(); // 访问最后一个元素

for (const auto& element : numbers) {
    // 对每个元素进行操作
    std::cout << element << " ";
}
```

### map 、 unordered_map

```cpp
    // 基于 hash 表
    unordered_map<string, int> um;

    um["name"] = 1;

    um.insert(make_pair("age", 20));

		// 返回 pair , 当返回 um.emdl() 的时候，就是没有查找到
    cout << um.find("name")->second << endl;

    cout << um.count("name1") << endl;

    um.erase("age");

    cout << um.size() << endl;
```

```cpp
    // 基于红黑树 ， 保持有序性	
    map<string, int> mm;

    mm.insert(make_pair("name", 12));
    mm.insert(make_pair("aaa", 22));

		// 返回指针
		for (auto it = myMap.begin(); it != myMap.end(); ++it) {
        std::cout << "Key: " << it->first << ", Value: " << it->second << std::endl;
    }

		// 返回引用
		for (const auto& pair : myMap) {
        std::cout << "Key: " << pair.first << ", Value: " << pair.second << std::endl;
    }
		
		// 直接使用
		for(auto& [k,v]:cnt) {
            
    }
```

### set 、unordered_set

```cpp

    // 基于红黑树
	set<int> ss;
    ss.insert(10);
    ss.insert(2);

		// 查找元素
    cout << *ss.find(10) << endl;

    for (auto n: ss) {
        cout << n;
    }

    for (int n: ss) {
        cout << n;
    }

    ss.erase(10);
```

### 优先级队列 privority_queue

```cpp

		// 优先级队列
    priority_queue<int> pq;

    // 插入元素
    pq.push(12);
    pq.push(21);

    // 默认最大堆
    cout << "Top Element : " << pq.top() << endl;

    pq.push(3);

    while (!pq.empty()) {
        cout << pq.top() << endl;
        pq.pop();
    }

    // 最小堆
    priority_queue<int, vector<int>, greater<int> > min_pq;
    min_pq.push(30);
    min_pq.push(20);
    cout << min_pq.top() << endl; // 20

		// 基于一个 vector 构建最大堆
		priority_queue<int> q (nums.begin() , nums.end());
```

### stack

```cpp
    stack<string> sk;
    sk.push("anner");
    sk.push("bnner");

    while (!sk.empty()) {
        cout << sk.top() << endl;
        sk.pop();
    }
```

### queue

```cpp
   queue<int> qu;

    qu.push(10);
    qu.push(100);

    while (!qu.empty()) {
        cout << qu.front();
        qu.pop();
    }
```

### deque

```cpp
    deque<int> de;
    de.push_back(10);
    de.push_front(20);
    de.pop_front();
    de.pop_back();
    cout<<de.front()<<endl;
    cout<<de.back()<<endl;
```

### 结构转换

```cpp
// int 转 string
to_string(num);

// string 转 int
std::stoi(str)
stol stof stod
```

### 常用的内置函数

- abs
- min
- max
- sqrt
- ceil 不小于
- floor 不大于

### algorithm 库

- sort(begin , end)
- reverse(begin , end)
- find (begin , end , value)
- count (begin , end , value)
- next_permutation
- isdigit(c)
- isalpha(c)
- isupper(c)
- islower(c)
- to_lower(c)
- to_upper(c)
- max_element (begin , end)
- min_element(begin , end)

- [ ] 复习 c++刷题语法 