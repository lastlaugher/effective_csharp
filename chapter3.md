# Effective C# - Chapter 3: 제네릭 활용

## Item 18: 반드시 필요한 제약 조건만 설정하라
타입 매개변수에 대한 제약 조건(constraint)은 최소한으로 설정해야 한다.
- 제약 조건이 없을 때는 런타임에 더 많은 검사를 수행하고, 더 자주 형변환을 해야하고, 잘못된 타입으로 인해 런타임 오류가 발생할 가능성이 높아진다.
- 불필요한 제약 조건을 설정하면 이 클래스를 사용하기 위해 과도하게 추가 작업을 해야 한다.

new 로 객체를 생성하는 방법 대신 default()를 사용하면 new() 제약 사항이 필요 없을 수도 있다. default() 연산자는 타입의 기본값을 가져온다. (값 타입에서는 0, 참조 타입에서는 null) 따라서 new()를 default()로 바꾸면 값 타입과 참조 타입에서 모두 사용할 수 있다.

## Item 19: 런타임에 타입을 확인하여 최적의 알고리즘을 사용하라
- 타입이나 메서드를 제네릭화하면 구체적인 타입이 주는 장점을 잃고 타입의 세부적인 특성까지 고려하여 최적화한 알고리즘을 사용할 수 없게 된다.
- 어떤 알고리즘이 특정 타입에 대해 더 효율적으로 동작한다면 그냥 그 타입을 이용하도록 코드를 작성하라.

(예제) ReverseEnumerable<T> 클래스를 작성
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
  
## Item 22: 공변성과 반공변성을 지원하라
  
## Item 23: 타입 매개변수에 대해 메서드 제약 조건을 설정하려면 델리게이트를 활용하라
  
## Item 24: 베이스 클래스나 인터페이스에 대해서 제네릭을 특화하지 말라
  
## Item 25: 타입 매개변수로 인스턴스 필드를 만들 필요가 없다면 제네릭 메서드를 정의하라
  
## Item 26: 제네릭 인터페이스와 논제네릭 인터페이스를 함께 구현하라
  
## Item 27: 인터페이스는 간략히 정의하고 기능의 확장은 확장 메서드를 사용하라
  
## Item 28: 확장 메서드를 이용하여 구체화된 제네릭 타입을 개선하라
  
