# skr-lab
Record my study progress
<details>
  <summary>**1-2weeks**</summary>
## 1-2 weeks 5.14-5.28
1-2周主要学习了c++，由于我没有c++基础，就从c++ primer看起了，但是由于primer里细碎的知识点过多，所以c++的很多特性还不熟，有以下收获
- 模版和泛型编程初步掌握
- 面向对象编程中类的一些特性：
  - 类的构造、析构函数
  - 类的继承方式的区别，比如private public protected等
  - 深拷贝和浅拷贝的区别
    - 浅拷贝出现在类里没有定义拷贝构造函数的时候：
    1. 类作为函数参数传递时
    2. 类作为函数返回值传递时
    3. 一个新类A被B初始化时，如下代码
      ``` c
      
      class Test
    {
      private:
      int* p;
    public:
      Test(int x)
      {
          this->p=new int(x);
          cout << "create the object" << endl;
      }
      ~Test()
      {
        if (p != NULL)
        {
            delete p;
        }
        cout << "delete the object" << endl;
      }
      int getX() { return *p; }
    };

    int main()
    {
      Test a(10);
      //浅拷贝
      Test b = a;
      return 0;
     }
     
    ```
       浅拷贝只是复制字面值，也就是说，当复制指针的时候，只会复制那个地址，也就是说会有两个指向同一位置的指针，这样就就会造成uaf漏洞
       就比如上面的a和b里的指针值是一样的，最后程序结束将这些释放会double free。
      - 深拷贝
        只要自己定义好拷贝构造函数就可以了，在里面完成分配内存并初始化的操作。
- 写了一部分stl的源码，但是并没有用c++11等的新特性,只写完了hash table
  - github链接:https://github.com/y-f00l/f00l_tiny_stl
  ### some bugs in my syl
  - vector 
  	pop_back后end迭代器会前向移动一个单位，但是这里它并没有检查移动后的end是否超前于begin，这样如果多次对vector pop，那么end就会超出本vector的范围，那么就会发生越界读写。asan编译后抛出 heap overflow的警告

  - list
    是erase的锅，如果在你疯狂对list进行erase，在它为空的时候，里面会有一个head node，由于list是双向循环链表，这时head node就会指向它自己，此刻在进行erase，就会对head node进行析构，然后释放对应内存，但是list里的erase函数会返回一个指向erase的结点的后继结点的迭代器，这样我们会拿到一个指向已释放内存的指针，会造成uaf。asan编译后会抛use after free的警告。
    
    总结一下，这些漏洞的成因是都是bound check的遗漏，看了一下别的师傅找到的发现有迭代器的位置不正确，导致析构出现uaf。

  ### to do list
    - 考试周复习(忙完考试周变日更
    - 考完试写编译器
</details>
