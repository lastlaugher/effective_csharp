# Effective C# Chapter2: .NET 리소스 관리
## Item 11: .NET 리소스 관리에 대한 이해
- 가비지 수집기(garbage collector)
  - 관리되는 메모리(managed memory)를 관장하며 메모리 누수(memory leak), 댕글링 포인터(dangling pointer), 초기화되지 않은 포인터, 여타의 메모리 관리 문제를 개발자들이 직접 다루지 않도록 자동화 해준다.
  - 마크/콤팩트(mark/compact) 알고리즘은 여러 객체 사이의 연관 관계를 효율적으로 파악하여 더 이상 사용됮 않는 객체를 자동으로 제거한다. 객체가 스스로 자신의 참여 횟수를 관리하도록 하지 않고, 응용프로그램의 최상위객체로부터 개별 객체까지의 도달 가능 여부를 확인하도록 설계되어 있다.
  - 사용 중인 객체들을 한쪽으로 차곡차곡 옮겨서 조각난 가용 메모리를 단일의 큰 메모리 공간으로 만드는 콤팩트(compact) 작업을 수행한다.
- 비관리 리소스는 여전히 개발자가 관리해야 한다. finalizer와 IDisposable 두 가지 방법을 제공한다.
- finalizer는 리소스에 대한 해제 작업이 반드시 수행될 수 있도록 도와주는 방어적인 매커니즘이다. 하지만 finalizer를 가지고 있는 객체는 가비지로 간주된 이후에도 꽤 긴 시간동안 메모리를 점유하게 되는 단점이 있다.
- 가장 좋은 방법은 IDisposable 인터페이스와 표준 Dispose 패턴을 활용하는 것이다. [Item 17: 표준 Dispose 패턴을 구현하라](#item17-dispose-pattern)

## Item 12: 할당 구문보다 멤버 초기화 구문이 좋다
생성자 내에서 멤버 변수에 값을 할당하기보다 멤버 초기화 구문(member initializer)를 사용하는 것이 좋다. 
```c#
public class MyClass
{
  private List<string> labels = new List<string>();
}
```
새로운 생성자를 추가하더라도 멤버 초기화 구문이 항상 포함되기 때문에, 새로운 생성자에서 별도로 멤버변수를 초기화할 필요가 없다.

다음의 경우에는 멤버 초기화 구문을 사용하지 않는 것이 좋다.
- 객체를 `0`이나 `null`로 초기화 하는 경우는 명시적으로 초기화할 필요가 없다.
- 멤버 초기화 구문은 객체 생성 방법이 모든 생성자에서 동일한 경우에만 사용하는 것이 좋다.
- 멤버 초기화 구문은 try로 감쌀 수 없으므로, 예외 처리가 반드시 필요한 경우에는 생성자 내부에 작성해야 한다.


## Item 13: 정적 클래스 멤버를 올바르게 초기화하라
정적 멤버 변수를 포함하는 타입이 있다면 인스턴스를 생성하기 전에 반드시 정적 멤버 변수를 초기화해야 한다. 이를 위해 C#에서는 다음의 두 가지 기능을 제공한다.
- 정적 멤버 초기화 구문: 간단한 경우라면 정적 생성자 대신 사용하는 것이 좋다.
```C#
public class MySingleton
{
  private static readonly MySingleton theOneAndOnly = new MySingleton();
  
  public static MySingleton TheOnly
  {
    get
    {
      return theOneAndOnly;
    }
  }
  
  private MySingleton()
  {
  }
}
```

- 정적 생성자: 모든 메서드, 변수, 속성에 최초로 접근하기 전에 자동으로 호출되는 특이한 메서드이다. 다른 언어에서 정적 멤버를 초기화할 때 겪는 어려움을 언어 차원에서 해결해주기 위해 추가되었다. 예외 처리가 필요한 경우에는 정적 멤버 초기화 대신 정적 생성자를 사용해야 한다. 
```C#
public class MySingleton2
{
  private static readonly MySingleton2 theOneAndOnly;
  
  static MySingleton2()
  {
    theOneAndONly = new MySingleton2();
  }
  
  public static MySingleton2 TheOnly
  {
    get
    {
      return theOneAndOnly;
    }
  }
  
  private MySingleton2()
  {
  }
}
```

## Item 14: 초기화 코드가 중복되는 것을 최소화하라
생성자의 기본 매개변수 기능을 활용하면 생성자 내의 중복 코드를 줄일 수 있다.
```C#
public class MyClass
{
  // new() 제약 조건을 만족시키려면 이 생성자가 필요하다.
  public MyClass() :
    this(0, string.Empty)
  {
  }
  
  public MyClass(int initialCount = 0, string name = "")
  {
    this.coll = (initialCount > 0) ?
      new List<ImportantData>(initialCount) :
      new List<ImportantData>()
      
    this.name = name;
  }
}
```
어떠한 경우에도 제한 없이 이런 구조를 사용하려면 매개변수가 없는 생성자를 명시적으로 작성해야 한다. new() 제약 조건을 명시한 제네릭 클래스와 함께 사용해야 할 경우에는 반드시 매개변수가 없는 생성자를 구현해야 한다.

인스턴스가 생성되는 동안 모든 멤버 변수들을 가능한 한 한 번만 초기화하도록 해야 한다. 이를 위해 초기화를 가능한한 이른 시점에 수행하는 방식을 적용하는 것이 좋다.
인스턴스를 생성할 때 수행되는 과정
1. 정적 변수의 저장 공간을 0으로 초기화
2. 정적 변수에 대한 초기화 구문 수행
3. 베이스 클래스의 정적 생성자 수행
4. 정적 생성자 수행
5. 인스턴스 변수의 저장 공간을 0으로 초기화
6. 인스턴스 변수에 대한 초기화 구문 수행
7. 적절한 베이스 클래스의 인스턴스 생성자 수행
8. 인스턴스 생성자 수행

## Item 15: 불필요한 객체를 만들지 말라
1. 모든 참조 타입의 객체는 설사 지역변수라 하더라도 동적으로 메모리를 할당한다. 자주 호출되는 지역변수는 멤버변수로 변경하여 재사용하는 것이 좋다.
2. 자주 사용되는 참조 타입의 인스턴스를 정적 멤버변수로 선언하면 하나의 인스턴스를 영원히 재사용하게 된다. 다음의 예제에서는 지연 평가(lazy evaluation)를 통해 최초로 요청될 때 비로소 필요한 객체를 생성함으로써 객체 생성을 극도로 제한하고 있다.
```C#
public class Brushes
{
  private static Brush blackBrush;
  
  public static Brush Black
  {
    get
    {
      if (blackBrush == null)
        blackBrush = new SolidBrush(Color.Black)
      return blackBrush;
    }
  }
}
```
3. 다음의 코드에서 msg 변수의 문자열이 변경되는 것이 아니라 새로운 문자열을 가진 새로운 string 객체가 반복적으로 생성된다. 이 경우에는 문자열 보간 방법([Item4: string.Format()을 보간 문자열로 대체하라](chapter1.md#item4-interpolated-string)) 또는 StringBuilder를 사용하는 것이 좋은 방법이다.
```C#
string msg = "Hello, ";
msg += thisUser.Name;
msg += ". Today is ";
msg += System.DateTime.Now.ToString();
```

## Item 16: 생성자 내에서는 절대로 가상함수를 호출하지 마라

## <a name="item17-dispose-pattern">Item 17: 표준 Dispose 패턴을 구현하라

  
