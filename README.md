## AutoPtr(YuXin)

- 此类的构造较为简单**是个类模板**，是跨CPP版本的**最低支持C++11**，类似于`std::shared_ptr`，提交pr可在github或2416145262这个QQ提交

- 如果定义**STD_NAMESPACE**宏那么整个类都会在名字为这个宏的值的命名空间里

- 模板定义(C++17前):

    `template <typename T, typename D = Delete<T>>`
    对于C++17后的模板定义是

    `template <typename T, template <typename> typename D = Delete>`

- Delete类模板是默认的Delete方式，T是管理的类型，使用delete关键字进行delete，**它拥有数组的特化，所以不必担心数组是否正确释放的问题**

- 若要自定义delete类那么它**必须拥有一个任意返回类型的operator()**，参数是要delete的类型的指针，**且这个类型必须要有可用的默认构造且不抛出异常**

类型别名:
1. deleteClass delete的类类型，C++17前是D，C++17后是D\<T\>
2. pointToType 指向的类型，即T
3. elementType 元素类型，即`remove_extent<T>::type`
4. sizeType    size_t

构造:
1. `AutoPtr(T\* d = nullptr)` noexcept 初始化指向为d，引用计数为1，**C++14起constexpr**
2. `AutoPtr(AutoPtr& r)` noexcept 指向和引用计数指针初始化为r的，并++引用计数，**C++14起constexpr**
3. `AutoPtr(AutoPtr&& r)` noexcept 移动构造，虽说是移动构造不过和第二个构造的作用一模一样

成员函数:
1. `constexpr sizeType getCount() const noexcept` 返回引用计数
2. `constexpr T* getData() const noexcept` 返回管理的数据的裸指针
3. `template <typename U> constexpr bool operator==(const AutoPtr<U>& r) const noexcept` 判断指向是否与r相同
4. `constexpr bool operator==(void* e) const noexcept` 判断指向是否与e相同
5. `elementType& constexpr operator*() const noexcept` 返回\*getData()
6. `elementType& constexpr operator[](sizeType in) const noexcept` 返回第in个索引的元素的引用
7. `AutoPtr& operator=(T* d) noexcept` 如果d与当前指向相同则什么也不做否则将指向改变为d并重新申请引用计数的内存，若在此时引用计数为1就先释放引用计数的内存如果getData()不为空指针那么把管理的数据也一齐释放了，返回*this
8. `AutoPtr& operator=(AutoPtr& r) noexcept` 指向和引用计数改变为r的指向和引用计数，其它的同7
9. `AutoPtr& operator=(AutoPtr&& r) noexcept` 同8
10. `T* release() noexcept` **将指向的数据的指针设置为空指针**并重新申请引用内存，若此时引用计数为1那么就先释放引用计数的内存如果不为1就--引用计数，返回制空前的指向的裸指针

析构就是判断引用计数是否为1如果是那么就释放引用计数，如果指向不为空那么就释放它

需要注意的有:
1. 若不直接将指向设置为一个AutoPtr对象的指向而是间接的设置那么此类不能保证正确的释放内存
2. **对于管理的类型为void和不完整的数组类型T[]具有特化**，特化实现为空
3. **C++17起具有推导指引**: 初始化为指针和AutoPtr对象时
