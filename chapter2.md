# .NET 리소스 관리
## Item 11: .NET 리소스 관리에 대한 이해
- 가비지 수집기(garbage collector)
  - 관리되는 메모리(managed memory)를 관장하며 메모리 누수(memory leak), 댕글링 포인터(dangling pointer), 초기화되지 않은 포인터, 여타의 메모리 관리 문제를 개발자들이 직접 다루지 않도록 자동화 해준다.
  - 마크/콤팩트(mark/compact) 알고리즘은 여러 객체 사이의 연관 관계를 효율적으로 파악하여 더 이상 사용됮 않는 객체를 자동으로 제거한다. 객체가 스스로 자신의 참여 횟수를 관리하도록 하지 않고, 응용프로그램의 최상위객체로부터 개별 객체까지의 도달 가능 여부를 확인하도록 설계되어 있다.
  - 사용 중인 객체들을 한쪽으로 차곡차곡 옮겨서 조각난 가용 메모리를 단일의 큰 메모리 공간으로 만드는 콤팩트(compact) 작업을 수행한다.
- 비관리 리소스는 여전히 개발자가 관리해야 한다. finalizer와 IDisposable 두 가지 방법을 제공한다.
- finalizer는 리소스에 대한 해제 작업이 반드시 수행될 수 있도록 도와주는 방어적인 매커니즘이다. 하지만 finalizer를 가지고 있는 객체는 가비지로 간주된 이후에도 꽤 긴 시간동안 메모리를 점유하게 되는 단점이 있다.
- 가장 좋은 방법은 IDisposable 인터페이스와 표준 Dispose 패턴을 활용하는 것이다. [Item 17: 표준 Dispose 패턴을 구현하라](Item 17: 표준 Dispose 패턴을 구현하라)
- 
- (#Item-17:-표준-Dispose-패턴을-구현하라)


- asdfa
- asdf
- asd
- fa
- dfas
- df
- asdf
- asd
- fas
- df
- asdf
- asd
- fas
- dfa
- sdf
- asd
- fas
- df
- asd
- fasdf
- asd
- fas
- df
- asdf
- 
sf
as
df
asd
fsa
dfa
sdf
sad
fsa
df
sf
s
f
sadf
saf
s
dfs
af

# abc
## Item 17: 표준 Dispose 패턴을 구현하라
