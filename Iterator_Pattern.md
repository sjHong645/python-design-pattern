## 사전지식 : Iterable vs Iterator 

- Iterable : container에 있는 항목(item)들을 하나씩 차례로 반환할 수 있는 object. 즉, 반복이 가능한 객체를 의미한다.   
  ex) list, str, tuple
``` python
my_list = [1, 2, 3]  # 리스트는 iterable입니다.

# 리스트에 대해 for 루프를 사용할 수 있다
for item in my_list:
    print(item)
```
- Iterator : iterable의 요소(item)들을 하나씩 반환하는 object. 더 이상 호출할 수 있는 데이터가 없다면 StopIteration 예외가 발생한다 

``` python
my_list = [1, 2, 3]  # 리스트는 iterable
iterator = iter(my_list)  # iter()를 사용해 이터레이터를 생성

print(next(iterator))  # 1
print(next(iterator))  # 2
print(next(iterator))  # 3
# 다음 next 호출은 StopIteration 예외를 발생
```

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

반복자 패턴을 공부하면서 다음과 같은 질문을 던져볼 수 있다. 
- 왜 iterator는 container와 분리되어야 하는가?
- 왜 각각의 container는 다음에 실행해야 할 객체가 뭔지 알려주는 counter를 포함하지 않을까?

`하나의 counter`는 한 번에 하나의 for문이 컨테이너에서 동작할 때 효과가 있다.  
하지만, 똑같은 container에서 한 번에 여러 개의 for문을 동작시켜야 하는 상황이 많이 발생한다. 

ex. 
``` python
sides = ['heads', 'tails']
for coin1 in sides:
    for coin2 in sides:
        print(coin1, coin2)

heads heads
heads tails
tails heads
tails tails
```

만약에 `sides`가 오직 하나의 counter만 갖고 있었다면,  
`내부 loop`에서 모든 item을 반복했기 때문에 `외부 loop`에서는 더 이상 반복할 item이 없다. 

대신에 `iter()` 메소드를 호출할 때마다 각각의 반복문이 주어진다. 

``` python
>>> it1 = iter(sides)
>>> it2 = iter(sides)
>>> it1
<list_iterator object at 0x7fa8b8b292b0>
>>> it2
<list_iterator object at 0x7fa8b8b29400>

>>> it1 is it2 # id값이 서로 다르다
False
```

이처럼 여러가지 상황에서 동시에 여러 개의 for문을 사용해야 하는 상황이 많이 있다.  
그래서 각각의 for문은 서로가 방해되지 않기 위해서 자기만의 iterator가 필요하다. 

## 변형 : 자기 자신이 iterator인 객체

ex. Python 파일 객체 

for문을 통해 반복 작업을 진행할 때 해당 파일의 line들을 yield한다. 

하지만, 다른 자료구조와 달리 새로운 for문을 이용하여 순회할 때 맨 첫줄에서 반복문을 다시 시작하지 않는다.   
파일에서 반복문이 마지막으로 실행되었던 위치를 기억해서 그 위치부터 line들을 yield한다. 

ex. 파일 포맷 
``` 
From jdoe@machine.example Fri Nov 21 09:55:06 1997
From: John Doe <jdoe@machine.example>
To: Mary Smith <mary@example.net>
Subject: Saying Hello
Date: Fri, 21 Nov 1997 09:55:06 -0600

This is a message just to say hello.
So, "Hello".
```
``` python
def parse_email(f):

    # 파일의 첫 번째 줄을 envelope에 저장
    for line in f:
        envelope = line
        break

    # 파일의 두 번째 줄부터 반복문 시작
    # `\n'(줄 바꿈) 나올 때까지 각 line에 대해 원하는 작업 수행
    headers = {}
    for line in f:
        if line == '\n':
            break
        name, value = line.split(':', 1)
        headers[name.strip()] = value.lstrip().rstrip('\n')

    # 2번째 반복문이 끝난 부분부터 반복문 시작
    # 'From'이 문장의 맨 앞 글자이면 멈춘다 
    body = []
    for line in f:
        if line.startswith('From'):
            break
        body.append(line)
    return envelope, headers, body
```

for문을 가지고 좀 더 얘기하기 전에 `iter()` 메서드를 호출해보자.

`iter()` 메서드가 반드시 iterator를 반환해야 한다는 규칙에는 iterator가 iterable 객체가 되지 않아야 한다고 얘기한 적이 없다. 파일 객체는 이 점을 이용했다. 

파일 객체 자체가 iterator 역할을 해서 별도의 iterator 객체 생성 없이 파일의 다음 줄을 반환할 수 있다.  
즉, 파일 객체에 있는 각각의 line의 모음을 제공해서 여러 사용자 중에서 `next()`를 호출한 사용자에게 다음 줄을 전달한다. 
``` python
>>> f = open('email.txt')
>>> f
<_io.TextIOWrapper name='email.txt' mode='r' encoding='UTF-8'>
>>> it1 = iter(f)
>>> it1
<_io.TextIOWrapper name='email.txt' mode='r' encoding='UTF-8'>
>>> it2 = iter(f)
>>> it2
<_io.TextIOWrapper name='email.txt' mode='r' encoding='UTF-8'>
>>> it1 is it2 is f
True
```

여기서 파일이 아닌 다른 종류의 객체에서 별도의 단계로 반복(iterate)하는 기능을 원하면 어떻게 해야 할까? 

ex. `parse_email()` 메서드에 단순히 줄(line)들의 리스트를 제공한다면? 

해당 메서드에서 정의한 `2,3번째 for문`에서 매개변수로 전달받은 list의 1번째 반복문의 마지막 부분에서 시작하는게 아니라 `해당 리스트의 맨 처음`부터 `반복문을 시작`하기 때문에 의도한대로 메서드가 실행되지 않는다. 

``` python
# email.txt 파일을 통해 모든 줄을 갖고 있는 리스트를 lines 변수에 할당함 
with open('email.txt') as f:
    lines = list(f)

# 방금 정의한 리스트를 매개변수로 전달함 
envelope, headers, body = parse_email(lines)
print(headers['To'])
print(len(body), 'lines')

>> 출력값 
Mary Smith <mary@example.net>
0 lines
```

Python list와 같은 표준 컨테이너에 어떻게 점진적인 반복을 지원할 수 있을까? (마치 파일 객체처럼)

기존 반복자 패턴에 대한 또 다른 확장을 사용하면 된다.  
Python에서는 iterator가 `iter() 함수의 매개변수`로 전달되면 자기 자신을 반환한다. 

즉, 반복자 자체가 for문에 전달될 수 있으니까 
내가 `next() 함수`또는 `for문을 이용한 반복`을 이용해서 원하는대로 컨테이너 안에 있는 item들에 접근할 수 있다. 

``` python
some_primes = [2, 3, 5]

it = iter(some_primes) # Python의 표준 컨테이너인 list를 iter 메서드의 매개변수에 전달
                       # 그 자체로 iterator가 되었다.
print(next(it))

for prime in it:
    print(prime)
    break

print(next(it))

2 # 먼저 나온 print(next(it))의 출력문 
3 # for문 안에 있는 출력문
5 # 나중에 나온 print(next(it))의 출력문 
```

```
some_primes = [2, 3, 5]
it = iter(some_primes)

>>> it 
<list_iterator object at 0x7f4fdce6c5f8> # 앞서 iter()를 통해 획득한 iterator 
>>> iter(it)
<list_iterator object at 0x7f4fdce6c5f8> # iter()에 iterator를 전달하면 자기 자신을 반환한다. 
>>> iter(it) is it
True
```

그러면, list를 가지고도 `parse_email()` 메서드를 의도한대로 사용할 수 있다. 

``` python
with open('email.txt') as f:
    lines = list(f)

it = iter(lines) # list 객체를 iter() 메서드에 전달해서 반복자를 얻어냈다

envelope, headers, body = parse_email(it)
print(headers['To'])
print(len(body), 'lines')

Mary Smith <mary@example.net>
2 lines
```

## Iterable과 Iterator 구현하기 

어떻게 클래스가 반복자 패턴을 구현하고 Python 기본 반복 메커니즘인 `for`, `iter()`, `next()`에 연결할 수 있을까?

- container는 반복자 객체를 반환하는 `__iter__()` 메서드를 제공해야 한다. 이 메서드를 구현함으로써 container는 `iterable`이 된다.
- 각각의 반복자(iterator)는 `__next__()` 메서드를 제공해야 한다. `__next()__`는 호출될 때마다 container의 다음 항목을 반환한다. 더 이상 반환할 항목이 없다면 `StopIterator 에러`를 발생시킨다.
- 앞서 다뤘던 특수한 경우 : `for문`에 기존의 container를 전달하지 않고 `iterator를 전달`하는 경우
  - 이런 상황을 위해 각각의 반복자는 자기 자신을 간단하게 반환하기 위해 `__iter__()` 메서드를 제공해야 한다. 

위와 같은 요구사항들이 어떻게 함께 동작하는지 간단한 반복자(iterator)를 구현해보면서 알아보자. 

`__next__()` 메서드가 반환하는 항목들이 컨테이너 내부에 저장되어야 할 필요없다.  
또한 `__next()__`가 호출될 때까지 해당 항목들이 존재할 필요없다. 

즉, 컨테이너에 별도의 저장소를 구현하지 않아도 간단하게 iterator 패턴을 만들어낼 수 있다. 

``` python
class OddNumbers(object):
    "An iterable object."

    def __init__(self, maximum):
        self.maximum = maximum

    def __iter__(self):
        return OddIterator(self)

class OddIterator(object):
    "An iterator."

    def __init__(self, container):
        self.container = container
        self.n = -1

    def __next__(self):
        self.n += 2
        if self.n > self.container.maximum:
            raise StopIteration
        return self.n

    def __iter__(self):
        return self
```

위와 같이 3개의 간단한 메서드(1개는 container 객체, 2개는 iterator 객체)를 통해 `OddNumbers 컨테이너`는 어엿한 Iterable 객체가 되었다. 당연히 for문에서 잘 동작한다. 

``` python
numbers = OddNumbers(7) 

for n in numbers: # numbers 객체 반복 시작
                  # 해당 객체의 __iter__() 메서드를 호출했다. 여기서 iterator 객체를 반환받았다
                  # 그 iterator의 __next()__ 메서드를 호출해서 동작을 실행한다. 
    print(n)

1
3
5
7
```

내장 함수 `iter()`, `next()`를 사용해도 동작한다. 

``` python
it = iter(OddNumbers(5)) # OddNumbers(5) 객체를 만들었다.
                         # 그 객체를 iter()에 전달해서 __iter()__를 호출했다.
                         # 즉, OddNumbers(5).__iter()__의 반환값이 it에 저장되어 있다
                         
print(next(it)) # next() 메서드에 it를 전달함으로써 it.__next()__를 호출한다 
print(next(it))

1
3
```

## Python의 추가 간접 계층(extra level of indirection) 

왜 Python은 `iter()`, `next()` 메서드를 갖고 있을까? 

일반적인 반복(iteration) 패턴은 대부분의 프로그래밍 언어에서 내장함수 없이 동작한다.  
대신에, 이 패턴은 `container`가 `하나의 메서드를 구현`하고 `iterable 객체`가 `다른 메서드를 구현`하도록 함으로써 객체 동작의 직접 관여한다. 

그렇다면, 왜 Python은 메서드의 이름을 직접 지정하고 `obj.method()` 표기법을 통해 직접 호출하도록 하지 않았을까? 

그 이유는 Python이 기존의 반복(iteration) 메커니즘을 지원해야 했기 때문이다.

원래 Python의 for문은 오직 정수 인덱스를 가진 container만 지원했다. 리스트, 튜플 같은 거.  
즉, for문이 아래와 같이 동작한 거다 
```
>>> some_primes[0]
2
>>> some_primes[1]
3
>>> some_primes[2]
5
>>> some_primes[3]
Traceback (most recent call last):
  ...
IndexError: list index out of range
```

[PEP-234](https://peps.python.org/pep-0234/)를 기반으로 Python2.2에 반복자 패턴이 합쳐지면서 문제가 생겼다. 

만약, for문이 container의 `__iter__()`를 호출하면서 시작한다면 이전까지 만들어왔던 container에 무슨일이 벌어질까? 

다행히도 컴퓨터 과학의 모든 문제는 다른 간접 계층(another level of indirection)을 통해 해결할 수 있다. 

Python은 사용자와 언어가 기존 방식의 반복자 패턴과 객체 기반의 반복자 패턴을 둘 다 지원해야 하는 불행한 현실 사이에 한 쌍의 내부함수를 끼워넣기로 했다. 

PEP는 말했다.   

- for문은 이제 반복자 패턴을 우선으로 실행한다. 만약 container가 `__iter__()`를 제공하면 for문은 `__iter__()`가 반환하는 반복자(iterator)를 사용한다.
- 옛날 container를 지원하기 위해 `for문`은 `__getitem__()` 메서드를 찾아보고 만약 그 메서드가 존재하면 정수 0, 1, 2, ...를 메서드에 전달해서 각각의 항목(item)을 받아낸다. 항목을 받다가 IndexError가 발생하면 중단한다.
- Python 프로그래머들에게 이러한 메커니즘을 드러내기 위해서 `iter()` 메서드가 도입되었다. `iter()`메서드는 프로그래머가 `iterable 객체로 부터 iterator를 생성`하는 기본 동작을 수행할 수 있도록 했다.
- `next()`메서드가 나중에 추가되면서 반복자 패턴의 모든 부분이 내장함수에 의해 다뤄질 수 있게 되었다.

이 과정에서 `iter()` 메서드에 편리한 기능을 하나 추가했다. 하나 대신 2개의 인자를 전달하는 방식이다.  
이 방식은 [표준 Python library 문서](https://docs.python.org/3/library/functions.html#iter)에 나와있으니 관심있으면 읽어보자




















