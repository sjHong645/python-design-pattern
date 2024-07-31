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

Factory method는 HTTP connection pool 클래스가 만들어내는 
