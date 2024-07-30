# 추상 팩토리 패턴

## 사전 지식

### 1. 일급 함수 (first-class functions)

`일급 함수`란 함수 자체를 다른 객체와 동일하게 취급할 수 있는 기능을 의미한다. 

- 특징
1. 함수를 변수에 할당할 수 있다.

``` python
def greet(name):
    return f"Hello, {name}!"

greet_function = greet # greet라는 함수를 변수에 할당

print(greet_function("Alice")) # greet라는 함수에 매개변수를 전달하듯이
                               # greet 함수를 할당한 greet_function에 매개변수를 전달했음 
                               # "Hello, Alice!" 그래서 왼쪽과 같은 결과가 출력됨 

```

2. 함수를 다른 함수의 인자로 전달할 수 있다.

``` python
def call_function(func, name):
    return func(name) # 매개변수로 전달받은 함수의 return값을 call_function 함수의 return값으로 반환 

print(call_function(greet, "Bob"))  # 앞서 정의한 greet라는 함수를 매개변수로 전달함 
                                    # "Hello, Bob!"
```

3. 함수를 함수의 반환값으로 사용할 수 있다.

``` python
def get_greet_function():
    def greet(name):
        return f"Hello, {name}!"
    return greet # 함수 자체를 반환값으로 지정함 
 
greet_func = get_greet_function()
print(greet_func("Charlie"))  # "Hello, Charlie!"

```

4. 함수를 데이터 구조에 저장할 수 있다.

``` python
# greet라는 함수 자체를 리스트에 저장함
# str.upper, str.lower 함수 자체를 리스트에 저장함 
functions = [greet, str.upper, str.lower] 

for func in functions:
    print(func("Dave"))  # "Hello, Dave!", "DAVE", "dave"
```

이 기능을 이용해서 추상 팩토리 패턴의 복잡한 구조를 사용하지 않고도 유사한 기능을 구현할 수 있다. 
``` python
class Dog:
    def speak(self):
        return "Woof!"

class Cat:
    def speak(self):
        return "Meow!"

def animal_speaker(animal_class): 
    animal = animal_class() # 매개변수로 받은 클래스 자체를 변수에 저장한다. 
    return animal.speak() # 해당 클래스에 speak() 라는 메소드의 반환값을 animal_speaker 함수의 반환값으로 지정함

print(animal_speaker(Dog))  # "Woof!"
print(animal_speaker(Cat))  # "Meow!"

```

## 저자의 의견

```
추상 팩토리 패턴은 일급 함수나 클래스를 지원하는 python에는 적합하지 않다. 
Python에서는 클래스 또는 함수 자체를 전달할 수 있기 때문이다. 
```

## 서론 

Python 표준 라이브러리 `json 모듈`을 가지고 설명하겠다. 

ex. 
``` python
text = '{"total": 9.61, "items": ["Americano", "Omelet"]}'
```

- 기본 설정 : `json 모듈의 loads() 함수`가 unicode object를 생성한다.
  - Americano, Omelet : 각각에 대해서는 str 객체로 변환 
  - ["Americano", "Omelet"] : list 객체로 변환
  - 9.61 : float 객체로 변환
  - 최상위 json object : dict 객체로 변환 

- 원하는 상황
  - 9.61 : 이 값을 Decimal 객체로 변환하고 싶음

이렇게 json 모듈이 만들어낸 numeric 객체를 바꾸고 싶은 건 일반적인 문제의 특정 사례이다. 

일반적인 문제

  - 어떤 작업을 수행하는 동안, 루틴(예: 함수, 메서드)이 호출자를 대신해 여러 객체를 생성해야 할 때가 있다.
  - 그러나 루틴이 기본적으로 인스턴스화하는 클래스가 모든 경우를 다루지 못할 수 있다.
  - 따라서 기본 클래스를 하드코딩하여 맞춤 설정을 불가능하게 만들지 말고 `호출자`가 `인스턴스화할 클래스를 지정`할 수 있도록 해야 합니다.

cf) 루틴 : 함수, 메서드, 서브루틴 등에서 특정 작업을 수행하기 위해 작성된 코드 블록 

내가 정의한 코드 블록에서 특정 클래스의 인스턴스가 필요할 수 있다. 
하지만, 특정 클래스의 인스턴스를 만드는 코드를 함수에 정의하게 되면 앞서 언급한 `하드코딩`과 같이 일반화된 코드가 작성되지 않는다. 

그래서 이러한 클래스를 `호출자`가 설정하게 함으로써 루틴 자체의 일반적인 특성을 유지하도록 할 수 있다. 

## 1번째 접근 : Pythonic 접근

``` python
import json
from decimal import Decimal

def build_decimal(string):
    return Decimal(string)

# parse_float 매개변수에 내가 정의한 build_decimal 함수를 전달함 
print(json.loads(text, parse_float=build_decimal)) 

# 출력값
{'total': Decimal('9.61'), 'items': ['Americano', 'Omelet']}

##################
# Decimal 자체가 함수이기 때문에 아래와 같이 간략하게 작성할 수 있다. 
import json
from decimal import Decimal
print(json.loads(text, parse_float=Decimal))

# 출력값 
{'total': Decimal('9.61'), 'items': ['Americano', 'Omelet']}

```

이게 어떻게 가능한지는 json 모듈을 자세히 살펴보면 알 수 있다. 

1. `json.loads()` 함수는 `JSONDecoder 클래스`를 호출하는 wrapper 함수다.
2. 그래서 JSONDecoder 클래스를 좀 더 살펴보겠다.

- JSONDecoder 클래스의 초기화 메서드
``` python
class JSONDecoder:
    def __init__(self, *, parse_float=None, **kwargs):
        self.parse_float = parse_float or float # parse_float 인수가 제공되지 않으면
                                                # 기본적으로 float 타입을 사용
        # ...
```

이후에 JSONDecoder 클래스에서 숫자를 파싱할 때 `parse_float`를 호출해서 아래와 같이 사용한다. 
``` python
def some_parsing_method(self, json_number_string):
    result = self.parse_float(json_number_string) # 초기화한 parse_float에 숫자 문자열을 매개변수로 전달받음
    return result

즉,
- float가 전달되었다면? float(json_number_string)의 반환값을 반환
- Decimal이 전달되었다면? Decimal(json_number_string)의 반환값을 반환
```
