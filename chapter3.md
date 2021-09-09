# Effective C# - Chapter 3: 제네릭 활용

## Item 18: 반드시 필요한 제약 조건만 설정하라
타입 매개변수에 대한 제약 조건(constraint)은 최소한으로 설정해야 한다.
- 제약 조건이 없을 때는 런타임에 더 많은 검사를 수행하고, 더 자주 형변환을 해야하고, 잘못된 타입으로 인해 런타임 오류가 발생할 가능성이 높아진다.
- 불필요한 제약 조건을 설정하면 이 클래스를 사용하기 위해 과도하게 추가 작업을 해야 한다.

new 로 객체를 생성하는 방법 대신 default()를 사용하면 new() 제약 사항이 필요 없을 수도 있다. default() 연산자는 타입의 기본값을 가져온다. (값 타입에서는 0, 참조 타입에서는 null) 따라서 new()를 default()로 바꾸면 값 타입과 참조 타입에서 모두 사용할 수 있다.

## Item 19: 런타임에 타입을 확인하여 최적의 알고리즘을 사용하라
- 타입이나 메서드를 제네릭화하면 구체적인 타입이 주는 장점을 잃고 타입의 세부적인 특성까지 고려하여 최적화한 알고리즘을 사용할 수 없게 된다.
- 어떤 알고리즘이 특정 타입에 대해 더 효율적으로 동작한다면 그냥 그 타입을 이용하도록 코드를 작성하라.

(예제) ReverseEnumerable<T> 클래스 작성
- IEnumerable<T> 타입(랜덤 액세스 불가)과 IList<T> 타입(랜덤 액세스 가능)의 생성자를 별도로 작성한다.
- string 은 IList<char>를 구현한 것이 아니므로 (그렇게 보이지만) string에서 제공해주는 고유의 메서드를 사용하기 위해서는 특화된 코드를 작성한다.

## Item 20: IComparable<T>와 IComparer<T>를 이용하여 객체의 선후 관계를 정의하라
컬렉션을 정렬하거나 검색하려면 객체의 선후 관계를 판단할 수 있는 기능을 정의해야 한다.
IComparable<T>: 타입의 기본적인 선후 관계를 정의
IComparer<T>: 기본적인 선후 관계 이외에 추가적인 선후 관계를 정의할 수 있다.
  
최신 API들은 IComparable<T>를 사용하지만 오래된 API들은 여전히 IComparable을 사용한다. 따라서 IComparable<T>와 IComparable도 함께 구현해야 한다. 추가로 관계 연산자들을 오버로딩하면 더 빠르게 비교 연산을 수행할 수 있다.
```C#
public struct Customer : IComparable<Customer>, IComparable
{
  private readonly string name;
  public Customer(string name)
  {
    this.name = name
  }
  
  // IComparable<Customer> 멤버
  public int CompareTo(Customer other) => name.CompareTo(other.name);
  
  // IComparable 멤버
  int IComparable.CompareTo(object obj)
  {
    if (!(obj is Customer))
      throw new ArgumentException("Argument is not a Customer", "obj");
  
    return this.CompareTo((Customer)obj);
  }
  
  // 관계 연산자 오버로딩
  public static bool operator <(Customer left, Customer right) => left.CompareTo(right) < 0;
  public static bool operator <=(Customer left, Customer right) => left.CompareTo(right) <= 0;
  public static bool operator >(Customer left, Customer right) => left.CompareTo(right) > 0;
  public static bool operator <=(Customer left, Customer right) => left.CompareTo(right) >= 0;                                                                                            
}
```

IComparer 인터페이스는 IComparable에서 정의한 선후 관계 외에 추가적인 선후 관계 논리를 제공해야 하는 경우를 위해 표준화된 방법이다.
```C#
public struct Customer : IComparable<Customer>, IComparable
{
  private double revenue;
  
  ...
  
  private static Lazy<RevenueComparer> revComp = new Lazy<RevenueComparer>(() => new RevenueComparer());

  public static IComparer<Customer> RevenueCompare => revComp.Value;

  // revenue를 기반으로 Customer 객체를 비교하는 클래스
  private class RevenueComparer : IComparer<Customer>
  {
    int IComparer<Customer>.Compare(Customer left, Customer right) => left.revenue.CompareTo(right.revenue);
  }
```

## Item 21: 타입 매개변수가 IDisposable을 구현한 경우를 대비하여 제네릭 클래스를 작성하라
아래 코드에서 타입이 IDisposable을 구현하고 있다면 메모리릭이 발생할 수 있다. 
```C#
public interface IEngine
{
  void DoWork();
}

public class EngineDriverOne<T> where T : IEngine, new()
{
  public void GetThingsDone()
  {
    T driver = new T();
    driver.DoWork();
  }
}
```
  
IDisaposable을 구현하고 있다면 아래처럼 추가적인 처리가 필요하다.
```C#
  public void GetThingsDone()
  {
    T driver = new T();
    using (driver as IDisposable)
    {
      driver.DoWork();
    }
  }
```

타입 매개변수로 전달한 타입을 이용하여 멤버 변수를 선언한 경우에는 IDisposal을 구현하여 해당 리소스를 처리해야 하므로 복잡한 코드를 추가해야 한다. 매개변수로는 지역변수 정도만을 생성하는 것이 좋다.
  
## Item 22: 공변성과 반공변성을 지원하라
- 타입의 가변성(variance): 특정 타입의 객체를 다른 타입의 객체로 변환할 수 있는 성격을 말한다.
  - 공변성(covariance): X -> Y가 가능할 때 C<T>가 C<X> -> C<Y>로 가능하다면 이는 공변이다.
  - 반공변성(contravariance) : X -> Y가 가능할 때 C<T>가 C<Y> -> C<X>로 사용 가능하다면 이는 반공변이다.
- 가변성의 반대는 불변성(invariance)이라고 하는데, 제네릭은 기본적으로 불변이다. 제네릭의 공변/반공변을 지원하기 위해 데코레이터(decorator)를 추가해야 한다.
- 공변성
```C#
public interface IEnumerable<out T> : IEnumerable
{
  new IEnumerator<T> GetEnumerator();
}
```
IEnumerable<T>를 정의할 때 T에 대해 out 데코레이터가 사용되었는데, 이는 T를 출력 위치에서만 사용하겠다는 의미이다. 출력 위치는 다음과 같다.
  - 함수의 반환값
  - 속성의 get 접근자
  - 델리게이트의 일부 위치
  
공변성을 적용하면, 다음의 메서드에서 List<Derived> 타입의 객체를 인자로 전달이 가능하다.
```C#
public static void CovariantGeneric(IEnumerable<Base> baseItems)
{
  foreach (var thing in baseItems)
    Console.WriteLine($"{thing.var1} {thing.var2}");
}
```
  
- 반공변성
```C#
public interface IComparable<in T>
{
  int CompareTo(T other);
}
```
매개변수 타입 T에 in 데코레이터를 사용하면, T를 입력 위치에서 사용하겠다는 의미이다. 입력 위치는 다음과 같다.
  - 메서드의 매개변수
  - 속성의 set 접근자
  - 델리게이트의 일부 위치
  
반공변성을 적용하면, 다음의 예제처럼 Base 인스턴스와 Derived 인스턴스의 비교가 가능하다.
```C#
IComparable<Base> baseObject = new Base();
baseObject.CompareTo(new Derived());
```

## Item 23: 타입 매개변수에 대해 메서드 제약 조건을 설정하려면 델리게이트를 활용하라
타입 매개변수에 제약 조건을 설정하는 방법
  1. 베이스 클래스의 타입이나 특정 인터페이스로 제약 조건 설정
  2. class 타입이나 struct 타입으로 형태를 제한
  3. new() 키워드를 사용해서 매개변수가 없는 생성자를 가져야 하는 조건

임의의 정적 메서드를 반드시 구현해야 하는 제약 조건을 설정하기 위해서는 복잡한 작업이 필요하다. 예를 들어, 2개의 T 객체를 더하는 메서드가 반드시 구현되야 하는 제약 조건을 설정하기 위해 IAdd<T> 인터페이스를 정의하고, 구현하는 클래스를 생성하고, Add() 메서드를 구현해야 한다.
이 방법보다는 제약 조건으로 설정하고 싶은 메서드의 델리게이트를 작성하는 것이 좋다. 아래에서는 System.Func<T1, T2, TOutput> 델리게이트를 사용했다.
```C#
public static class Example
{
  public static T Add<T>(T left, T right, Func<T, T, T> AddFunc) => AddFunc(left, right);
}
```
이 클래스의 사용자는 다음처럼 람다 표현식을 이용하여 제네릭 클래스가 호출할 AddFunc 메서드를 정의하면 된다.
```C#
int a = 6;
int b = 7;
int sum = Example.Add(a, b, (x, y) => x + y);
```

Enumerable.Zip 이라는 메서드도 비슷한 방식으로 구현이 되었다.
```C#
public static IEnumerable<TOutput> Zip<T1, T2, TOutput>(IEnumerable<T1> left, IEnumerable<T2> right, Func<T1, T2, TOutput> generator)
{
  IEnumerator<T1> leftSequence = left.GetEnumerator();
  IEnumerator<T2> rightSequence = right.GetEnumerator();
  
  while (leftSequence.MoveNext() && rightSequence.MoveNext())
  {
    yeild return generator(leftSequence.Current, rightSequence.Current);
  }
  
  leftSequece.Dispose();
  rightSequece.Dispose();
}
```

## Item 24: 베이스 클래스나 인터페이스에 대해서 제네릭을 특화하지 말라
컴파일러는 제네릭 메서드의 타입 매개변수가 다른 타입으로 다양하게 변경될 수 있음을 고려하여 오버로드된 메서드 중 하나를 선택한다.

```C#
public class Base {}

public class Derived : Base {}

class Program
{
  static void Overloaded(Base b);
  static void Overloaded<T>(T obj);
  
  static void Main(string[] args)
  {
    Derived d = new Derived();
    Overloaded(d);        // Overloaded<T>(T obj)가 Overloaded(Base b)보다 우선적으로 호출된다. T가 Derived로 대체되면 정확히 일치하기 때문이다.
    Overloaded((Base)d);  // 명시적 형변환에 맞게 Overloaded(Base b) 가 호출된다.
  }
}
```
이러한 동작 방식때문에 베이스 클래스와 파생 클래스에 대해서 모두 수행 가능하도록 하기 위해서 제네릭을 특화(specialization)하려는 시도는 바람직하지 않다. 특히 인터페이스에 대해서 제네릭을 특화하면 오류가 발생할 가능성이 너무 높다. 제네릭 특화가 아니라 타입 매개변수로 지정할 수 있는 타입별로 각각 특화된 코드를 작성하는 편이 나을 수도 있다. 다음이 그러한 예제이다.
```C#
static void WriteMessage<T>(T obj)
{
  if (obj is MyBase)
    WriteMessage(obj as MyBase);
  else if (obj is IMessageWriter)
    WriteMessage((IMessageWriter)obj);
  else
    WriteLine(obj.ToString());
```

[완벽히 이해하지는 못해서 나중에 더 공부가 필요함]
  
## Item 25: 타입 매개변수로 인스턴스 필드를 만들 필요가 없다면 제네릭 메서드를 정의하라

  
## Item 26: 제네릭 인터페이스와 논제네릭 인터페이스를 함께 구현하라
  
## Item 27: 인터페이스는 간략히 정의하고 기능의 확장은 확장 메서드를 사용하라
  
## Item 28: 확장 메서드를 이용하여 구체화된 제네릭 타입을 개선하라
  
