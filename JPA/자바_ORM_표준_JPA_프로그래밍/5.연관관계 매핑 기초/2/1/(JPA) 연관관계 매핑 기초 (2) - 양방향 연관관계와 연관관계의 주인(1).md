![김영한-JPA](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/JPA%EB%A9%94%EC%9D%B8.png)

> 본 포스팅의 이미지 저작권은 [자바 ORM 표준 JPA 프로그래밍 - 기본편 (김영한)](https://www.inflearn.com/course/ORM-JPA-Basic) 강의에 있습니다.

## 양방향 매핑

![](https://raw.githubusercontent.com/oasis791/blog-posting/12063087704ed75e003a2c781c56d6964c4d5859/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/5.%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84%20%EB%A7%A4%ED%95%91%20%EA%B8%B0%EC%B4%88/2/1/1.png)

객체와 테이블이 양방향으로 연관관계가 이루어져 있다. 양방향의 연관관계를 맺으면 어느쪽에서든 참조할 수 있다.
-> 양방향 연관관계를 맺기 위해서 객체간에는 변경이 있지만 **테이블 연관관계는 단방향 연관관계와 차이가 없다.**
-> JOIN을 통해 양방향 연관관계를 얼마든지 표현 가능하다. (**테이블 연관관계에서는 방향이라는 개념이 없다.**)

## 객체에서 양방향 연관관계 맺기

```java
@Entity 
public class Member {
    @Id @GeneratedValue 
    @Column(name = "MEMBER_ID")
    private Long id;
    
    @Column(name = "USERNAME")
    private String name;
    
    @ManyToOne 
    @JoinColumn(name = "TEAM_ID")
    private Team team;

	//Getter, Setter, Constructor
}
```

N:1 관계에서 N에 해당하는 Member는 `@ManyToOne` 애노테이션으로 연관관계를 맺어준다. `@JoinColumn` 애노테이션을 통해 JoinColumn이 될 컬럼을 설정해 준다.

```java
@Entity 
public class Team {
    @Id @GeneratedValue 
    @Column(name = "TEAM_ID")
    private Long id;
    private String name;
    
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
    
    //Getter, Setter, Constructor
}
```

N:1 관계에서 1에 해당하는 Team은 `@OneToMany` 애노테이션과 컬렉션을 통해 연관관계를 맺어준다. `mappedBy` 속성으로 연관관계가 맺어질 필드 값을 추가해준다. (mappedBy 속성은 이후에 설명하겠다.)

관례상 List는 `new ArrayList<>();` 로 초기화 해주어, nullPointException을 피한다.

```java
// 팀 저장  
Team team = new Team();  
team.setName("TeamA");  
em.persist(team);  
  
// 회원 저장  
Member member = new Member();  
member.setName("member1");  
member.setTeam(team);  
em.persist(member);  
  
em.flush();  
em.clear();  
  
// 팀 조회  
Member findMember = em.find(Member.class, member.getId());  
List<Member> members = findMember.getTeam().getMembers();  
  
for (Member m : members) {  
	System.out.println("m = " + m.getName());  
}  
  
tx.commit();
```

조회 한 회원인 `findMember`에서 `getTeam()` 메소드를 이용해 소속 된 팀을 확인할 수 있고, `getMemebers()` 메소드로 해당 팀에 소속된 회원들을 `List` 형태로 확인 할 수 있다.

### 실행결과

```sql
Hibernate: 
    call next value for hibernate_sequence
Hibernate: 
    call next value for hibernate_sequence
Hibernate: 
    /* insert hellojpa.Team
        */ insert 
        into
            Team
            (name, TEAM_ID) 
        values
            (?, ?)
Hibernate: 
    /* insert hellojpa.Member
        */ insert 
        into
            Member
            (USERNAME, TEAM_ID, MEMBER_ID) 
        values
            (?, ?, ?)
Hibernate: 
    select
        member0_.MEMBER_ID as MEMBER_I1_0_0_,
        member0_.USERNAME as USERNAME2_0_0_,
        member0_.TEAM_ID as TEAM_ID3_0_0_,
        team1_.TEAM_ID as TEAM_ID1_1_1_,
        team1_.name as name2_1_1_ 
    from
        Member member0_ 
    left outer join
        Team team1_ 
            on member0_.TEAM_ID=team1_.TEAM_ID 
    where
        member0_.MEMBER_ID=?
Hibernate: 
    select
        members0_.TEAM_ID as TEAM_ID3_0_0_,
        members0_.MEMBER_ID as MEMBER_I1_0_0_,
        members0_.MEMBER_ID as MEMBER_I1_0_1_,
        members0_.USERNAME as USERNAME2_0_1_,
        members0_.TEAM_ID as TEAM_ID3_0_1_ 
    from
        Member members0_ 
    where
        members0_.TEAM_ID=?
        
m = member1
```

최하단의 결과를 보면 성공적으로 결과값이 출력된 것을 알 수 있다.

-> GeneratedValue로 생성된 ID값은 `INSERT` 쿼리로 데이터베이스에 반영되어야 한다. 따라서, `em.flush()`를 통해 데이터베이스에 쿼리를 전송하고, `em.clear()` 를 통해 1차 캐시에 존재하는 데이터들을 초기화 시킨다. 이후에 `SELECT` 쿼리를 통해 데이터베이스에서 정상적으로 데이터를 가져오는 것을 확인할 수 있다.

## 연관관계의 주인과 mappedBy
- **객체와 테이블간에 연관관계를 맺는 차이**를 이해해야 한다.

### 객체의 양방향 관계

![](https://raw.githubusercontent.com/oasis791/blog-posting/12063087704ed75e003a2c781c56d6964c4d5859/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/5.%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84%20%EB%A7%A4%ED%95%91%20%EA%B8%B0%EC%B4%88/2/1/2.png)

- 객체의 양방향 관계는 **사실 양방향 관계가 아니라 서로 다른 2개의 단방향 관계이다.**
- 객체를 양방향으로 참조하려면 **단방향 연관관계를 2개** 만들면 된다.

```java
class A {
	B b;
}

class B {
	A a;
}
```

- A -> B로 참조하기 위해
	- `a.getB();`
- B -> A로 참조하기 위해
	- `b.getA();`

### 테이블의 양방향 관계

![](https://raw.githubusercontent.com/oasis791/blog-posting/12063087704ed75e003a2c781c56d6964c4d5859/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/5.%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84%20%EB%A7%A4%ED%95%91%20%EA%B8%B0%EC%B4%88/2/1/3.png)

- 테이블은 **외래 키 하나**로 두 테이블의 연관관계를 관리한다.
- MEMBER.TEAM_ID 외래 키 하나로 양방향 연관관계를 가질 수 있다.

```sql
SELECT *
FROM MEMBER M
JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
```

```sql
SELECT *
FROM TEAM T
JOIN MEMBER M ON T.TEAM_ID = M.TEAM_ID
```

### 객체의 2개의 양방향 관계 중 하나로 외래 키를 관리해야 한다.

![](https://raw.githubusercontent.com/oasis791/blog-posting/12063087704ed75e003a2c781c56d6964c4d5859/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/5.%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84%20%EB%A7%A4%ED%95%91%20%EA%B8%B0%EC%B4%88/2/1/4.png)

회원이 속한 팀이 변경되었을 때 Member의 `Team team`을 변경하여 외래키 `TEAM_ID`를 관리해야 할까? 아니면 Team의 `List members`를 변경하여 외래키 `TEAM_ID`를 관리해야 할까?
-> **규칙(주인)을 정하여, 둘 중 하나로 외래키를 관리해야 한다.**

## 연관관계의 주인(Owner)
- **양방향 매핑 규칙이다.**
- 객체의 두 관계 중 하나를 연관관계의 **주인**으로 지정한다.
- **연관관계의 주인만이 외래키를 관리(등록, 수정)할 수 있다.
- 주인은 `mappedBy` 속성을 사용하면 안된다.
- 주인이 아니면 `mappedBy` 속성으로 주인을 지정한다.
- **※ 주인이 아닌 쪽은 읽기만 가능하다. ※**
	- `mappedBy`가 있는 쪽에 데이터를 삽입하더라도, 변화가 없다.
	- 데이터를 삽입하고 싶다면, **반드시, 주인 쪽에 삽입해야한다.**

## 주인 지정하는 방법
- **외래키가 있는 곳을 주인으로 지정한다.**
- 예시에서는 `Member.team`이 연관관계의 주인이 된다.

![](https://raw.githubusercontent.com/oasis791/blog-posting/12063087704ed75e003a2c781c56d6964c4d5859/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/5.%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84%20%EB%A7%A4%ED%95%91%20%EA%B8%B0%EC%B4%88/2/1/5.png)

- 외래키가 있는 엔티티를 주인으로 지정하지 않고, 1쪽 엔티티를 주인으로 지정하여 외래키를 관리한다면, 값이 변경 되었을 때, 다른 테이블에 UPDATE 쿼리가 날아간다.
	- 굉장히 복잡해 질 위험이 있고, 성능적인 이슈가 발생할 수 있다.
	- **외래키가 존재하는 엔티티를 주인으로 설정해야, 직관적으로 테이블에 쿼리가 날아가는 것을 확인할 수 있다.**
- **`@ManyToOne`이 붙은 다(N) 쪽이 주인이 된다고 생각하면 편하다!**