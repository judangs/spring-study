## JUnit 테스트

#### JUnit 테스트
 - 프레임워크
 - public 메소드 사용
 - @Test annotation 부여


##### \* 주요 메소드와 어노테이션

 assertThat()
 - 테스트 성공 유무를 알려줌
 - 실패 시 AssertionError 발생
 
 ```
 assertThat(user2.getName(), is(user.getName()));
 ```
예외조건 테스트
- @Test에 expected 엘리먼트 추가
- expected: 테스트 메소드 실행 중 발생할 것이라고 기대하는 예외 클래스 명시
```
@Test(expected=EmptyResultDataAccessException.class)
public void getUserFailure() throws SQLException{
    ...
}
```

@Before과 @After
- 테스트 메소드의 공통적인 준비 작업 및 정리 작업을 각각 @Before과 @After에 넣음으로써 테스트 코드가 간결해짐(ex. setUp())
- @Test가 붙은 메소드 실행 전과 후에 각각 @Before과 @After이 자동 실행됨
- @Before과 @After 메소드를 테스트 메소드 안에서 직접 호출하진 않음
    * 주고받을 오브젝트가 있다면 인스턴스 변수를 사용해야 함
- 각각의 테스트를 실행할 때마다 @Before과 @After도 각각의 테스트 전후로 실행됨

@BeforeClass
- 테스트 클래스 전체에 걸쳐 딱 한 번 실행
- 애플리케이션 컨텍스트를 만들어 스태틱 변수에 저장함
- 자주 사용x(스프링의 컨텍스트 테스트를 더 많이 사용)

스프링 테스트 컨텍스트 프레임워크
- @Autowired: ApplicationContext 인스턴스 변수에 선언
```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/applicationContext.xml")
public class UserDaoTest{

    @Autowired
    private ApplicationContext context;
    ...

}
```
- 테스트 메소드가 컨텍스트를 공유할 수 있고, 테스트 클래스가 하나의 컨텍스트를 공유할 수 있게 함

@Autowired
- 스프링 DI에 사용됨(별도 DI 설정 없이 빈을 자동으로 가져옴)
- 테스트에 사용시, 변수 타입과 일치하는 컨텍스트 내의 빈을 찾아 인스턴스 변수에 주입함


##### 테스트 특징
- 단위 테스트는 일관성 있는 결과가 보장돼야 한다.

    -> DB에 남아 있는 데이터와 같은 외부 환경에 영향을 받지 않아야 한다.

    -> 테스트 실행 순서가 바뀌어도 동일한 결과가 보장돼야 한다.
- 포괄적인 테스트를 만들어야 한다. 

    -> "네거티브 테스트"를 먼저 만들어야 한다.
    
    -> 예외적인 상황을 빠뜨리지 않는 테스트를 작성해야 한다.

##### TDD(테스트 주도 개발)
- 방식: 테스트를 먼저 만들고, 테스트가 성공하도록 하는 코드만 만드는 방식으로 진행
- 테스트를 빼먹지 않고 꼼꼼하게 만들 수 있음
- 테스트 작성 시간 - 애플리케이션 코드 작성 시간 간격 짧아짐
- 자연스럽게 단위 테스트를 작성할 수 있음
- 코드를 만들어 테스트를 실행하는 간격이 짧기 때문에 오류를 빠르게 발견할 수 있음

##### DI를 테스트에 사용하는 3가지 방법
1. 테스트 코드에 의한 DI
    * DirtiesContext
        : 스프링 테스트 컨텍스트 프레임워크에 해당 클래스의 테스트에서 애플리케이션의 컨텍스트의 상태를 변경한다는 것을 알려줌
```
@DirtiesContext
public class UserDaoTest{
    ...
    @Before
    public void setUp(){
        DataSource dataSource = new SingleConnectionDataSource("jdbc:mysql://localhost/testdb", "spring", "book", true);
        dao.setDataSource(dataSource);
    }
}
```
2. 테스트를 위한 별도의 DI 설정
    * 테스트에서 사용될 클래스가 빈으로 정의된 테스트 전용 설정파일을 만드는 방법

3. 컨테이너 없는 DI 테스트
    * 스프링 컨테이너 없이 테스트를 생성하는 방법


##### 학습 테스트
본인이 만들지 않은 프레임워크나 라이브러리 등에 대해서 작성하는 테스트

**장점**
- 다양한 조건에 따른 기능을 손쉽게 확인해볼 수 있음
- 학습 테스트 코드를 개발 중에 참고할 수 있음
- 프레임워크나 제품 업그레이드 시 호환성 검증을 도와줌
- 테스트 작성에 훈련이 됨

##### 버그 테스트
코드에 오류가 있을 때 그 오류를 가장 잘 드러낼 수 있는 테스트

*일단 실패하도록 만들어야 함*

**장점**
- 테스트의 완성도를 높임
- 버그의 내용을 명확히 분석하게 해줌
- 기술적 문제 해결에 도움이 됨