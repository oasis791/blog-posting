![김영한-JPA](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/JPA%EB%A9%94%EC%9D%B8.png)

> 본 포스팅의 이미지 저작권은 [자바 ORM 표준 JPA 프로그래밍 - 기본편 (김영한)](https://www.inflearn.com/course/ORM-JPA-Basic) 강의에 있습니다.

## 일대다(1:N) 단방향
- 일(1)이 연관관계 주인이다. (1방향에서 외래키를 관리하겠다)
- 권장하지는 않는 스펙이다.

![](https://raw.githubusercontent.com/oasis791/blog-posting/975ec84260ee1bc94f6fd607299b82f6a448b7c5/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/6.%EB%8B%A4%EC%96%91%ED%95%9C%20%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84%20%EB%A7%A4%ED%95%91/2.%EC%9D%BC%EB%8C%80%EB%8B%A4%20%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84/1.png)

- 일(1)쪽에서 참조를 생성하여 외래키를 관리하는 **연관관계 주인**이 된다.
- **DB는 객체와 상관없이 무조건 다(N)쪽에 외래키가 들어가야 한다.**
- 따라서, Team의 `List members`값이 변경되었을 때, 다른 테이블의 외래키가 업데이트 되어야한다.

```java
@Entity 
public class Team {
    @Id @GeneratedValue 
    @Column(name = "TEAM_ID")
    private Long id;
    
    private String name;
    
    @OneToMany 
    @JoinColumn(name = "TEAM_ID")
    private List<Member> members = new ArrayList<>();

	//생성자 및 Getter, Setter
	
}
```

Team 클래스에서, `@OneToMany` 애노테이션으로 일대다(1:N) 연관관계를 지정해주고, `@JoinColumn` 애노테이션을 통해 연관관계의 주인을 지정했다.

```java
	Member member = new Member();
	member.setName("member1");
	em.persist(member);
	
	Team team = new Team();
	team.setName("teamA");
	// 외래키를 세팅해주기 위해
	team.getMembers().add(member);
	em.persist(team);
	
	tx.commit();
```

멤버를 생성하고, 팀을 생성할 때, 외래키를 세팅해주기 위해서 members에 생성된 멤버를 추가한다.

```sql
Hibernate: 
    /* insert hellojpa.Member
        */ insert 
        into
            Member
            (USERNAME, MEMBER_ID) 
        values
            (?, ?)
Hibernate: 
    /* insert hellojpa.Team
        */ insert 
        into
            Team
            (name, TEAM_ID) 
        values
            (?, ?)
Hibernate: 
    /* create one-to-many row hellojpa.Team.members */ update
        Member 
    set
        TEAM_ID=? 
    where
        MEMBER_ID=?
```

앞서 생성한대로, member와 team의 INSERT 쿼리가 생성되었다.
Team 객체에서 외래키를 관리하고 있지 않기 때문에, 외래키를 갖고 있는 MEMBER 테이블에 UPDATE 쿼리가 실행됨으로써, MEMBER 데이터에 TEAM 값을 세팅해줄 수 있게 된다.

![](https://raw.githubusercontent.com/oasis791/blog-posting/975ec84260ee1bc94f6fd607299b82f6a448b7c5/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/6.%EB%8B%A4%EC%96%91%ED%95%9C%20%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84%20%EB%A7%A4%ED%95%91/2.%EC%9D%BC%EB%8C%80%EB%8B%A4%20%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84/2.png)

데이터베이스에 의도한대로 데이터가 들어감을 확인할 수 있다.

## 일대다(1:N) 단방향 정리
- 일대다(1:N) 단방향은 일대다에서 **일(1)이 연관관계의 주인이다.**
- 그렇지만, DB의 테이블 일대다 관계는 항상 **다(N) 쪽에 외래키가 존재한다.**
- 객체와 테이블의 패러다임 차이 때문에 반대편 테이블의 외래키를 관리하는 특이한 구조이다.
- `@JoinColumn`을 꼭 사용해야 한다.
	- 사용하지 않으면, 조인 테이블 방식(중간에 테이블 하나 추가)을 사용한다.
	- 조인 테이블의 장점도 있지만, 성능이 저하될 수 있고, 운영이 힘들어질 수 있다.

## 일대다 단방향 매핑의 단점
- **엔티티가 관리하는 외래키가 다른 테이블에 있다.**
- 연관관계 관리를 위해 추가로 UPDATE SQL이 실행된다.

**일대다 단방향 매핑보다는 다대일 양방향 매핑을 사용하자**

## 일대다 양방향
![](https://raw.githubusercontent.com/oasis791/blog-posting/975ec84260ee1bc94f6fd607299b82f6a448b7c5/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/6.%EB%8B%A4%EC%96%91%ED%95%9C%20%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84%20%EB%A7%A4%ED%95%91/2.%EC%9D%BC%EB%8C%80%EB%8B%A4%20%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84/3.png)

```java
@Entity 
public class Member {
    @Id @GeneratedValue 
    @Column(name = "MEMBER_ID")
    private Long id;
    
    @Column(name = "USERNAME")
    private String name;
    
    @ManyToOne 
    @JoinColumn(name = "TEAM_ID", insertable = false, updatable = false)
    private Team team;

	// 생성자 및 Getter, Setter
}
```

`@ManyToOne` 애노테이션을 통해 연관관계를 맺어주나, `insertable = false` 속성과 `updatable = false` 속성으로 데이터의 변경과 삽입을 막음으로써 읽기 전용 매핑 처럼 동작하게 한다.

## 일대다 양방향 정리
- 이러한 매핑은 **공식적인 스펙에 존재하지 않는다.**
- **읽기 전용 필드**를 사용해서 양방향처럼 사용하는 방법이다.

**다대일 양방향을 사용하자!!!**