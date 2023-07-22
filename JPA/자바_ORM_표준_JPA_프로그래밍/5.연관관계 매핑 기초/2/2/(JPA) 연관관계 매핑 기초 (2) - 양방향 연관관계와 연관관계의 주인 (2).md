![김영한-JPA](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/JPA%EB%A9%94%EC%9D%B8.png)

> 본 포스팅의 이미지 저작권은 [자바 ORM 표준 JPA 프로그래밍 - 기본편 (김영한)](https://www.inflearn.com/course/ORM-JPA-Basic) 강의에 있습니다.

## 양방향 매핑시 가장 많이 하는 실수
- 연관관계의 주인에 값을 입력하지 않음

### 잘못된 코드

```java
// 회원 저장
	Member member = new Member();
	member.setName("member1");
	em.persist(member);

// 팀 저장
	Team team = new Team();
	team.setName("TeamA");
	team.getMembers().add(member);
	
	em.persist(team);
	
	em.flush();
	em.clear();
	
	tx.commit();
```

![](https://raw.githubusercontent.com/oasis791/blog-posting/c27a3dbe42022583e381a18bc170201a94528e86/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/5.%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84%20%EB%A7%A4%ED%95%91%20%EA%B8%B0%EC%B4%88/2/2/1.png)

주인이 아닌 방향(`Team의 List<Member> members`)에만 연관관계(`team.getMembers().add(memeber)`를 설정하면, 멤버 테이블의 외래키에는 null 값이 들어가있는 것을 확인할 수 있다.

-> **반드시 연관관계의 주인에 값을 입력해야 한다.**

### 올바른 코드

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
	
	tx.commit();
```

![](https://raw.githubusercontent.com/oasis791/blog-posting/c27a3dbe42022583e381a18bc170201a94528e86/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/5.%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84%20%EB%A7%A4%ED%95%91%20%EA%B8%B0%EC%B4%88/2/2/2.png)

주인 방향 (`Member의 Team`)에 연관관계(`member.setTeam(team)`)을 설정하면, 정상적으로 멤버 테이블에 외래키가 저장되는 것을 확인할 수 있다.

## 순수한 객체 관계를 고려하면, 항상 양쪽 다 값을 입력하라

### 주인 방향에만 연관관계를 설정해주었을 때

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
	
// 팀 조회 및 팀에 속한 멤버 조회
	Team findTeam = em.find(Team.class, team.getId());
	List <Member> members = findTeam.getMembers();
	
	System.out.println("=====================");
	for (Member m : members) {
	    System.out.println("m = " + m.getName());
	}
	System.out.println("=====================");
	
	tx.commit();
```

현재 `member.setTeam(team)`을 통해 주인 방향에만 연관관계를 설정해주었다. `em.flush()`를 이용하여 SQL을 데이터베이스에 보냈고, `em.clear()`를 통해 1차 캐시를 모두 비워주었다. `em.find()` 메소드를 통해 데이터베이스에서 저장한 팀을 조회했고, 해당 팀에 속한 멤버를 조회하였다.

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
        team0_.TEAM_ID as TEAM_ID1_1_0_,
        team0_.name as name2_1_0_ 
    from
        Team team0_ 
    where
        team0_.TEAM_ID=?
=====================
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
=====================
```

멤버 이름을 조회하기 위해 select 쿼리가 실행되고, 그에 따라 `m = member1` 이라는 결과 값이 출력되는 것을 알 수 있다.

앞서 설명한 시나리오에서 본 것 처럼, 주인 방향에만 연관관계를 설정하더라도 큰 문제는 없어보인다. 그러나, **양방향으로 객체의 연관관계를 설정해주지 않으면 다음 2가지 문제가 발생할 수 있다.**

### 문제점 (1) - flush가 일어나지 않고, 1차 캐시를 clear 해주지 않는다면?

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
	
	//em.flush();
	//em.clear();
	
// 팀 조회 및 팀에 속한 멤버 조회
	Team findTeam = em.find(Team.class, team.getId());
	List <Member> members = findTeam.getMembers();
	
	System.out.println("=====================");
	for (Member m : members) {
	    System.out.println("m = " + m.getName());
	}
	System.out.println("=====================");
	
	tx.commit();
```

`em.flush()`와 `em.clear()`를 주석처리 해주었다. 따라서, **현재 생성 된 `team`과 `member`는 데이터베이스에 반영되지 않고, 1차캐시에 존재한다.** 
`em.find()` 메소드는 **1차 캐시에 생성된 순수한 team 객체를 `findTeam`에 저장하기 때문에** 생성 된 팀에 속한 멤버를 의도대로 불러올 수 없다.

```sql
Hibernate: 
    call next value for hibernate_sequence
Hibernate: 
    call next value for hibernate_sequence
=====================
=====================
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
```

### 문제점 (2) - 테스트 케이스 작성 시

테스트를 작성할 때는 JPA를 이용하지 않고도 **순수한 자바 객체를 이용**하여 테스트 하는 경우가 많은데, 양방향 연관관계를 설정해주지 않으면 한쪽 방향에는 null이 나온다.

### 해결 방법 - 양방향 연관관계는 객체의 양쪽 연관관계를 모두 설정한다.

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

	team.getMembers().add(member);
	//em.flush();
	//em.clear();
	
// 팀 조회 및 팀에 속한 멤버 조회
	Team findTeam = em.find(Team.class, team.getId());
	List <Member> members = findTeam.getMembers();
	
	System.out.println("=====================");
	for (Member m : members) {
	    System.out.println("m = " + m.getName());
	}
	System.out.println("=====================");
	
	tx.commit();
```

마찬가지로, `em.flush()`와 `em.clear()`를 주석처리 한 상황에서, `team.getMembers().add(member)`를 통해 객체의 양방향 연관관계를 설정해주었다.

```sql
Hibernate: 
    call next value for hibernate_sequence
Hibernate: 
    call next value for hibernate_sequence
=====================
m = member1
=====================
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
```

순수한 자바 객체에 멤버가 추가되었으므로, 의도대로 멤버 이름이 출력되는 것을 확인할 수 있다.

**이처럼 순수한 객체 관계를 고려해서 항상 양쪽에 값을 설정하자!!**

## 양방향 연관관계를 설정할 때 편의 메소드를 생성하자

```java
// 팀 저장
	Team team = new Team();
	team.setName("TeamA");
	em.persist(team);
	
// 회원 저장
	Member member = new Member();
	member.setName("member1");
	member.setTeam(team); // 1
	em.persist(member);

	team.getMembers().add(member); // 2
```

코드를 사람이 작성하다보니, 양방향 연관관계를 맺는 주석 1번과 2번 중 하나가 누락될 수 있다. 이러한 상황을 방지하고자 **편의 메소드**를 생성하여 사용하도록 하자.

```java
@Entity 
public class Member {
    @Id @GeneratedValue 
    @Column(name = "MEMBER_ID")
    private Long id;
    
    @Column(name = "USERNAME")
    private String name;
    
    @ManyToOne @JoinColumn(name = "TEAM_ID")
    private Team team;

	// 생성자 및 Getter Setter ...
    
    public void changeTeam(Team team) {
        this.team = team;
        team.getMembers().add(this);
    }
}
```

주인 연관관계를 갖고 있는 `Member`에서 `changeTeam` 메소드를 편의 메소드로 사용할 수 있다.
`changeTeam` 메소드가 호출 될 때, 해당 멤버는 파라미터로 넘어온 team과 연관관계를 맺게 되고, `team.getMembers().add(this)` 를 통해 반대 방향의 연관관계도 자동으로 맺게 된다.

```java
@Entity 
public class Team {
    @Id @GeneratedValue 
    @Column(name = "TEAM_ID")
    private Long id;
    
    private String name;
    
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
    
    public void addMember(Member member) {
        member.setTeam(this);
        members.add(member);
    }

	// 생성자 및 Getter Setter ...
}
```

주인 연관관계를 갖고 있지 않은 반대 방향에서도 `addMember()` 메소드를 편의 메소드로 이용할 수 있다.
`addMemeber` 메소드가 호출 될 때, 파라미터로 넘어온 member에 현재 team을 세팅해주고, 현재 팀의 속한 멤버를 `members.add(member)` 메소드를 통해 추가해준다.

**-> 양방향에 편의 메소드가 있으면, 무한루프와 같은 문제가 발생할 수 있으므로 편의 메소드는 한쪽에서만 사용하자.**

**※ `setTeam()`또는 `setMember()`이 아닌, `changeTeam()`, `addMember()`로 사용하는 이유**
- `set***()`은 Java의 관례에 따라, 단순히 값을 세팅해줄 때 사용한다.
- **어떠한 로직이 들어가면 혼란을 방지하기 위해 다른 이름의 메소드로 작명해준다.**

## 양방향 매핑시에 무한 루프를 조심하자
- toString(), lombok, JSON 생성 라이브러리 등에서 문제가 발생할 수 있다.

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

	// 생성자 및 Getter Setter ...
	
    @Override 
    public String toString() {
        return "Member{" + 
		        "id=" + id + 
		        ", name='" + name + '\'' + 
		        ", team=" + team + 
		        '}';
    }
}
```

```java
@Entity 
public class Team {
    @Id @GeneratedValue 
    @Column(name = "TEAM_ID")
    private Long id;
    
    private String name;
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();

	// 생성자 및 Getter, Setter ...
	
    @Override 
    public String toString() {
        return "Team{" + 
		        "id=" + id + 
		        ", name='" + name + '\'' + 
		        ", members=" + members + 
		        '}';
    }
}
```

양방향에 `toString()` 메소드를 생성하면, `Member`의 `toString()`에서  `Team`의 `toString()`이 호출되고, 반대로 `Team`의 `toString()`에서 `Member`의 `toString()`이 호출되는 무한루프에 빠지게 된다.

마찬가지로, lombok에서 생성하는 `toString()`이나, 엔티티를 JSON 문자열로 변환될 때 무한루프에 빠질 위험성이 있다.

### 해결 방법
- `toString()`을 웬만하면 생성하지 않고, lombok에서도 `toString()`을 웬만하면 생성하지 않는다.
- JSON 생성 라이브러리의 무한루프를 방지하기 위해 **Controller에는 Entity를 절대 반환하지 않고, DTO를 통해 전달하여, JSON을 생성하도록 한다.**
	- Controller에 Entity를 직접 반환하면, 무한루프가 생길 수 있고, Entity의 스펙이 변경될 수 있다.

## 양방향 매핑 정리
- **단방향 매핑만으로도 이미 연관관계 매핑은 완료됐다.**
	- JPA 모델링 단계에서, 단방향 매핑으로 설계를 완료해야한다.
- 양방향 매핑은 반대 방향으로 조회(객체 그래프 탐색) 기능이 추가 된 것 뿐이다.
- 개발을 진행하다보면 JPQL에서 역방향으로 탐색할 일이 많이 생긴다.
- **단방향 매핑을 잘 해놓으면 양방향은 필요할 때 추가하면 된다. (테이블에는 영향을 주지 않기 때문)**

**-> JPA 모델링 단계에서 모든 객체를 단방향 매핑으로 설계를 완료한 뒤, 필요할 때 양방향 매핑을 추가하자!**

### 연관관계 주인을 정하는 기준
- 비즈니스 로직을 기준으로 연관관계 주인을 선택하면 안된다.
- **연관관계 주인은 외래키의 위치를 기준으로 선택해야 한다!!**