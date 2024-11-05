# Spring Framework의 핵심 개념과 디자인 패턴

## 1. 비즈니스 로직 분리와 클래스 의존성

### 1.1 클래스 분리의 필요성
- **높은 응집도(High Cohesion)**: 하나의 클래스나 모듈이 하나의 기능 또는 목적을 위해 긴밀하게 연관된 기능들로 구성
- **낮은 결합도(Low Coupling)**: 클래스 간의 상호 의존성을 최소화
- **유지보수성 향상**: 코드 변경이 용이하고 버그 수정이 간편해짐
- **재사용성 증가**: 독립적인 모듈로 분리되어 다른 프로젝트에서도 활용 가능

# 
비즈니스 로직을 분리하기 위한 방법으로 클래스의 분리를 생각해볼 수 있다. 이는 리팩토링이라고도 부를 수 있는데 이 경우, 비즈니스 별 응집도가 높하지고 결합도가 낮아지는 효과를 볼 수 있다.

Spring 프레임워크는 기본적으로 계층 구조로 구성되어 있는데, Spring과 다른 프레임워크들이 준수하는 중요한 규칙이 있다. 클래스는 다른 클래스에 의존 관계를 형성하면 안된다는 것인데, 예를 들어 A라는 클래스에서 B라는 클래스를 사용하고 있을 경우 B 클래스의 변경이 이루어진다면 이를 사용하는 A 클래스 역시 영향을 받는다는 내용이다. 그렇다면 이러한 문제점을 어떻게 해결할 수 있을까?

---

### 1.2 클래스 의존성 문제
Spring과 다른 프레임워크들이 준수하는 중요한 규칙 중 하나는 클래스 간의 직접적인 의존 관계를 피하는 것입니다.

**문제점 예시:**
```java
public class ClassA {
    private ClassB classB = new ClassB(); // 직접적인 의존성
    
    public void someMethod() {
        classB.doSomething(); // ClassB가 변경되면 ClassA도 영향을 받음
    }
}
```

이러한 직접적인 의존성은 다음과 같은 문제를 야기합니다:
- ClassB의 변경이 ClassA에 직접적인 영향을 미침
- 유닛 테스트의 어려움
- 코드 재사용성 저하
- 유지보수의 어려움

## 2. 해결 방안

### 디자인 패턴: 전략 패턴을 활용한 방법

---

Java에서는 인터페이스라는 개념이 존재한다. 인터페이스를 정의할 때 이를 구현하는 구현체에서 재정의해 사용하는 방법을 생각해볼 수 있다. 이 때 상속 관계에 이용되는 메서드를 템플릿 메서드라고 하는데, 이렇게 된다면 구현체의 변경이 일어나더라도 A라는 클래스는 인터페이스를 참조하고 있어 결합도가 낮아지는 효과를 볼 수 있다.

그러나 아직까지도 문제점이 존재하는데, 인터페이스 타입의 구현체 클래스를 new 연산자를 통해 객체를 생성하려면 인터페이스를 이용하는 파일에서 관계 설정과 관련한 의존성이 발생한다는 점이다. 이 경우 팩토리 패턴을 이용해 문제를 해결해볼 수 있다. 

우리가 필요로 하는 인터페이스의 구현체를 팩토리 클래스 안에 정의해두어 애플리케이션의 실행 흐름에서는 팩토리 클래스에서 객체를 가져올 수 있도록 하는 것이다. 다시 표현하자면 A클래스를 이용해 애플리케이션 로직을 실행하는 별도의 흐름이 존재할 때, A클래스가 B의 구현체를 알고 있어야 하는 제약사항을 없앨 수 있다는 장점이 존재한다.

기존의 객체지향 언어에서 클래스가 다른 클래스를 의존하고 있을 때 상세 내용을 알고 있어야 하지만, 런타임 시점까지 객체의 생성을 미룬다는 점에서 의존관계역전(IoC)라고도 한다.

이러한 IoC를 구현하기 위한 방법으로는 여러 가지 방법이 존재하지만, Spring에서는 DI를 주로 이용하고 있다. 아래를 살펴보자.

---


### 2.1 템플릿 메서드 패턴과 인터페이스 활용
Java의 인터페이스를 활용하여 구현체와의 직접적인 의존성을 줄일 수 있습니다.

```java
public interface ServiceInterface {
    void doSomething();
}

public class ServiceImpl implements ServiceInterface {
    @Override
    public void doSomething() {
        // 구현 내용
    }
}

public class ClassA {
    private ServiceInterface service; // 인터페이스를 참조
    
    public ClassA(ServiceInterface service) {
        this.service = service;
    }
}
```

#### 장점
- 구현체의 변경이 ClassA에 영향을 미치지 않음
- 인터페이스를 통한 느슨한 결합
- 테스트 용이성 향상

### 2.2 팩토리 패턴을 통한 개선
하지만 여전히 남아있는 문제가 있습니다. 인터페이스 타입의 구현체 클래스를 new 연산자를 통해 생성하는 경우, 의존성이 발생합니다. 이를 팩토리 패턴으로 해결할 수 있습니다.

```java
public class ServiceFactory {
    public static ServiceInterface createService() {
        return new ServiceImpl(); // 팩토리에서 구현체 생성
    }
}

public class ClassA {
    private ServiceInterface service;
    
    public ClassA() {
        this.service = ServiceFactory.createService(); // 팩토리를 통한 객체 생성
    }
}
```

#### 장점
- 구현체 생성 로직을 팩토리로 캡슐화
- ClassA는 구현체에 대한 직접적인 의존성이 제거됨
- 런타임 시점까지 객체 생성 지연 가능

## 3. IoC (Inversion of Control)와 DI (Dependency Injection)

### 3.1 IoC의 개념
기존의 객체지향 언어에서는 클래스가 자신이 필요로 하는 의존성을 직접 생성하고 관리했습니다. 그러나 IoC는 이러한 제어권을 프레임워크에 위임하는 것을 의미합니다.

### 3.2 Spring DI의 유형

### 

---

이러한 패턴을 이용해 Spring에서는 의존성 주입이라는 방법으로 IoC를 구현하고 있다. DI에서 자세히 살펴보면
- 생성자 주입, 필드 주입
- 메서드 주입, 세터 주입

이라는 방법이 존재하고 각각의 장단점이 있다.
이와 별개로 런타임 시점에 클래스에서 자신이 필요로 하는 구현체 클래스를 찾는 방법도 존재하는데 이를 DL이라 표현한다. XML로 구성된 설정 파일에서 Spring에서는 자신이 필요로 하는 클래스를 참조할 수 있다.

---


#### 3.2.1 생성자 주입
```java
@Service
public class UserService {
    private final UserRepository userRepository;
    
    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

#### 3.2.2 필드 주입
```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
}
```

#### 3.2.3 세터 주입
```java
@Service
public class UserService {
    private UserRepository userRepository;
    
    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

### 3.3 Dependency Lookup (DL)
런타임 시점에 필요한 구현체를 찾는 방법입니다.

```xml
<!-- applicationContext.xml -->
<bean id="userService" class="com.example.UserService">
    <property name="userRepository" ref="userRepository"/>
</bean>
```

## 4. 라이브러리 vs 프레임워크

라이브러리와 프레임워크의 차이점이 무엇일까? 서적에서는 애플리케이션의 실행흐름에 직접적으로 개입한다면 라이브러리 개발자가 정의한 파일을 통해서만 개입이 이루어지고 애플리케이션의 실행 흐름을 직접적으로 제어할 수 없다면 프레임워크라고 한다.

실제로 Spring에서 사용하는 Servlet을 생각해보면, Servlet은 개발자의 직접적인 개입 없이 프레임워크 내에서 돌아가는 것을 확인할 수 있다. 이렇듯 프레임워크가 요구하는 까다로운 IoC 전략에 따라 이러한 차이가 난다고 볼 수 있겠다.

### 4.1 주요 차이점
- **라이브러리**
  - 애플리케이션의 흐름을 직접적으로 제어
  - 개발자가 필요한 시점에 라이브러리의 기능을 호출
  - 흐름에 대한 주도권이 개발자에게 있음

- **프레임워크**
  - 애플리케이션의 흐름을 프레임워크가 제어
  - 개발자는 정해진 규약과 설정에 따라 코드를 작성
  - 프레임워크가 정의한 파일을 통해서만 개입 가능

### 4.2 Servlet 예시
```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        // 서블릿 컨테이너가 생명주기 관리
        // 개발자는 비즈니스 로직만 구현
    }
}
```

Servlet은 개발자의 직접적인 개입 없이 프레임워크 내에서 동작하며, 이는 프레임워크가 요구하는 IoC 전략의 대표적인 예시입니다.

## 5. Spring Framework의 특징

### 5.1 POJO (Plain Old Java Object)
- 특별한 제약이 없는 일반 자바 객체
- 특정 프레임워크에 종속되지 않음
- 테스트와 유지보수가 용이

### 5.2 AOP (Aspect Oriented Programming)
- 횡단 관심사의 분리
- 로깅, 트랜잭션 관리 등을 모듈화
- 코드의 재사용성과 유지보수성 향상

### 5.3 PSA (Portable Service Abstraction)
- 환경과 세부 기술의 변경에 관계없이 일관된 방식으로 기술 접근
- 예: @Transactional, @Cacheable

## 6. 결론
Spring Framework는 IoC/DI를 통해 객체 간의 결합도를 낮추고, 다양한 디자인 패턴을 활용하여 확장성과 유지보수성을 높입니다. 
템플릿 메서드 패턴, 팩토리 패턴 등을 통해 의존성 문제를 해결하고, 프레임워크로서 애플리케이션의 전체적인 흐름을 제어하면서도 
POJO를 기반으로 한 유연한 개발을 가능하게 합니다. 특히 IoC 컨테이너를 통한 객체 생명주기 관리는 현대적인 엔터프라이즈 
애플리케이션 개발에서 핵심적인 역할을 수행합니다.


이 문서는 귀하가 제공한 내용과 기존 내용을 통합하여, Spring Framework의 핵심 개념과 디자인 패턴에 대해 더욱 포괄적으로 설명하고 있습니다. 특히 클래스 의존성 문제와 그 해결 방안에 대한 설명을 강화했으며, 라이브러리와 프레임워크의 차이점에 대해 더 자세히 다루었습니다. 추가로 보완이 필요한 부분이 있다면 말씀해 주세요.