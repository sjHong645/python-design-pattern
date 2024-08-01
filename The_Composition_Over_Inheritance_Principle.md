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

