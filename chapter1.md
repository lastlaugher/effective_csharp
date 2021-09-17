# Effective C# - Chapter 1: C# 언어 요소
## Item 1: 지역변수를 선언할 때는 var를 사용하는 것이 낫다
- 코드를 읽을 때 타입을 명시적으로 드러내야 하는 경우가 아니라면 var를 사용하는 것이 좋다.
- 내장 숫자 타입(int, float, double 등)을 선언할 때는 명시적으로 타입을 선언하는 편이 낫다.
- 타입을 명시적으로 지정하면 잘못된 동작을 미연에 방지할 수 있다.

아래의 코드는 심각한 성능 문제를 유발한다. LINQ 쿼리는 IQueryable<string> 타입을 반환하지만, IEnumerable<string>으로 선언했으므로 IQueryable\<string\>의 장점을 잃게 된다. (IQueryable\<T\>가 IEnumerable\<T\>를 상속)
```c#
IEnumerable<string> q =
  from c in db.Customers
  select c.ContactName;

var q2 = q.Where(s => s.StartWith(start));
```  
   아래는 IEnumerable<string>가 var로 변경된 코드이다. q는 IQueryable<string> 타입으로 추론이 되고, 컴파일러에 의해 첫번째 LINQ 쿼리에 where절을 추가된 최적화가 수행된다.
```c#
var q =
  from c in db.Customers
  select c.ContactName;

var q2 = q.Where(s => s.StartWith(start));
```

## Item 2: const 보다는 readonly가 좋다
- const
  - 컴파일 타임 상수
  - 컴파일타임에 변수가 값으로 대체된다.
  - 내장된 숫자형, enum, 문자열, null에 대해서만 사용가능 하다.
- readonly
  - 런타임 상수
  - 런타임에 값이 평가된다.
  - 모든 타입에 사용가능하다.
- const로 정의된 변수의 값이 수정되었을 때, 응용프로그램 전체가 리빌드 되지 않고 변경된 어셈블리만 배포되었을 때, 다른 어셈블리에는 변경 내용이 반영되지 않는다. 컴파일 시점에 값이 결정되므로, 이전의 값을 계속 사용하게 된다.
- const를 사용했을 때, readonly보다 빠르다는 장점이 있긴 하지만, 성능 개선 효과가 크지 않고 위와같은 유연성을 해치는 단점이 있다.
- C++에서는 const가 런타임 상수, constexpr이 컴파일타임 상수이므로 주의한다.
  
## Item 3: 캐스트보다는 is, as가 좋다
- 형변환 수행 방법 두가지
  - as 연산자
    - 더 방어적인 코드를 작성하기 위해 우선 is 연산자로 형변환 가능한지 확인 후에 실제 형변환 할 수도 있음
  - 캐스트 연산자
- as 연산자는 사용자 정의 형변환이 지원되지 않으므로, 런타임에 객체의 타입이 변환하려는 타입과 정확히 일치할 경우에만 형변환이 가능하다. (더 안전하고 효율적임)
- 캐스트 연산자는 InvalidCastException을 발생시키므로 try/catch 문이 필요하다. (as 연산자가 더 깔끔함)
  
## <a name="item4-interpolated-string">Item 4: string.Format()을 보간 문자열(string interpolation)으로 대체하라
- 기존에 많이 사용되던 string.Format()은 포맷 문자열과 인자 리스트를 분리해서 전달하는 구조이기 때문에 실수하기가 쉬운 구조이다.
   ```C#
   Console.WriteLine("The value of pi is {0}", Math.PI);
   ```
  
- 보간 문자열을 사용하면 코드의 가독성이 상당히 늘어난다.
   ```C#
   Console.WriteLine($"The value of pi is {Math.PI}");
   ```

## Item 5: 문화권별로 다른 문자열을 생성하려면 FormattableString을 사용하라
앞서 설명한 문자열 보간기능을 사용한 코드이다.
```C#
string first = $"It's the {DateTime.Now.Day} of the {DateTime.Now.Month} month";
```
FormattableString로 선언을 하면, 지정된 문화권을 고려한 문자열을 생성할 수 있다.
```C#
FormattableString second = $"It's the {DateTime.Now.Day} of the {DateTime.Now.Month} month";
```

아래는 독일어/독일 문화권을 지정하여 최종의 문자열로 변환하는 코드이다.
```C#
public static string ToGerman(FormattableString src)
{
    return string.Format(
        CultureInfo.CreateSpecificCulture("de-De"),
        src.Format,
        src.GetArgument(0));
}
```
  
## Item 6: nameof() 연산자를 적극 활용하라
nameof() 연산자는 심볼 그 자체를 해당 심볼을 포함하는 문자열로 대체해준다. 아래는 가장 일반적인 활용 예시이다.
```C#
public string Name
{
  get
  {
    return name;
  }
  set
  {
    if (value != name)
    {
      name = value;
      PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(Name));
    }
  }
}   
```
```C#
public static void ExceptionMessage(object thisCantBeNull)
{
    if (thisCantBeNull == null)
        throw new ArgumentNullException(nameof(thisCantBeNull),
                "We told you this can't be null");
}
```
  
하드코딩 대비 nameof() 연산자의 장점
- 이름 바꾸기 작업등의 리팩토링 작업을 수행할 때 실수를 줄일 수 있다.
- 인자의 이름을 전달해야하는 매개 변수에 정확히 그 값을 전달할 수 있다.
  
## Item 7: 델리게이트(deligate)를 이용하여 callback을 표현하라
C#에서 callback 함수는 델리게이트를 이용하여 표현된다. (C++의 function object와 비슷)

미리 정의 되어 있는 델리게이트
- Predicate<>: 조건을 검사하여 bool을 리턴
- Func<>: 여러개의 매겨변수를 받아 단일 결과를 리턴, Func<T, bool>과 Predicate<T>는 동일함
- Action<>: 여러개의 매개변수를 받아 void를 리턴

LINQ에서 델리게이트를 인자로 받아서 콜백을 수행한다. 
```C#
List<int> numbers = Enumerable.Range(1, 200).ToList();

var oddNumbers = numbers.Find(n => n % 2 == 1);
var test = numbers.TrueForAll(n => n < 50);
numbers.RemoveAll(n => n % 2 == 0);
numbers.ForEach(item => Console.WriteLine(item));
```
  
델리게이트는 기본적으로 멀티캐스트가 가능하다. 멀티캐스트 델리게이트인 경우에 반환값은 멀티케이스 체인에서 마지막으로 호출된 함수의 반환값이 되며 다른 반환값은 모두 무시된다.
```C#
public void LengthyOperation(Func<bool> pred)
{
  ...
}
  
Func<bool> cp = () => CheckWithUser();
cp += () => CheckWithSystem();
c.LengthyOperation(cp);
```  
  
## Item 8: 이벤트 호출 시에는 null 조건 연산자를 사용하라
델리게이트로 선언된 이벤트에 결합된 이벤트가 없으면 NullReferenceExpception 이 발생하게 된다. 다음과 같이 null 체크를 하면 대부분 잘 동작하지만 if 문과 실행문 사이에 다른 스레드에서 이벤트 핸들러 등록을 취소하면 여전히 문제가 발생한다.
```C#
public void RaiseUpdates()
{
  counter++;
  if (Updated != null)
    Updated(this, counter);
}
```
  
이런 경우에는 null 조건 연산자(null conditional operator)를 사용하면 간단하게 해결이 가능하다. ? 연산자 뒤에는 ()를 붙여 호출할 수 없으므로 Invoke 메서드를 사용해야 한다. C# 컴파일러는 모든 델리게이트와 이벤트에 대하여 Invoke() 메서드를 생성해준다.
```C#
public void RaiseUpdates()
{
  counter++;
  Updated?.Invoke(this, counter);
}
```

## Item 9: 박싱과 언박싱을 최소화화라
- 박싱
  - 값 타입의 객체를 임의의 참조 타입 내부에 포함시키는 방법이다. 
  - 참조 타입이 필요한 경우에도 값 타입을 쓸 수 있다.
  - 새롭게 생성된 참조 타입의 객체는 힙에 생성되며, 값 타입의 복사본이 새롭게 생성된 객체 내부에 저장된다.
  - ![](https://docs.microsoft.com/ko-kr/dotnet/csharp/programming-guide/types/media/boxing-and-unboxing/boxing-operation-i-o-variables.gif)

- 언박싱
  - 박싱되어 있는 참조 타입의 객체로 부터 값 타입 객체의 복사본을 가져오는 방법이다.
  - ![](https://docs.microsoft.com/ko-kr/dotnet/csharp/programming-guide/types/media/boxing-and-unboxing/unboxing-conversion-operation.gif)
  
- 박싱과 언박싱은 암시적으로 이뤄지며 객체에 대한 복사본을 생성하곤 하는데, 이로 인해 버그가 발생할 수도 있고 성능을 느리게 만든다.
- 가능한한 박싱과 언박싱이 일어나지 않도록 해야 한다.
  
## Item 10: 베이스 클래스가 업그레이드된 경우에만 new 한정자를 사용하라
BaseWidget이라는 클래스를 상속하여 다음과 같이 MyWidget을 정의했다.
```C#
public class MyWidget : BaseWidget
{
  public void NormalizeValues()
  {
  }
}
```
BaseWidget이 업데이트되었는데 공교롭게도 NormalizedValues()가 추가되는 바람에 빌드가 실패하게 되는 경우를 생각해보자.
```C#
public class MyWidget
{
  public void NormalizeValues()
  {
  }
}
```
이런 경우에 MyWidget의 NormalizedValues의 메서드 이름을 변경하는 대신 new 한정자를 사용하는 방법이 있다.
```C#
public class MyWidget : BaseWidget
{
  public new void NormalizeValues()
  {
  }
}
```
new 한정자를 남발하면 메서드를 호출할 때 모호한 상황이 발생할 수 있으므로, 위의 예처럼 베이스 클레스가 업그레이드되어 이름이 충돌하는 특별한 경우가 아니면 사용해서는 안된다.
  


