## 저자의 의견 

Python은 프로그래밍 언어에서 사용할 수 있는 가장 기본적인 단계에서 반복자 패턴을 지원한다.  
즉, 파이썬의 문법에 이 패턴이 내장되어 있다. 

하지만, `객체 기반의 반복자 패턴` 뿐만 아니라 `자체적인 기존 iteration 프로토콜`을 지원하기 위해서  
실제 반복 객체 프로토콜들을 2개의 dunder 메서드로 위임했다.  

직접 이 메서드들을 호출하는 대신 내장함수를 통해 반복자 패턴을 수행한다.

## 개요 

반복자 패턴은 어떻게 데이터 구조가 순회되는지에 대한 디테일들을 `iterator 객체`로 이동시켰다.  
그래서 바깥에서는 데이터 구조의 내부 디자인을 모르지만 간단하게 각각의 값을 순서대로 반환한다. 

## for문을 이용한 반복 

Python의 for문은 반복자 패턴을 완전히 추상화했다.  

``` python
some_primes = [2, 3, 5]
for prime in some_primes:
    print(prime)

2
3
5
```

``` python
elements = [('H', 1.008), ('He', 4.003), ('Li', 6.94)]

# You’re not limited to a single name like “tup”...
for tup in elements:
    symbol, weight = tup
    print(symbol, weight)

# ...instead, unpack right inside the "for" statement
for symbol, weight in elements:
    print(symbol, weight)
```

## 패턴 : iterable과 iterator

`for 루프`에 대해서 좀 더 살펴보면서 관련된 디자인 패턴에 대해서 알아보자

전통적인 반복자 패턴은 3가지 object를 포함한다. 

1. container 객체
2. container의 내부 로직은 여러 개의 item 객체를 모아서 정리할 수 있도록 해준다.
3. 반복자 패턴의 가장 중요한 점  
  - 컨테이너가 item들을 지나가는 자기만의 메서드 호출을 만들지 말고
  - `일반적인 iterator 객체`를 통해서 `각각의 item에 순차 접근`해야 한다.
  - 이때 iterator 객체는 다른 종류의 container의 반복자와 마찬가지로 동일한 인터페이스를 구현해야 한다.

Python은 for 루프 뒤에서 직접 반복을 제어할 수 있도록 하는 2개의 내장함수를 제공한다.

- `iter()` : 매개변수로 `container 객체`를 받는다. 그 객체를 가지고 `새로운 iterator 객체`를 만들어서 반환한다.
  - 만약, 전달한 매개변수가 container 객체가 아니라면 `TypeError : object is not iterable 가 발생`한다.
- `next()` : 매개변수로 `iterator 객체`를 받는다. 이 메소드를 호출할 때 마다 `container에 있는 다음 item 객체`를 반환한다.
  - 만약, container에 더 이상 반환할 객체가 없다면 StopIteration 예외가 발생한다.
 
ex. 
``` python
some_primes = [2, 3, 5]
>>> it = iter(some_primes)
>>> it
<list_iterator object at 0x7f072ffdb518>
>>> print(next(it))
2
>>> print(next(it))
3
>>> print(next(it))
5
>>> print(next(it))
Traceback (most recent call last):
  ...
StopIteration
```

이 과정은 앞서 `for 반복문`을 통해서 실행된 과정과 비슷해보인다는 걸 확인할 수 있다. 

반복자 패턴을 공부하면서 드는 질문은 다음과 같다.  

- 왜 iterator는 container와 분리되어야 하는가?
- 왜 각각의 container는 다음에 실행해야 할 객체가 뭔지 알려주는 counter를 포함하지 않을까?
























