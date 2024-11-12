# 템플릿

## 1. 개방 폐쇠 원칙 OCP

### OCP

확장에는 자유롭게 열려 있고 변경에는 굳게 닫혀 있다는 개체지향 설계의 핵심 원칙.
코드에서 어떤 부분은 변경을 통해 그 기능이 다양해지고 확장하려는 성질이 있고, 어떤 부분은 고정되어 변하지 않으려는 성질이 있어 목적과 시점에 따라 독립적으로 변경될 수 있는 효율적인 구조를 만들 수 있다.
템플릿에서는 변경이 거의 일어나지 않으며 일정한 패턴으로 유저되는 특성을 독립시켜 효과적으로 활용할 수 있도록 하는 방법.

---

### 1.1 코드의 성질
코드에는 변하는 코드와 변하지 않는 코드를 살펴볼 수 있는데, 서적의 경우 초난감 DAO를 예시로 들어 설명했다. 대표적인 try-catch-finally 블록을 살펴보자

**try-catch-finally:**
```java
public class 초난감DAO {
    
    public void deleteAll() {
        Connection conn = null;
        PreparedStatemet pstmt = null;

        try {
            conn = dataSource.getConnection();
            pstmt = conn.preparedStatement("delete from TABLE");

            int row = pstmt.executeUpdate();
        } catch(SQLException e) {

        } finally {
            conn.close();
            pstmt.close();
        }
    }
}
```

### 1.2 메소드 추출

여기서 데이터베이스 커넥션을 위한 conn을 할당한 뒤 작업을 마치면서 `conn.close()`가 호출되지 않는 경우를 생각해보자. 스프링에서는 데이터베이스 커넥션에 대해 커넥션 풀을 만들어 관리하게 되는데, 자원이 해제되지 않으면서 계속 커넥션을 점유하고 있는 상황이 만들어진다. 따라서 이러한 문제점을 해결하기 위해 finally 블록에 자원을 해제하는 코드가 필요한데, 서적에서는 이러한 코드블록이 변하지 않는 코드라고 표현했다.

이렇듯 변하지 않는 코드가 메서드 내에서 반복적으로 사용된다면 리팩토링 시 메소드 추출을 생각해볼 수 있다.

**메소드 추출:**
```java
public class 초난감DAO {

    private PreparedStatement makeStmt() throws SQLExpcetion {
        ...
    }
    private void closeSource() {
        ...
    }
}
```

메소드 추출을 통해 분리시킨 메소드를 다른 곳에서 재사용할 수 있어야 하는데, 분리된 메소드가 DAO 로직마다 새롭게 만들어서 확장해야 하는 부분처럼 변경되었다.
#

### 1.3 템플릿 메소드 패턴
추상 메소드를 정의해둔 뒤 변하지 않는 부분을 상위 클래스, 변하는 부분은 서브 클래스에서 오버라이드하는 방식을 생각해볼 수 있다. 그러나 새로운 DAO 로직을 만들어 낼 때마다 그에 맞는 상속을 통해 새로운 클래스를 만들어야 한다는 단점이 존재한다.
또 확장구조가 이미 클래스를 설계하는 시점에 고정되어 버린다는 점이 문제점이다.
#

### 1.4 전략 패턴의 적용
유연하면서도 확장성이 뛰어는 방법은 오브젝트를 둘로 분리하고 클래스 레벨에서는 인터페이스를 통해서만 의존하도록 만들 수 있다. 

**메소드 추출:**
```java
public interface StatementStrategy {

    PreparedStatement makeStmt(Connection conn) throws SQLException;

}

public class DeleteAllStatementStrategy implements StatementStrategy {
    
    @Override
    PreparedStatement makeStmt(Connection conn) throws SQLException {
        pstmt = conn.preparedStatement("delete from TABLE");
        return pstmt;
    }
}

```

PreparedStategy 전략을 확장해 DeleteAll 전략이 만들어졌다. 그러나 이를 사용하는 context에서 구체적인 전략 클래스를 사용하는 것보다도 OCP를 만족하기 위해서는 클라이언트와 컨텍스트를 분리할 필요가 있다. client(초난감DAO)라고 했을 때 전략 클래스를 전달해 사용하는 context를 별도로 구현해볼 수 있다는 이야기다. 서적의 이전 챕터에서도 살펴보았듯이 컨텍스트가 사용하는 인터페이스의 구현체에는 클라이언트가 정하는 것이 객체지향 원칙을 만족하는데 좋은 방법이라는 점을 알 수 있었다.

#
### 1.5 DI 적용을 위한 클라이언트/컨텍스트 분리
```java

public void jdbcContextWithStrategy(StatementStrategy stmt) {
    ...
    
    try {
        conn = dataSource.getConnection();
        pstmt = stmt.makePreparedSatement(conn);
        ...
    }
}


public Client {
    public void deleteAll() {
        aStatementStrategt stmt = new DeleteAllStatementStrategy();
        jdbcContextWithStrategy(stmt);
    }
}

```

클라이언트로부터 StatementStrategy 타입의 오브젝트를 제공받고 try-catch-finally 구조로 만들어진 컨텍스트 내에서 작업을 수행한다. PreparedStatement 역시 생성이 필요한 시점에 호출해서 사용한다고 볼 수 있다.


## 2. 전략 패턴의 최적화

만약 StatementStrategy의 다른 전략을 사용하는 DAO 메소드가 필요하다면 어떨까?
메소드의 개수만큼 구현 클래스를 만들어야 한다는 문제점이 존재한다. 외에도 StatementStrategy에 전달할 부가적인 정보가 있는 경우 이를 위해 오브젝트를 전달받는 생성자와 이를 전달해둘 인스턴스 변수를 만들어야 한다. 이 오브젝트가 사용되는 시점은 컨텍스트가 전략 오브젝트를 호출할 때이므로 잠시라도 어딘가에 저장해 둘 필요가 있기 때문이다.
#

### 로컬 클래스
전략 클래스를 매번 독립된 파일로 만들지 않고 Context 클래스 메서드 안에 내부 클래스로 정의하는 방법이 있다.

```java

public void deleteAll() throws SQLException {
    
    class DeleteAllStatementStrategy implements StatementStrategy {
        
        @Override
        public PreparedStatement makeStmt(Connection conn) {
            ...
        }
    }
}
```

이 경우 Context 메서드와 강하게 결합되지만, 특정 메서드에서만 사용하는 것이라면 내부 클래스를 이용하는 방법이 좋다.
메소드마다 추가해야 했던 클래스 파일을 하나 줄일 수 있다는 것도 장점이고, 내부 클래스의 특징을 이용해 로컬 변수를 바로 가져다 사용할 수 있다는 것도 큰 장점이다.

#
### 익명 내부 클래스
```java

    StatementStrategy st = new StatementStrategy() {
        @Override
        public PreparedStatement makeStmt(Connection conn) throws SQLException {
            ...
        }
    }

```

익명 내부 클래스는 선언과 동시에 오브젝트를 생성한다.
익명 내부 클래스는 어떻게 템플릿과 같이 사용될 수 있을까? 우리는 여기서 템플릿과 콜백 패턴에 대해 알아야 한다.

## 3. 템플릿과 콜백 패턴
템플릿/콜백 패턴의 콜백은 보통 단일 메소드 인터페이스를 사용한다. 템플릿의 작업 흐름 중 특정 기능을 위해 한 번 호출되는 경우가 일반적이기 때문이다.
따라서 콜백은 일반적으로 하나의 메소드를 가진 인터페이스를 구현한 익명 내부 클래스로 만들어진다고 볼 수 있다.

```java

    public void deleteAll() throws SQLException {
        Context.workWithStatementStrategy(
            new StatementStrategy() {
                @Override
                public PreparedStatement makeStmt(Connection conn) throws SQLException {
                    return conn.prepareStatement("delete from TABLE");
                }
            }
        )
    }

```
여기서도 자주 바뀌는 부분을 확인해볼 수 있는데, query 별로 다른 기능을 실행하는 메서드가 필요했을 때, 콜백 함수를 메소드마다 만들어야 한다는 문제점이 생긴다.
따라서 파라미터를 통해 메서드를 분리해보는 방법을 생각해볼 수 있다.

```java

    public void deleteAll() {
        executeSQL("delete from TABLE");
    }

    public void executeSQL(String query) throws SQLException {
        Context.workWithStatementStrategy(
            new StatementStrategy() {
                @Override
                public PreparedStatement makeStmt(Connection conn) throws SQLException {
                    return conn.prepareStatement(query);
                }
            }
        )
    }

```

#

### 4. 코드를 이용한 DI
만약 인터페이스를 사용하지 않고 DI를 적용하는 경우를 생각해보자. 이 경우 DI를 만족한다고 볼 수 있을까?
스프링의 DI를 넓은 의미에서 해석하면 객체의 생성과 관계 설정에 대한 제어 권한을 외부로 위임했다는 IoC라는 개념을 포괄한다.
이런 의미에서 인터페이스를 사용하지 않아도 DI의 기본을 따르고 있다고 볼 수 있다.

만약 인터페이스를 사용하지 않은 오브젝트가 스프링 빈을 의존하고 있는 경우라면 스프링 빈으로 DI가 이루어져야 한다. 스프링 빈에서는 스프링이
생성하고 관리하는 IoC 대상이어야 DI에 참여할 수 있기 때문에 스프링 빈으로의 설정이 이루어져야 한다.
반대로 싱글톤 패턴을 포기하면서 오브젝트 내부에서 직접 DI를 적용하는 코드를 이용한 DI 방법이 존재한다. 자신이 사용할 오브젝트를 직접 만들고 초기화하는 전통적인
방법을 사용하는 것이다. 내부에 두는 상태정보가 존재한다고 가정한다면 이러한 DI 패턴이 좋은 방법일 수 있다.

```xml

<beans>
    <bean id="dao" class="Dao">
        <property name="dataSource" ref="dataSource" />
    </bean>

    <bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
        ...
    </bean> 
</beans>

```

```java

    public class Dao {
        private Context context;

        public void setDataSource(DataSource source) {
            this.context = new Context();
            context.setDataSource(source);
        }
    }

```

`setDataSource()` 메서드는 DI 컨테이너가 DataSource 오브젝트를 주입해줄 때 호출된다. 이 때 수동 DI 작업을 진행하면 코드를 이용한 DI를 적용할 수 있다.
context의 경우 외부에서 빈으로 만들어져 주입된 것인지, 내부에서 직접 만들고 초기화한 것인지 구분할 수 없지만 필요에 따라 context를 사용할 수 있다.


- **빈을 이용한 DI**
    - 장점: 오브젝트 사이의 실제 의존 관계가 명확하게 드러남
    - 단점: 구체적인 클래스와의 관계가 설정에 직접 노출

- **코드를 이용한 DI**
    - 장점: 인터페이스를 따로 두지 않아도 될 긴밀한 오브젝트를 어색하게 빈으로 분리하지 않고 내부에서 직접 만들어 DI를 적용
    - 단점: 여러 오브젝트가 사용하더라도 싱글톤으로 만들 수 없고 DI 작업을 위한 부가적인 코드가 필요

..


