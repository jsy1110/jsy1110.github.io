---
title:  "[Spring Boot] 스프링 입문 - 코드로 배우는 스프링 부트, 웹MVC, DB 접근 기술 (Repository) #4 (Repository)"
date:   2022-04-02 23:00:00 +0900
categories: [post]
tags:
- study
- lecture
- spring
---
# Repository 구현

### 1. 메모리 클래스 사용

앞에서 본것과 같이 멤버 리포지토리는 Interface로 구성하여 향후 사용하는 DB에 따라 구현 클래스를 변경한다. 멤버 리포지토리의 interface는 아래와 같이 정의했다. 당연한 이야기지만 향후에 어떤 DB를 사용해서 구현하든 아래의 메서드는 반드시 구현되어야 한다.

![Untitled](https://user-images.githubusercontent.com/6336815/161395500-ef972668-1682-491a-a76b-a9fbdaaea8bf.png)

```java
public interface MemberRepository {

    Member save(Member member);

    Optional<Member> findById(Long id);

    Optional<Member> findByName(String name);

    List<Member> findAll();

}
```

```java
public class MemoryMemberRepository implements MemberRepository {

    private static Map<Long, Member> store = new HashMap<>();
    private static long sequence = 0L;

    @Override
    public Member save(Member member) {
        member.setId(++sequence);
        store.put(member.getId(), member);
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        return Optional.ofNullable(store.get(id));
    }

    @Override
    public Optional<Member> findByName(String name) {
        return store.values().stream()
                .filter(member -> member.getName().equals(name))
                .findAny();
    }

    @Override
    public List<Member> findAll() {
        return new ArrayList<>(store.values());
    }

    public void clearStore() {
        store.clear();
    }
}
```

먼저 Memory를 이용해서 간단한 회원 클래스를 정의했다. 인텔리J를 사용하면 최초에 MemberRepository를 implements 하도록 클래스의 껍데기만 만들어주면 자동으로 구현해야할 인터페이스의 메서드들의 껍데기를 만들어준다. 

![Animation](https://user-images.githubusercontent.com/6336815/161395497-1a990617-bb3c-4a51-bf7f-cef52132714f.gif)

### 2. 실제 DB 사용

해당 강의에서는 H2라는 실습용 DB를 이용해서 위에서 만든 Memory 리포지토리를 대체했다. 해당 글에서는 간단히 `대체했다` 라는 한마디로 표현하고 있지만 이런 상황이 실무에서 일어났다고 생각하면 끔찍할 것이다. 하지만 해당 강의의 예제는 DB가 대체되는 상황에서도 코드 변경을 최소화 하면서 이를 수행한다. 객체지향이라는 개념은 코드가 유지보수되고 확장되는 순간에 빛을 발한다.

*물론 현실에서 DB가 변경될 경우 예제와 같이 간단할 수는 없다. DB, WAS 등 전체적인 구조를 손봐야 하기 때문이다. 현업에서는 이런 상황이 일어나지 않기를 바란다...하지만 그나마 JAVA 코드는 위와 같이 대체 될 수 있다는 것이다.*

H2 는 설치 후 `build.gradle` 파일에 h2 관련 라이브러리를 추가하고, `application.properties` 파일에 DB 연결 정보를 추가해 주는것으로 설정이 끝난다.

![Untitled 1](https://user-images.githubusercontent.com/6336815/161395499-7554f72b-51fe-4802-8152-a612263d42ce.png)

### 3. JDBC → 스프링 JdbcTemplate → JPA → Spring data JPA

 해당 강의에서는 순수 Jdbc 를 사용하는 쿼리 구현부터 Spring data JPA 를 이용한 구현을 보여준다. JPA를 공부하는 개발자들에게 `JPA를 사용하는 것이 어떤 장점이 있는가?` 를 설득하기에 매우 적절한 예제였다. 김영한님도 강의를 하면서 언급하셨지만 JDBC 를 사용하는 것은 매우 고통스럽다. 모든 페이지마다 DB connection을 가져오고, try ~ catch 문이 덕지덕지 붙는다. 그리고 String 으로 만들어져서 +를 이용해서 줄바꿈을 하면서 덕지덕지 100줄 이상 늘어나버린 Query를 봐야하는 상황이 되면 한숨부터 나온다. 

*놀랍게도 나는 그 짓(?)을 비교적 최근인 2019년까지 해왔다. 김영한님은 “편하게~~~~ 들으십시오” 하셨지만 나와 같이 이것을 경험해봤던 사람이라면 닭살이 돋았을 수도 있다.*

### 순수 JDBC

자세한 설명은 생략한다. DB 커넥션부터 상세 query 작성까지 개발자가 한땀한땀 작성하고 있다 정도로 이해하면 될것 같다.

```java
/**
 * 순수 JDBC를 이용한 save 메서드 구현
 */

public class JdbcMemberRepository implements MemberRepository {

    //SpringConfig 에서 직접 @Bean 설정으로 주입
    private final DataSource dataSource;
    public JdbcMemberRepository(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    @Override
    public Member save(Member member) {
        String sql = "insert into member(name) values(?)";
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        try {
            conn = getConnection();
            pstmt = conn.prepareStatement(sql,
                    Statement.RETURN_GENERATED_KEYS);
            pstmt.setString(1, member.getName());
            pstmt.executeUpdate();
            rs = pstmt.getGeneratedKeys();
            if (rs.next()) {
                member.setId(rs.getLong(1));
            } else {
                throw new SQLException("id 조회 실패");
            }
            return member;
        } catch (Exception e) {
            throw new IllegalStateException(e);
        } finally {
            close(conn, pstmt, rs);
        }
    }
		...중략

    private Connection getConnection() {
        return DataSourceUtils.getConnection(dataSource);
    }
    private void close(Connection conn, PreparedStatement pstmt, ResultSet rs)
    {
        try {
            if (rs != null) {
                rs.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        try {
            if (pstmt != null) {
                pstmt.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        try {
            if (conn != null) {
                close(conn);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    private void close(Connection conn) throws SQLException {
        DataSourceUtils.releaseConnection(conn, dataSource);
    }
}
```

 

### Spring JdbcTemplate

이후에 나온 스프링의 JdbcTemplate은 순수 JDBC 사용시 반복되는 코드를 최소화 할 수 있도록 기능을 제공해준다. DB 커넥션을 가져오고, SQL 실행에 따른 exception 관련 코드들을 제거할 수 있게 도와준다. 사용하는 query 또한 조금은 JAVA스럽게 만들 수 있도록 메서드들을 제공한다. 하지만 어딘가 불편하고, 대부분의 상황에서 여전히 query를 string으로 작성할 수 밖에 없다.

```java
/**
 * 스프링 JdbcTemplate를 이용한 메서드 구현
 */
public class JdbcTemplateMemberRepository implements MemberRepository {
    private final JdbcTemplate jdbcTemplate;

    public JdbcTemplateMemberRepository(DataSource dataSource) {
        jdbcTemplate = new JdbcTemplate(dataSource);
    }
    @Override
    public Member save(Member member) {
        SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(jdbcTemplate);
        jdbcInsert.withTableName("member").usingGeneratedKeyColumns("id");
        Map<String, Object> parameters = new HashMap<>();
        parameters.put("name", member.getName());
        Number key = jdbcInsert.executeAndReturnKey(new
                MapSqlParameterSource(parameters));
        member.setId(key.longValue());
        return member;
    }
    @Override
    public Optional<Member> findById(Long id) {
        List<Member> result = jdbcTemplate.query("select * from member where id = ?", memberRowMapper(), id);
        return result.stream().findAny();
    }
    @Override
    public List<Member> findAll() {
        return jdbcTemplate.query("select * from member", memberRowMapper());
    }
    @Override
    public Optional<Member> findByName(String name) {
        List<Member> result = jdbcTemplate.query("select * from member where name = ?", memberRowMapper(), name);
        return result.stream().findAny();
    }
    private RowMapper<Member> memberRowMapper() {
        return (rs, rowNum) -> {
            Member member = new Member();
            member.setId(rs.getLong("id"));
            member.setName(rs.getString("name"));
            return member;
        };
    }
}
```

### 순수 JPA(Java Persistence API)

JPA는 Entity Manager를 이용해서 엔티티와 영속성(persist)을 맺고, 영속성 컨텍스트를 관리하며 트랜잭션을 제어한다. DB 구조와 상관없이 최대한 JAVA 객체의 관점으로 DB까지 컨트롤 하겠다는 것이다. 여전히 SQL 문과 비슷한 JPQL이라는 String 객체 만들어 query를 사용하고 있지만 코득가 꽤나 간결해졌다.

*아래 코드에서 EntityManager 객체는 직접 생성하지 않고, 외부에서 주입받는다. 이를 `DI(Dependency Injection), 의존성 주입` 이라고 부른다. 상세한 내용은 다른 포스트를 통해 설명한다.*

```java
public class JpaMemberRepository implements MemberRepository {
    private final EntityManager em;
    public JpaMemberRepository(EntityManager em) {
        this.em = em;
    }
    public Member save(Member member) {
        em.persist(member);
        return member;
    }
    public Optional<Member> findById(Long id) {
        Member member = em.find(Member.class, id);
        return Optional.ofNullable(member);
    }
    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }
    public Optional<Member> findByName(String name) {
        List<Member> result = em.createQuery("select m from Member m where m.name = :name", Member.class)
                .setParameter("name", name)
                .getResultList();
        return result.stream().findAny();
    }
}
```

### Spring Data JPA

Spring Data JPA를 사용하면 위와 같은 default들은 굳이 구현할 필요도 없다. 사용하는 방법 또한 JpaRepository 인터페이스를 상속받는 것으로 해결이 된다. FindByName 메서드 또한 `FindBy...` 방식으로 ... 대신에 필요한 변수값을 넣으면 된다.

```java
public interface SpringDataJpaMemberRepository extends JpaRepository<Member, Long>, MemberRepository {

    @Override
    Optional<Member> findByName(String name);
}
```

### 4. Spring Data JPA vs. MyBatis

스프링 JdbcTemplate 이후 국내의 상황은 MyBatis 를 사용하는 쪽으로 발전해왔다. 국내 대부분의 기업들이 MyBatis를 사용했고, 본인은 아직까지 현업에서 MyBatis를 사용하고 있다. 확실히 Jdbc를 직접 사용할 때에 비해 MyBatis를 사용하면서 생산성이 훨씬 올랐고, 유지보수 업무를 하는게 많이 편해졌다. 김영한님께서 [자바 ORM 표준 JPA 프로그래밍](http://www.yes24.com/Product/Goods/19040233) 이라는 책을 출간할 때만해도 국내에 JPA가 많이 알려지지 않았다고 한다. 하지만 이 책이 출간 되면서 국내 빅테크 기업들 사이에서 JPA 사용이 늘어나기 시작했고, 현재 시점에는 스프링을 사용하는 99%의 빅테크 기업은 JPA 사용하는 것으로 대세가 넘어간 것 같다.

기술의 대세를 바꾸는 것은 쉽지 않다. 특정 상황에서 어떤 기술을 사용할지 결정해야 하는 상황은 누가 답을 정해줄 수 없는 문제이다. 그렇기 때문에 해당 상황에서 어떤 기술을 사용할지는 전적으로 개발자의 성향에 달려있다. 하지만 선택에 대한 주요 요인은 결국 `본인이 얼마나 그 기술에 익숙한가`가 크게 작용하게 된다. 예전에 했던 프로젝트에서도 Node.js와 Spring 사이에서 backend 기술스택을 결정해야했는데 나는 결국 Spring을 선택했다. 몇가지 요인이 있었지만 보안성을 제외하고도 아래의 요인이 결정적이었다.

1. **본인이 JAVA에 익숙하다.**
2. **관련 서적 등 국내에 공부할 수 있는 자료의 양이 Spring이 많다.**

지금은 많은 빅테크 기업들이 JPA를 사용하면서 JPA가 복잡한 쿼리까지 충분히 대체할 수 있다는 수많은 증거가 있지만 그 당시에는 레퍼런스가 없었기 때문에 MyBatis를 포기하지 못했던 개발자들의 마음이 이해가 된다. 솔직히 현재 MyBatis를 사용하고 있는 입장에서 MyBatis를 JPA로 대체하는 시도는 하지 않을 것이다. 아직까지 공부중인 JPA보다는 쿼리가 한눈에 보이는 MyBatis가 편하다. DB 테이블이 많아지고, 쿼리가 복잡해질수록 더욱 JPA로의 변경은 힘들다.

그런데 연차가 쌓일수록 이런 고민이 늘고 있다. 대부분의 주니어 개발자의 경우 SQL 작성 능력이 떨어진다. 이런 상황은 최근 들어 더욱 심해지고 있다고 생각한다. MyBatis를 사용하고 있는 현업의 경우 결국 주니어 개발자들이 스스로 SQL을 익힐때까지 생산성이 떨어질 수 밖에 없다. 사람에 따라 다르지만 SQL 또한 새로운 언어라고 본다면 SQL 문법을 새로 익히는것은 쉽지 않을 것이다. 아예 두 기술을 모르는 상태라면 MyBatis보다 JPA를 배우는 것이 쉬울 것이다. MyBatis를 익힌다는 것은 MyBatis의 동작원리 + SQL 문법 모두를 배운다는 의미이기 때문이다. 그렇기 때문에 코드 생산성이 중요하고, 주니어 개발자 비율이 높은 초기 스타트업은 JPA를 사용하는 것이 자연스럽다. 현재는 Spring 프로젝트에서 Spring Data JPA까지 만들면서 JPA를 대세로 굳혀줬기 때문에 고민할 필요도 없다.

하지만 개발이 아닌 운영, 유지보수가 주 업무가 되는 SM 단계에서 JPA가 MyBatis보다 큰 강점을 가질까를 생각해보면 아직 잘 모르겠다. 기업 SM을 하게 되면 자료 추출과 DB 데이터 변경이 빈번히 일어나는데 SQL을 모르면 업무가 불가능하기 때문에 어쩔 수 없이 SQL을 알아야 한다. 또한, JPA에서도 JPQL 이나 QueryDSL 등의 특정 문법을 추가로 배워야 하기 때문에 알아야  할 범위가 늘어날 수 있다. 결국 정답은 없지만 Spring에서 대세를 JPA로 찍어줬기 때문에 조만간 MyBatis 또한 잊혀진 기술이 될 가능성이 높기 때문에 이제 개발을 시작하는 개발자의 경우는 JPA를 우선적으로 배우는 것이 맞다.