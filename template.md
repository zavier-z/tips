## c++ template

### immediate context
* function template - validity of the template parameter list, function signature and return type.
* class template - validity of the template parameter list

### type function
* is_same, s_same_v
```cpp
template<class T, class U>
struct is_same : std::false_type {};
std::is_same<std::int64_t, long>::value // true
std::is_same<std::int32_t, int>::value  // true
// cv qualifier is a different type

template<class T>
struct is_same<T, T> : std::true_type {};

template< class T, class U >
inline constexpr bool is_same_v = is_same<T, U>::value;
```

* enable_if, enable_if_t
```cpp
template<bool B, class T = void>
struct enable_if {};
 
template<class T>
struct enable_if<true, T> { typedef T type; };

template< bool B, class T = void >
using enable_if_t = typename enable_if<B,T>::type;
```

### value template
```cpp
//primary template class
template <typename T>
struct st_integer{
	static const bool value = false;
};
//full specialization
template <>
struct st_integer<int> {
	static const bool value = true;
}
template <>
struct st_integer<unsigned int> {
	static const bool value = true;
}
//... char, short, long, long long for signed and unsigned integer type
// value template using constexpr keyword
template <typename T>
constexpr bool is_integer = st_integer<T>::value;
```

### type alias
```cpp
// using template - type alias
template<typename T>
using integer_t = typename st_integer<T>::type;
```
模版中，typename关键字用于显式标识类型。由于表达式中包含域作用符“::”，其可访问类静态成员对象，静态成员方法或alias类型。

编译器有时不能区分这三者，因此使用typename可表明其为一种类型，而非对象或函数。

### class template and member function template
* SFINAE out member function template
```cpp
template<typename T, 
		typename = integer_t<T> // this SFINAE out any non-integral types
>
class Modulo{
protected:
	T m_mem;
public:
	// SFINAE out any non-integer type of S
	// if S is not integral type, then this funtion disappears at compile time
	template <typename S, typename integer_t<S>>
	std::enable_if_t<std::is_same_v<S, T>, S> operator%(S m) {
		return m_mem % m;
	}
	//...
};
```
当类成员函数需要特化参数，对参数类型做进一步的限制时，需要声明此函数为member template。

此处，modulo operator% 希望传入参数类型S，一定要为integer_t，同时要求传入参数S和T类型相同。

### template template type
```cpp
template <template <typename, typename> class Cntr, typename T, typename Allocator>
void print(Cntr<T, Allocator> container) {
	for(const T& v : container) {
		std::cout << v << ",";
	}
	std::cout << std::endl;
}

// usage
vector<int> v;
print(v);
```

### SFINAE
SFINAE: substitution failure is not an error.
当template parameter list替换为显式指定的类型或推导类型template argument list失败时，从重载集合中丢弃这个特化，而非导致编译失败。
```cpp
template <typename T>
struct st_integer {
	static const bool value = false;
};
template <>
struct st_integer<int> {
	static const bool value = true;
	using type = int;
}
//...
template<typename T>
using integer_t = typename st_integer<T>::type;

// SFINAE out any non-integral types
template <typename T, typename = integer_t<T>>
class Mudulo;
```
integer_t已经特化了所有内置整型类型，包括带符号和无符号的char、short、int、long和long long。

对于未被特化的类型，如double，float等，其st_integer未定义type，因此integer_t拿不到相应的type。

对于不合法的template argument类型，将产生SFINAE failure，它不是一种错误（error）。

### template parameter and argument
* template parameter list表示定义模版时，指定的模版参数；
* template argument list表示模版实例化时，传入的模版参数；

### template argument match rule
如果template arguments同时匹配primary template和specialization，那么更加特化的版本是最佳匹配，且不会报错。

### const and constexpr
* const对象可在编译期或者运行时初始化，但constexpr只能在编译期求值；
* const对象可在不同的编译单元共享，但constexpr不行；
* const和constexpr的共享性在于能否使用extern在头文件中声明对象，使其在其他编译单元可见。
```cpp
int rt_gen_ten() { return 10; }
constexpr int ct_gen_ten() { return 10; }

// initialize const int n1 at run-time
const int n1 = rt_gen_ten(); 
// initialize const int n2 at compile-time
const int n2 = ct_gen_ten();
```

### non-type template parameter
non-type template parameter的模板并不是没有参数，而是模板参数是指定类型。
他们可能是integer type，enum type、lvalue ref、pointer或者浮点类型。
利用non-type模板，我们可以在编译期做一些计算工作。
* 求和
```cpp
// primary template
template <unsigned n>
struct st_sum {
    static const unsigned value = n + st_sum<n-1>::value;
};
template <>
struct st_sum<1> {
    static const unsigned value = 1;
};

// usage
cout << st_sum<10>::value << endl;
```
求和操作使用了递归，特别注意需要一个递归退出时的特化版本。

* 素数判断
正常运行期判断素数，递归版本大致是，
```cpp
bool is_prime_recu(unsigned p, unsigned d) {
	if(d == 2) return (p % d != 0); // p is not divisible by d, then return true
	else {
		if(p % d != 0) return is_prime_recu(p, --d); // p is not divisible by d
		else return false; // p is divisible by d
	}
}
bool is_prime_recusion(unsigned p) {
	if(p < 4) return p>1; //p=2 or 3, then return true
	else return is_prime_recu(p, p/2);
}
```

当使用non-type模板时，也可以看做是翻译上述过程。
```cpp
//primary template 
template <unsigned p, unsigned d>
struct st_is_prime_recu {
    static const bool value = (p%d) && st_is_prime_recu<p, d-1>::value;
};
template <unsigned p>
struct st_is_prime_recu<p,2> {
    static const bool value = p%2;
};
template <unsigned p>
struct st_is_prime {
    static const bool value = st_is_prime_recu<p, p/2>::value;
};
template <>
sturct st_is_prime<3> {
    static const bool value = true;
};
template <>
sturct st_is_prime<2> {
    static const bool value = true;
};
template <>
sturct st_is_prime<1> {
    static const bool value = false;
};
template <>
sturct st_is_prime<0> {
    static const bool value = false;
};
```
non-type template最大使用场景是编译器使用递归求值。
除了使用上述方法，我们也可以使用constexpr function来实现。
```cpp
constexpr bool is_prime_recu(unsigned p, unsigned d) {
	return (d == 2) ? (p % d) : (p % d) && is_prime_recu(p, --d);
}
constexpr bool is_prime_recursion(unsigned p) {
	return p < 4 ? (p > 1) : is_prime_recu(p, p/2);
}
```

### structure binding
结构绑定适用于pair，tuple，数组，initializer_list等。
```cpp
auto t = make_tuple(19, 'F', "Ma");
auto [age, gender, name] = t;

```
 










