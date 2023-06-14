![김영한-JPA](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/JPA%EB%A9%94%EC%9D%B8.png)

> 본 포스팅의 이미지 저작권은 [자바 ORM 표준 JPA 프로그래밍 - 기본편 (김영한)](https://www.inflearn.com/course/ORM-JPA-Basic) 강의에 있습니다.

## @Id (직접 할당)
- JPA 엔티티 객체의 식별자로 사용할 필드에 적용한다.
- 자동 할당 옵션을 사용하지 않는다면, 데이터베이스에 식별자를 직접 할당해야한다.

## @GeneratedValue (자동 할당)
`@GeneratedValue(strategy = GenerationType.***)` 키워드로 기본 키 자동 생성 전략을 선택할 수 있다.

### IDENTITY
- 주로 MySQL, PostgreSQL, SQL Server, DB2에서 사용한다.
	- 예시) MySQL의 AUTO_INCREMENT

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

```sql
// MySQL Database

Hibernate: 
    create table Member (
       id bigint not null auto_increment,
        name varchar(255),
        primary key (id)
    )
```

### IDENTITY 전략의 특징
- 기본 키 생성이 자바 코드 상에서 이루어지는 것이 아닌, **데이터베이스**에 위임한다.
- 보통 Entity가 영속 상태가 되면, 1차 캐시에서 기본키가 사용되는데, 기본키가 데이터베이스에서 생성되기 때문에 **INSERT 쿼리가 날아가기 전까지는 기본키를 알 수 없다.**

![1차캐시.png](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/3.JPA_%EB%82%B4%EB%B6%80_%EA%B5%AC%EC%A1%B0/2/1%EC%B0%A8%EC%BA%90%EC%8B%9C.png)

- 따라서, IDENTITY 전략은 `em.persist()` 시점에 **즉시 INSERT 쿼리가 실행**되고, DB에서 식별자를 조회한다.

```java
	Member member = new Member();  
	member.setUsername("A");  
	  
	System.out.println("===================");  
	em.persist(member);  
	System.out.println("===================");

	tx.commit();
```

```sql
===================
Hibernate: 
    /* insert hellojpa.Member
        */ insert 
        into
            Member
            (id, name) 
        values
            (null, ?)
===================
```

- 이러한 동작이후에, JPA가 데이터베이스에서 PK를 가져와 사용하게 된다.
-> **쓰기 지연은 IDENTITY 전략에서는 불가능하다.**

### SEQUENCE
- 데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트이다.
	- 주로 Oracle, PostgreSQL, DB2, H2 데이터베이스에서 사용한다.

```java
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE)
private Long id;
```

```sql
// H2 Database

Hibernate: 
    drop sequence if exists hibernate_sequence
Hibernate: 
	create sequence hibernate_sequence start with 1 increment by 1
Hibernate: 
    create table Member (
       id bigint not null,
        name varchar(255),
        primary key (id)
    )
```

Sequence 오브젝트를 따로 생성하지 않으면 Hibernate에서는 Hibernate Sequence 오브젝트를 이용하여 자동 할당을 한다.

Sequence 오브젝트를 생성하고 싶다면 `@SequenceGenerator` 애노테이션을 사용하면 된다.

```java
@Entity  
@SequenceGenerator(  
		name = "MEMBER_SEQ_GENERATOR",  
		sequenceName = "MEMBER_SEQ",  
		initialValue = 1, allocationSize = 1  
)  
public class Member {  
  
	@Id  
	@GeneratedValue(strategy = GenerationType.SEQUENCE,  
			generator = "MEMBER_SEQ_GENERATOR")  
	private Long id;
	
	...
}
```

클래스단에서 `@SequenceGenerator` 애노테이션을 통해 Sequence 오브젝트를 생성할 수 있다. 

```sql
Hibernate: 
    drop sequence if exists MEMBER_SEQ
Hibernate: 
	create sequence MEMBER_SEQ start with 1 increment by 1
Hibernate: 
    create table Member (
       id bigint not null,
        name varchar(255),
        primary key (id)
    )
```

`@SequenceGenerator`의 sequenceName 속성으로 Sequence 오브젝트가 생성된 것을 확인할 수 있다.

### SEQUENCE 전략의 특징
- IDENTITY 전략과 마찬가지로 **기본키 생성을 데이터베이스의 SEQUENCE에 위임한다.**
- 따라서, JPA가 영속성 컨텍스트의 1차 캐시에서 PK를 매핑할때, **데이터베이스의 SEQUENCE를 조회한다.**

```java
	Member member = new Member();  
	member.setUsername("A");  
	  
	System.out.println("===================");  
	em.persist(member);  
	System.out.println("===================");  
	  
	tx.commit();
```

```sql
===================
Hibernate: 
    call next value for MEMBER_SEQ
===================
Hibernate: 
    /* insert hellojpa.Member
        */ insert 
        into
            Member
            (name, id) 
        values
            (?, ?)
```

- IDENTITY 전략과는 다르게, **트랜잭션 commit 시점에 INSERT 쿼리가 실행된다.**
-> 따라서, **쓰기 지연이 가능하다.**

그러나, 영속 상태가 될 때마다 네트워크를 왔다갔다 해야 하므로, 오버헤드가 발생하진 않을까?
-> allocationSize 옵션으로, 성능 최적화가 가능하다.

```java
	@Entity  
	@SequenceGenerator(  
	name = "MEMBER_SEQ_GENERATOR",  
	sequenceName = "MEMBER_SEQ",  
	initialValue = 1, allocationSize = 50  
	)  
	public class Member {}
```

이와 같이, allocationSize 속성을 50으로 설정하면,  sequence value가 50씩 증가하게 된다.
미리 50개의 value를 잡아두고, 엔티티가 영속 상태가 될 때마다 하나씩 증가시켜준다면, sequence를 참조하는 오버헤드가 해결된다.

```java
	Member member1 = new Member();  
	member1.setUsername("A");  
	Member member2 = new Member();  
	member2.setUsername("B");  
	Member member3 = new Member();  
	member3.setUsername("C");  
	  
	System.out.println("===================");  
	em.persist(member1);  
	em.persist(member2);  
	em.persist(member3);  
	System.out.println("===================");  
	  
	tx.commit();
```

```sql
Hibernate: 
    
    drop table Member if exists
Hibernate: 
    
    drop sequence if exists MEMBER_SEQ
Hibernate: create sequence MEMBER_SEQ start with 1 increment by 50
Hibernate: 
    
    create table Member (
       id bigint not null,
        name varchar(255),
        primary key (id)
    )
    
===================
Hibernate: 
    call next value for MEMBER_SEQ
Hibernate: 
    call next value for MEMBER_SEQ
===================
Hibernate: 
    /* insert hellojpa.Member
        */ insert 
        into
            Member
            (name, id) 
        values
            (?, ?)
Hibernate: 
    /* insert hellojpa.Member
        */ insert 
        into
            Member
            (name, id) 
        values
            (?, ?)
Hibernate: 
    /* insert hellojpa.Member
        */ insert 
        into
            Member
            (name, id) 
        values
            (?, ?)
```

call next value가 2번 호출 되는 것을 알 수 있는데, value를 1로 initialize 해주기 위해 한번 호출되고, 51까지의 value를 확보하기 위해 한번 호출되는 것이다.

![](https://raw.githubusercontent.com/oasis791/blog-posting/c7343f209a96fb56c12a4c6c2adeddfe329b4186/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/4.%EC%97%94%ED%8B%B0%ED%8B%B0%20%EB%A7%A4%ED%95%91/3/1.png)

key값이 정상적으로 저장되는 것을 알 수 있다.

### TABLE
- **키 생성 전용 테이블**을 하나 만들어서 데이터베이스 시퀀스를 흉내내는 전략
- 장점 : 모든 데이터베이스에 적용이 가능하다.
- 단점 : 성능이 좋지 않다.

`@SequenceGenerator`과 비슷하게, `@TableGenerator` 애노테이션을 이용하여 키 생성 전용 테이블을 만들어줄 수 있다.

```java
@Entity  
@TableGenerator(  
		name = "MEMBER_SEQ_GENERATOR",  
		table = "MY_SEQUENCES",  
		pkColumnValue = "MEMBER_SEQ", allocationSize = 1  
)  
public class Member {  
  
	@Id  
	@GeneratedValue(strategy = GenerationType.TABLE,  
			generator = "MEMBER_SEQ_GENERATOR")  
	private Long id;
	
	...
}
```

이와 같이 **키 생성 전용 테이블**을 만들고 나서 애플리케이션을 실행 시켜본다면

```sql
Hibernate: 
    drop table Member if exists
Hibernate: 
    drop table MY_SEQUENCES if exists
Hibernate: 
    create table Member (
       id bigint not null,
        name varchar(255),
        primary key (id)
    )
Hibernate: 
    create table MY_SEQUENCES (
       sequence_name varchar(255) not null,
        next_val bigint,
        primary key (sequence_name)
    )
Hibernate: 
    insert into MY_SEQUENCES(sequence_name, next_val) values ('MEMBER_SEQ',0)
```

`@TableGenerator`에서 선언해준대로, `MY_SEQUENCES`라는 테이블이 생성되었고, sequence_name 컬럼의 `MEMBER_SEQ`값이 들어가게 되며, next_val 값이 0으로 초기화 된다.

![](https://raw.githubusercontent.com/oasis791/blog-posting/c7343f209a96fb56c12a4c6c2adeddfe329b4186/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/4.%EC%97%94%ED%8B%B0%ED%8B%B0%20%EB%A7%A4%ED%95%91/3/2.png)

실제로 MY_SEQUENCES 테이블에 MEMBER_SEQ와 자동 할당 값이 잘 들어가있는 것을 확인할 수 있다.

### TABLE 전략의 특징
- `@SequenceGenerator`의 속성과 같이 allocationSize를 설정할 수 있다.

### AUTO
- 전략을 따로 기입하지 않으면 default 값이다. 
- DB Dialect에 따라 자동으로 전략을 선택해준다.

## 권장하는 식별자 전략
- **기본 키 제약 조건** : Not Null, 유일성, **변하면 안된다.**
	- Not Null 조건과 유일성 조건은 만족하기 쉽지만, 변하면 안된다는 조건을 만족시키기 어렵다.
		-> 10년, 20년 후에도 이 조건을 만족시키는 **자연키**(비즈니스 적으로 의미 있는 주민등록번호, 전화번호 등)는 찾기 어렵다. -> **대체키**를 사용하자.
- **김영한 선생님의 권장 전략** : **Long형 + 대체키 + 키 생성 전략**을 적절히 혼합하여 사용한다.
	- Auto_Increment
	- Sequence
	- UUID, 랜덤값 조합 등등
	- **절대 비즈니스를 키로 끌고 오지 않는다!!**
