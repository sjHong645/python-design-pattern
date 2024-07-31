## 저자의 말 

Factory 메소드 패턴은 Python에 적합하지 않은 방식이다.  
이 패턴은 클래스와 함수가 매개변수로써 전달되지 못하거나 변수에 저장되지 않는 프로그래밍 언어를 위해 만들어졌기 때문이다. 

## 서론 

Python에서 내가 생성한 객체가 다른 객체를 생성하는 역할을 수행할 때가 있다. 

- 간단한 예시
``` python
class Car:
    def __init__(self, engine_type):
        self.engine = Engine(engine_type)

class Engine:
    def __init__(self, type):
        self.type = type

# Car 객체를 생성하면, 그 과정에서 Engine 객체도 생성됩니다.
my_car = Car('V8')
```

HTTP connection Pool도 관련된 예시 중 하나다.  
HTTP connection Pool은 때때로 오래된 connection이 기간이 만료되거나 종료될 때 새로운 connection을 만들어야 한다.  

여기서 문제가 발생한다. 

만약에 내 프로그램이 특정 회사 프록시 뒤에서 동작하고  
HTTP connection pool이 일반적으로 생성하는 소켓이 아닌 특별 구성된 네트워크 연결을 생성하고 사용해야 한다면? 

Factory method는 HTTP connection pool 클래스가 어떤 종류의 connection을 생성할지 선택할 수 있는 방법을 제공하는 하나의 패턴이다.

이 어색한 패턴이 어떻게 작동하는지 배우기 전에, 동일한 문제를 해결하는 Pythonic 디자인 패턴을 살펴보자. 

## 회피책 : DI(의존성 주입) 

여기서 한 번 더 생각해보자. 

내가 디자인하는 클래스가 정말로 다른 클래스를 생성할 필요가 있는가? 

만약 클래스가 필요로 하는 모든 객체를 미리 알고 있다면,  
`의존성 주입(Dependency Injection)`을 통해 직접 클래스에 객체를 제공하는 걸 고려해야한다.

ex. json 파서
``` python
# path를 요구하지 않고
# json 모듈의 load() 메소드를 사용해서 파일을 open 하도록 했다.

# 이처럼 Python은 파일 경로를 직접 제공하고 라이브러리가 파일을 직접 열도록 하는 대신에 
# 이미 열려있는(already-open) 파일 객체를 전달하는 방식을 사용할 수 있다. 

import json
with open('input_data.json') as f:
    data = json.load(f)
```

위와 같이 파일을 인스턴스화 하는 작업을 사용자에게 맡김으로써 아래와 같은 목적들을 달성할 수 있다. 

- Decoupling : 라이브러리는 `open() 메소드`가 전달받은 `모든 매개변수를 알 필요가 없`다. 또한 `모든 매개변수를 라이브러라 자체가 직접 전달받을 필요는 없`다. 만약 파일 객체가 나중에 더 많은 생성 매개변수를 갖게 되더라도, json.load()를 `변경할 필요가 없`다.
- Efficiency : 이미 다른 이유로 파일을 열어둔 상태라면, `열린 파일 객체를 그대로 이용`할 수 있다. 즉, 라이브러리가 파일을 다시 열 필요가 없다. 만약, 한 번만 읽을 수 있는 데이터 소스가 있다면 이는 굉장히 중요한 특징이다. 
- Flexibility : 원하는 `유사 파일(file-like) 객체를 전달할 수` 있다. 이는 표준 파일 객체의 서브클래스일 수 있고 file 처럼 동작하지만 표준 파일 객체의 서브클래스가 아닌 완전히 독립적인 객체일 수 있다. 

이처럼 간단한 경우에서, DI 패턴은 다른 객체를 어떻게 생성할 지에 대한 세부 사항으로 인해 JSON 파서와 같은 객체의 설계가 복잡해지는 것을 막아준다. 

따라서, 클래스가 항상 특정 객체를 필요로 하거나 사용자가 그 객체의 인스턴스화 매개변수를 어느 정도는 사용자 정의하고 싶어 한다면, DI를 고려해봐야 한다. 

## 대안 : class attribute factory 

생성돼야 하는 클래스는 `생성(creating) 작업을 수행할 클래스`의 `속성(attribute)`으로써 접근할 수 있어야 한다.  
이러한 패턴은 Python 표준 라이브러리의 옛날 모듈에서 확인할 수 있다. 

ex. `HTTPConnection`이 요청을 보냈을 때 해당 클래스는 response를 받고 분석하는 동작을 수행하도록 기대된다. 

하지만, 응답(response)을 분석하고 원하는 방식으로 출력하기 위해서는 어떤 클래스를 사용해야 할까?  
만약, 내가 서버에 요청을 보냈는데 그 서버가 특수 처리를 해줘야 하는 비표준적인 정보를 응답으로 전송한다면? 

`Attribute Factory Pattern`은 Python에서 `클래스`가 `일급 객체`라는 걸 이용한다.  
`생성 작업을 수행할 클래스`를 하나의 속성으로써 접근한다. 

``` python
class HTTPConnection:
    ...
    response_class = HTTPResponse # class의 속성 변수 response_class에
                                  # HTTPResponse라는 클래스를 전달했다. 
    ...
    def getresponse(self):
        ...
        # HTTPResponse 클래스 초기화에 필요한 매개변수를 전달함으로써 인스턴스를 생성하여
        # response라는 변수에 할당 
        response = self.response_class(self.sock, method=self._method)
        ...
```

이렇게 하면 `HTTPConnection`을 사용하는 코드가 응답을 생성할 때 어떤 일이 발생하는지 완전히 제어할 수 있다.  
추가적으로 필요한 게 있다면 서브클래스를 생성해서 사용한다. 

``` python
class SpecialHTTPConnection(HTTPConnection):
    # 부모 클래스인 HTTPConnection는 원래 response_class 변수에 HTTPResponse를 할당했다.

    # 그렇게 하고 싶지 않다면 서브 클래스를 만들어서
    # SpecialHTTPResponse를 할당하도록 설정해주면 된다. 
    response_class = SpecialHTTPResponse

```

이렇게 서브 클래스를 만들어서 이용하는 방식은 Python 언어의 흐름에 역행하지만, 잘 동작하기는 한다. 

이 부분에서 알아봐야 할 더 많은 유연성이 있지만 먼저 class attribute factory에 대한 더 현대적인 대안을 살펴보도록 하자. 

## 대안 : Instance Attribute Factory 
```
왜 클래스의 행동을 커스터마이즈 하기 위해서 객체를 서브클래스로 만들어야 하는 걸까? 
```

이 질문은 굉장히 중요하다. 

실제로 프로그래밍 역사상 `객체 지향이라는 교리를 신봉하는 일부 사람들`은 이렇게 말해왔다.  
`Submit이라는 버튼이 필요하다`면, 단순히 label="Submit" 같은 `매개변수로 버튼을 인스턴스화하는 것이 아니`라,  
버튼을 `서브클래싱하고 상위 클래스에 있던 label() 메서드를 오버라이드`하여 `새로운 값을 반환`하도록 해야 한다고 주장한 적이 있습니다.

다행히도, 서브클래싱을 이용한 커스터마이징의 대안이 Python 커뮤니티에서 널리 퍼져있었다.  
이를 `Instance Attribute Factory`라고 부르겠다. 이를 설명하기에 좋은 모듈이 json 모듈이다. 

### 예시. json 모듈

json 모듈은 입력값을 숫자로 받을 때마다 그 `숫자`를 `출력할 수 있는 Python 객체`로 인스턴스화해야 한다.  
하지만, 숫자 클래스를 어떤 클래스로 인스턴스화 해야 할까? 

소수 부분(1.33에서 33)이 0인 integer? float? Decimal? 

json 모듈은 이 문제를 유려하게 해결해냈다. 

``` python
class JSONDecoder(object):
    ...
    
    def __init__(self, ... parse_float=None, ...):
        ...
        # 숫자 값을 입력값으로 받을 때마다 
        # json 모듈은 self.parse_float() 함수를 호출한다. 
        self.parse_float = parse_float or float
```

이렇게 정의된 Python 코드는 모든 상황에서 유용하게 사용될 수 있다.  
개발자가 숫자 타입을 별도로 정의하고 싶지 않다면, 각각의 숫자 값은 매우 빠른 속도로 float 타입으로 해석된다.  
개발자가 숫자 타입을 별도로 정의하고 싶다면, 개발자가 원하는 callable을 전달하면 된다. 

더 예술인 건 이 작업을 단 하나의 클래스도 추가하지 않고 수행할 수 있다는 거다. 

개발자에게 자기가 원하는 방식으로 동작하는 새로운 클래스를 생성하도록 강요하는 대신에  
각각의 JSONDecoder 인스턴스를 직접 설정할 수 있다. 
``` python
from decimal import Decimal
from json import JSONDecoder

my_decoder = JSONDecoder(parse_float=Decimal)
```

매개변수를 통해 객체를 커스터마이징하는 것의 장점은 명확성과 간결함 뿐만 아니라  
Python에서 매개변수가 쉽게 조합될 수 있다는 것이다.  
Python에서는 여러 매개변수를 결합해야 할 때, 비어있는 딕셔너리에 update() 메서드를 사용해서 이를 쉽게 관리할 수 있다. 

## Instance attribute가 class attribute를 오버라이드할 수 있다. 

앞서 2가지 디자인 패턴을 비교했다. `class attribute factory`, `instance attribute factory` 
두 패턴 모두 객체를 생성할 때 `self`를 사용해서 특정 속성에 접근한다. 
여기서 `클래스 속성`과 `인스턴스 속성`이 `동일한 이름`을 가진다면, 인스턴스 속성을 우선시한다. 

ex. 
``` python
conn = HTTPConnection() # HTTPConnection 클래스의 인스턴스를 conn 이라는 변수에 할당 
conn.response_class = SpecialHTTPResponse # 원래는 conn에 저장된 인스턴스의 response_class는 클래스 변수로 선언되었다.
                                          # 하지만, 내가 원하는 인스턴스가 있으면 좋겠다고 생각해서
                                          # conn.response_class 변수에 SpecialHTTPResponse를 할당했다. 
```

대원칙은 읽기 쉽고 유지보수하기 쉬운 패턴을 선택해야 한다. 

- 객체 생성 방식을 개발자가 자주 customizing 한다면 => 




