---
title:  "[Spring Boot] 스프링 입문 - 코드로 배우는 스프링 부트, 웹MVC, DB 접근 기술 (인프런) #4 (Repository)"
date:   2022-04-02 23:00:00 +0900
categories: [post]
tags:
- all
- study
- lecture
- spring
---
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

먼저 Memory를 이용해서 간단한 회원 클래스를 정의했다. 인텔리J를 사용하면 최초에 MemberRepository를 implements 하도록 클래스만 만들어주면 자동으로 구현해야할 인터페이스의 메서드들의 껍데기를 만들수 있도록 해준다. 

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