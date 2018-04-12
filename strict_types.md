

# Types as a Negative Cost Abstraction

# Strict Typedef

* Template Wrapper um (primitiven) Typ
* aka. Named Value
* Domain Driven Design

# Code

~~~{.cpp}
template<class T, class tag>
class NamedValue {
public:
	/// Require explicit conversion from original value
	explicit NamedValue(T v) :
			value { std::move(v) } {
	}

	NamedValue() = default;

	/// short call to get original value
	T get() const {
		return value;
	}
private:
	T value;
};
~~~
----
~~~{.cpp}
struct height_t {};
struct widh_t {};
using Height = NamedValue<double, height_t>;
using Width = NamedValue<double, widh_t>;
 
auto a = Rectangle{Height{1.2}, Width{3.4}};
~~~
 
# Codegen

~~~{.cpp}
struct height_tag {};
struct width_tag {};
struct area_tag{};
using Height = NamedValue<double, height_tag>;
using Width = NamedValue<double, width_tag>;
using Area = NamedValue<double, area_tag>;

void area(const double& x, const double& y, double& area, double& area2){
    area = x*y;
    area2 = x*y;
}
void area(const Width& x, const Height& y, Area& a,  Area& a2){
    a = Area{ x.get() * y.get() };
    a2 = Area{ x.get() * y.get() };
}
~~~
 
# Results

~~~{.asm}
area(double const&, double const&, double&, double&):
  movsd xmm0, QWORD PTR [rdi]
  mulsd xmm0, QWORD PTR [rsi]
  movsd QWORD PTR [rdx], xmm0
  movsd xmm0, QWORD PTR [rdi]
  mulsd xmm0, QWORD PTR [rsi]
  movsd QWORD PTR [rcx], xmm0
  ret
area(NamedValue<> const&,NamedValue<> const&,NamedValue<>&,NamedValue<>&):
  movsd xmm0, QWORD PTR [rdi]
  mulsd xmm0, QWORD PTR [rsi]
  movsd QWORD PTR [rdx], xmm0
  movsd QWORD PTR [rcx], xmm0
  ret
~~~
# Fazit

* Lesbarerer Code
* Weniger Bugs
* Schnellerer Code
* Benchmarks are hard

# Further Reading

* Beispielimplementierung: https://github.com/CasparKielwein/cpp-bubbles/
* Boost Units: https://www.boost.org/doc/libs/1_65_0/doc/html/boost_units.html
* experiment on compiler explorer: https://godbolt.org/g/9VJSbC
