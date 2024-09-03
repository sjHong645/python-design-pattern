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












