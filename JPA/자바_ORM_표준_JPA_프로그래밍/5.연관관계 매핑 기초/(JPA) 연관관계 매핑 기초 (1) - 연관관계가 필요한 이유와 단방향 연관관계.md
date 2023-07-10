![김영한-JPA](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/JPA%EB%A9%94%EC%9D%B8.png)

> 본 포스팅의 이미지 저작권은 [자바 ORM 표준 JPA 프로그래밍 - 기본편 (김영한)](https://www.inflearn.com/course/ORM-JPA-Basic) 강의에 있습니다.

## 연관관계가 필요한 이유
### 예제 시나리오
- 회원과 팀이 있다.
- 회원은 하나의 팀에만 속할 수 있다.
- 회원과 팀은 다대일 관계이다.

### 문제점 - 객체를 테이블에 맞추어 모델링한다.

![](https://github.com/oasis791/blog-posting/blob/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/5.%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84%20%EB%A7%A4%ED%95%91%20%EA%B8%B0%EC%B4%88/1.png?raw=true)

객체는 따로 연관관계가 존재하지 않고, 테이블 연관관계를 따른다.

**Member.java**

```java
@Entity public class Member {
    @Id @GeneratedValue @Column(name = "MEMBER_ID")
    private Long id;
    @Column(name = "USERNAME")
    private String name;
    @Column(name = "TEAM_ID")
    private Long teamId;
    public Member() {}
    public Long getId() {
        return id;
    }
    public void setId(Long id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public Long getTeamId() {
        return teamId;
    }
    public void setTeamId(Long teamId) {
        this.teamId = teamId;
    }
}
```

**Team.java**

```java
@Entity public class Team {
    @Id @GeneratedValue @Column(name = "TEAM_ID")
    private Long id;
    private String name;
    public Team() {}
    public Long getId() {
        return id;
    }
    public void setId(Long id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

### 시나리오
멤버와 팀을 저장하고, 저장된 멤버가 소속되어 있는 팀을 조회한다.

```java
// 팀 저장
Team team = new Team();
team.setName("TeamA");
em.persist(team);
// 회원 저장
Member member = new Member();
member.setName("member1");
member.setTeamId(team.getId());
em.persist(member);
// 팀을 조회하기 위해서?
Member findMember = em.find(Member.class, member.getId());
Long findTeamId = findMember.getTeamId();
Team findTeam = em.find(Team.class, findTeamId);
tx.commit();
```

보다시피, 연관관계가 없기 때문에 객체간의 관계로 객체를 조회하는 것이 아닌, 테이블간의 관계로 객체를 조회하는 상황이 연출된다.

### 객체를 테이블에 맞추어 데이터 중심으로 모델링하면, 협력 관계를 만들 수 없다.
- **테이블은 외래 키로 조인**을 사용해서 연관된 테이블을 찾는다.
- **객체는 참조**를 이용하여 연관 된 객체를 찾는다.
- **테이블과 객체 사이에는 이러한 큰 간격이 있다.**

## 단방향 연관관계
### 객체지향 모델링

![](https://github.com/oasis791/blog-posting/blob/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/5.%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84%20%EB%A7%A4%ED%95%91%20%EA%B8%B0%EC%B4%88/2.png?raw=true)

앞서 보여준 테이블에 맞춘 모델링과 차이점은 Member 클래스에 `teamId`와 같은 외래키를 갖는 것이 아닌, `Team` 객체 자체를 갖고 있다.

```java
@Entity public class Member {

    @Id @GeneratedValue @Column(name = "MEMBER_ID")
    private Long id;
    
    @Column(name = "USERNAME")
    private String name;
    
    @ManyToOne @JoinColumn(name = "TEAM_ID")
    private Team team;
    
    public Member() {}
    public Long getId() {
        return id;
    }
    public void setId(Long id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public Team getTeam() {
        return team;
    }
    public void setTeam(Team team) {
        this.team = team;
    }
}
```

현재 Member와 Team이 N:1 관계이므로, `@ManyToOne` 애노테이션을 통해 연관관계를 매핑해준다.
조인 컬럼으로 어떠한 컬럼을 이용할 것인지를 지정하기 위해 `@JoinColumn` 애노테이션을 통해 TEAM_ID 컬럼을 조인 컬럼으로 지정한다.

![](https://github.com/oasis791/blog-posting/blob/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/5.%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84%20%EB%A7%A4%ED%95%91%20%EA%B8%B0%EC%B4%88/3.png?raw=true)

테이블에 맞춘 모델링과는 다르게, Team과 TEAM_ID(외래키)를 연관관계 매핑한다.

```java
// 팀 저장
Team team = new Team();
team.setName("TeamA");
em.persist(team);
// 회원 저장
Member member = new Member();
member.setTeam(team);
em.persist(member);
// 팀 조회
Member findMember = em.find(Member.class, member.getId());
Team findTeam = findMember.getTeam();
tx.commit();
```

조회 한 회원에서 `findMember.getTeam()`메소드로 속한 팀을 바로 끄집어 낼 수 있다.