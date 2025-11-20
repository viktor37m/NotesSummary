# Реализация
Для удобства работы с векторами лучшего всего завести отдельную структуру данных, в который есть всё что нужно. Это будет удобной обёрткой, чтобы не хранить и не обрабатывать координаты вручную. Лучше всего для этого подходит создание отдельной *структуры*:

```c++
struct Point {
	int x, y;
	
	// Инициализация
	Point(int x = 0, int y = 0) : x(x), y(y) {}
}
```

Потом определить основные *арифметические* операции, *перегрузив* соответствующие операторы в структуре:

```c++
Point operator+(const Point &other) const {
	return Point(x + other.x, y + other.y);
}

Point operator-(const Point &other) const {
	return Point(x - other.x, y - other.y);
}
```

Для умножения нужен чуть другой подход. Если мы умножаем число слева на вектор справа $n \cdot \vec{v}$, то нужно добавить ключевой слово `friend`:

```c++
Point operator*(int other) const {
	return Point(x * other, y * other);
}

friend Point operator*(int other, const Point &point) {
	return point * other;
}
```

Для скалярного и векторного умножения можно определить функции, а можно перегрузить другие операторы:

```c++
// векторное произведение
int operator^(const Point &other) const {
	return x * other.y - y * other.x;
}

// скалярное произведение
int operator*(const Point &other) const {
	return x * other.x + y * other.y;
}
```

Для нахождения длины вектора лучше всего написать 2 функции: одна будет вычислять *квадрат длины*, и возвращать *целоичсленный* тип, а вторая будет определять просто длину, и возвращать число с *плавающей* точкой. Для чего это нужно? Числа с плавающей точкей имеет *погрешность*, которая со временем может *накапливаться*. Использовать вторую нужно только для формирования конечного ответа. Целочисленный тип погрешность не имеет, поэтому его можно использовать в операциях сравнения.

Основное правило: использовать целочисленную арифметику до тех пор, пока это *возможно*.

```c++
int len2() const {
	return x * x + y * y;
}

double len() const {
	return sqrt((double)len2());
}

// вместо if (a.len() < 5)
// используем if (a.len2() < 25)
```

Хорошо также определить операции сравнения для сортировки и некоторых задач (нахождение пересечения отрезков).

```c++
bool operator<(const Point& other) const {
	if (x == other.x) {
		return y < other.y;
	}
	return x < other.x;
}

bool operator==(const Point& other) const {
	return (x == other.x) && (y == other.y);
}

bool operator>(const Point& other) const {
	if (x == other.x) {
		return y > other.y;
	}
	return x > other.x;
}
```

Также определить ввод/вывод.

```c++
friend istream& operator>>(istream &in, Point &point) {
	return in >> point.x >> point.y;
}
  
friend ostream& operator<<(ostream &out, const Point &point) {
	return out << point.x << " " << point.y;
}
```

По хорошему, во всех случаях выше лучше использовать типы `long long` и `long double`, во избежания переполнения. Можно определить несколько разных типов вектора: целочисленный, с плавующей точкой и другие. Лучше всего это делается с помощью такой конструкции:

```c++
template<typename T> // <---
struct Point {
	T x, y;
	
	Point(T x = 0, T y = 0) : x(x), y(y) {}
	// ...
	
	T operator*(const Point<T> &other) const {
		return x * other.x + y * other.y;
	}
	
	// ...
	
	// однако для длины оставить как есть
	long double len() const {
		return sqrt((long double)len2());
	}
}
// Можно писать так: Point<long, long> a;
// Или определить несколько разных видов
typedef Point<long long> PointL;
typedef Point<long double> PointLD;
// PointLD a;
```

Напоследок определим несколько вспомогательных функций вне структуры:

```c++
int sign(long long x) {
	if (x < 0) return -1;
	if (x > 0) return 1;
	return 0;
}

// для вещественных чисел нужно сравнивать с учётом погрешности
const long double EPS = 1e-7;
int sign(long double x) {
	if (x < EPS) return -1;
	if (x > EPS) return 1;
	return 0;
}

// возвращает угол в радианах между векторами
// [-PI, +PI]
template<typename T>
long double angle(const Point<T> &a, const Point<T> &b) {
	return atan2(a ^ b, a * b);
}

// возвращает угол в градусах между векторами
// [-180, 180]
template<typename T>
long double degrees(const Point<T> &a, const Point<T> &b) {
	return angle(a, b) * 180.0 / M_PI;
}
```

По поводу углов между векторами. Мы помним, что `atan2` принимает числитель и знаменатель тангенса. Распишем векторное и скалярное произведение:

$$
\cfrac{\vec{a} \times \vec{b}}{\vec{a} \cdot \vec{b}} = \cfrac{|a| \cdot |b| \cdot \sin{\theta}}{|a| \cdot |b| \cdot \cos{\theta}} = \cfrac{\sin{\theta}}{\cos{\theta}} = \tan \theta
$$

Поэтому в функцию можно подавать векторное произведение (числитель) и скалярное произведение (знаменатель) в качестве параметров.