# Chapter 1: C# 언어 요소
## Item 1: 지역변수를 선언할 때는 var를 사용하는 것이 낫다
- 코드를 읽을 때 타입을 명시적으로 드러내야 하는 경우가 아니라면 var를 사용하는 것이 좋다.
- 내장 숫자 타입(int, float, double 등)을 선언할 때는 명시적으로 타입을 선언하는 편이 낫다.
- 타입을 명시적으로 지정하면 잘못된 동작을 미연에 방지할 수 있다.

아래의 코드는 심각한 성능 문제를 유발한다. LINQ 쿼리는 IQueryable<string> 타입을 반환하지만, IEnumerable<string>으로 선언했으므로 IQueryable<string>의 장점을 잃게 된다. (IQueryable<T>가 IEnumerable<T>를 상속)
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
  
## Item 4: string.Format()을 보간 문자열(string interpolation)으로 대체하라
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
  
### Item 6: nameof() 연산자를 적극 활용하라

  
  
