## 저자의 말 

다른 언어와 마찬가지로 Python에서도  
`"상속(Inheritance)보다 구성(Composition)"이라는 위대한 원칙`은 소프트웨어 아키텍트들이 객체 지향 프로그래밍(Object Orientation)에서 벗어나  
`객체 기반 프로그래밍(Object Based programming)의 더 단순한 방식들`을 `즐기도록 장려`한다. 

[Gang of Four Book](https://python-patterns.guide/gang-of-four/)에서 얘기한 2번째 원칙은 아래와 같다.  

```
클래스 상속(Inheritance)보다 객체 조합(composition)을 더 선호할 것
```

하나의 문제 상황을 예시로 들어서 이 원칙이 여러 개의 GoF 디자인 패턴을 통해 어떻게 구현되어 있는지 살펴보자.  
각각의 디자인 패턴들은 간단한 클래스를 조합하여 상속을 줄여 우아한 해결책이 된다. 

## 문제 상황 : 과다 하위 클래스 

디자인 전략으로써 상속의 가장 큰 단점은 클래스가 종종 여러 다른 디자인 축에 따라 특수화되어야 한다는 점이다.  
이로 인해 하위 클래스의 개수가 굉장히 많아진다. 

ex. Python의 logging 모듈 

해당 모듈은 `상속 보다 구성(composition)`을 따른 모듈이다. 

``` python
import sys
import syslog

# The initial class. 기본 클래스 
class Logger(object):
    def __init__(self, file):
        self.file = file

    def log(self, message):
        self.file.write(message + '\n')
        self.file.flush()

# 로그 메시지를 보내는 목적지에 따라 하위 클래스를 만들어냈다
class SocketLogger(Logger):
    def __init__(self, sock):
        self.sock = sock

    def log(self, message):
        self.sock.sendall((message + '\n').encode('ascii'))

class SyslogLogger(Logger):
    def __init__(self, priority):
        self.priority = priority

    def log(self, message):
        syslog.syslog(self.priority, message)
```

여기서 만약 로그 메시지가 필터링되도록 디자인된 로거 클래스를 만든다면? 역시나 새로운 하위 클래스를 생성해줘야 한다.

``` python
# New design direction: filtering messages.

class FilteredLogger(Logger):
    def __init__(self, pattern, file):
        self.pattern = pattern
        super().__init__(file)

    def log(self, message):
        if self.pattern in message:
            super().log(message)

# It works.

f = FilteredLogger('Error', sys.stdout)
f.log('Ignored: this is not important')
f.log('Error: but you want to see this')
```

여기에 더해, 로그의 내용을 파일이 아닌 소켓에 전송한다면... 역시나 또 다른 하위 클래스를 만들어야 한다.  
지금까지 만든 각 로거의 필터링된 기능을 추가하고 싶다면... 만들어놨던 클래스를 상속해서 또 하위 클래스를 추가해줘야 한다. 

이렇게 클래스의 개수가 계속해서 증가하게 된다. GoF에서 피해야 한다고 얘기한 바로 그 내용이다. 

문제점은 메시지를 필터링과 로깅 두 가지 역할을 맡은 클래스가 너무 복잡하다는 것이다.  
이는 `SRP(Single Responsibility Principle)` 위반이다. 

하지만 메시지 필터링과 메시지 출력을 두 가지 기능으로 나누어 서로 다른 클래스에 어떻게 분배할 수 있을까? 

## 1번째 해결책 : Adapter 패턴 

원형(original) 로거 클래스는 개선할 필요가 없다고 결정한다. 
왜냐하면, 메시지를 출력하는 메커니즘은 로거가 기대하는 파일 객체처럼 보이도록 감쌀 수 있기 때문이다. 

1. 그래서 `원형 Logger`를 유지하고
2. `FilteredLogger`도 유지하자
3. 여기서, 목적지에 따라서 로거 클래스를 만드는 대신에 각각의 목적지를 파일의 동작 방식에 맞도록 적응(adapt)시킨 후 적응시킨 어댑터를 출력 파일로써 Logger에 전달한다.

ex. 
``` python
import socket

# The initial class. 기본 클래스 
class Logger(object):
    def __init__(self, file):
        self.file = file

    # log 메서드에 file의 메소드 write, flush를 사용했다. 
    def log(self, message):
        self.file.write(message + '\n')
        self.file.flush()

# 다른 거 필요없이 write, flush 메소드만 구현해줬다.
# 물론, socket에 전송하기에 적합하게 구현했다.
class FileLikeSocket:
    def __init__(self, sock):
        self.sock = sock

    def write(self, message_and_newline):
        self.sock.sendall(message_and_newline.encode('ascii'))

    def flush(self):
        pass

# 마찬가지로 
# 다른 거 필요없이 write, flush 메소드만 구현해줬다.
# 물론, syslog에 저장하기에 적합하게 구현했다.
class FileLikeSyslog:
    def __init__(self, priority):
        self.priority = priority

    def write(self, message_and_newline):
        message = message_and_newline.rstrip('\n')
        syslog.syslog(self.priority, message)

    def flush(self):
        pass
```

Python은 `덕 타이핑`을 지원한다. 때문에 어댑터는 적절한 메서드만 전달해주면 된다.  
또한, 파일이 제공하는 모든 메소드를 다시 구현할 필요가 없다.  
내가 필요한 게 `꽥(quack)`이라면 `오리(duck)`가 `걷는지는 중요하지 않은 것`처럼 어댑터는 Logger 클래스가 사용하는 2개의 파일 메서드만 구현하면 된다. 

위 예시는 Logger 클래스가 log 명령어를 위해서 사용하는 `write`, `flush`만 다시 구현한 2개의 클래스를 볼 수 있었다. 

이제 과다 하위 클래스 문제를 해결할 수 있다.  
이제 추가적인 클래스 생성없이 Logger object 또는 adapter object를 자유롭게 섞어서 사용할 수 있다. 

``` python
sock1, sock2 = socket.socketpair()

fs = FileLikeSocket(sock1) # 파일은 파일인데 socket에 전송하기에 더 적합하게 구현함 
logger = FilteredLogger('Error', fs) # 어차피 write, flush가 구현되어 있으니 상관없음 (덕 타이핑) 
logger.log('Warning: message number one')
logger.log('Error: message number two')

print('The socket received: %r' % sock2.recv(512))
```

## 2번째 해결책 : Bridge 패턴 

브리지 패턴은 `호출자가 보는 추상 object`와 `안에 감싸져있는 구현 object`로 클래스의 행동을 나눈다.  












