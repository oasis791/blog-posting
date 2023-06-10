![김영한-JPA](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/JPA%EB%A9%94%EC%9D%B8.png)

> 본 포스팅의 이미지 저작권은 [자바 ORM 표준 JPA 프로그래밍 - 기본편 (김영한)](https://www.inflearn.com/course/ORM-JPA-Basic) 강의에 있습니다.

## 매핑 애노테이션 정리
1. @Column : 컬럼 매핑
2. @Temporal : 날짜 타입 매핑
3. @Enumerated : enum 타입 매핑
4. @Lob : BLOB, CLOB 매핑
5. @Transient : 특정 필드를 컬럼에 매핑하지 않음

## @Column 속성
- `name` : 필드와 매핑할 테이블의 **컬럼 이름**을 지정할 수 있다.
- `insertable`, `updatable` : 기본값은 true이다. false 지정 시, 해당 컬럼을 수정했을때 **데이터베이스에 insert 또는 update를 하지 않는다.**
- `nullable` :  기본값은 true이다. false 지정 시, **DDL을 생성할 때 not null 제약 조건**이 걸린다.
- `unique` : 기본값은 false이다. true 지정 시, **DDL을 생성할 때 unique 제약 조건이 걸린다.** 
	- **잘 안쓴다.** -> 제약 조건 이름이 랜덤값으로 지정돼서 알아보기 힘들기 때문
	- Column에서 unique 속성을 사용하지 않고, `@Table(uniqueContraints = " ")`로 사용하면 제약 조건의 이름도 지정할 수 있다.
- `length` :  String type에서만 사용한다. 기본값은 255이며, DDL을 생성할 때 **문자 길이 제약 조건이 걸린다.**
- `columnDefinition` : **데이터베이스 컬럼 정보를 직접 줄 수 있다.**
	- `@Column(columnDefinition = "varchar(100) defalut 'EMPTY'")
- `precision`, `scale` : BigDecimal 타입이나, BigInteger 타입에서 사용된다.
	- precision은 **소수점을 포함한 전체 자릿수**를 지정한다. (기본값 19)
	- scale은 **소수의 자릿수**이다. (기본값 2)

## @Enumerated 속성
- `Enumerated.ORDINAL` (default)
	- **Enum의 순서**를 데이터베이스에 저장
- `Enumerated.String`
	- **Enum의 이름**을 데이터베이스에 저장

-> **ORDINAL은 절대 사용하지 않는다.**

### ORDINAL 사용

```java
public enum RoleType {  
	USER, ADMIN  
}
```

-> 현재 ENUM 클래스에는 USER (순서 0), ADMIN (순서 1)이 선언되어 있다.

```java
	Member member = new Member();  
	member.setId(1L);  
	member.setUsername("A");  
	member.setRoleType(RoleType.USER);
```

```sql
Hibernate: 
    create table Member (
       id bigint not null,
        age integer,
        createdDate timestamp,
        description clob,
        lastModifiedDate timestamp,
        roleType integer,
        name varchar(255),
        primary key (id)
    )
```

-> DDL이 생성될 때, roleType이 integer형으로 생성된다.

![roletype_ordinal](https://raw.githubusercontent.com/oasis791/blog-posting/3b4c9f37dcdbd1dbcad2fc73f72a553d1510cb09/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/4.%EC%97%94%ED%8B%B0%ED%8B%B0%20%EB%A7%A4%ED%95%91/2/1.png)

-> ROLETYPE에 ENUM 클래스의 순서가 저장이 된다.

```java
	Member member = new Member();  
	member.setId(2L);  
	member.setUsername("B");  
	member.setRoleType(RoleType.ADMIN);
```

![](https://raw.githubusercontent.com/oasis791/blog-posting/3b4c9f37dcdbd1dbcad2fc73f72a553d1510cb09/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/4.%EC%97%94%ED%8B%B0%ED%8B%B0%20%EB%A7%A4%ED%95%91/2/2.png)

-> 현재 ENUM 클래스에 ADMIN이 USER의 다음 순서로 지정이 되어 있기 때문에, ROLETYPE에 1이 저장됨을 알 수 있다.

### 요구 사항 변경 -> ENUM 클래스에 GUEST 추가

```java
public enum RoleType {  
	GUEST, USER, ADMIN  
}
```

-> USER와 ADMIN보다 앞 순서에 GUEST를 지정하였다.

```java
	Member member = new Member();  
	member.setId(3L);  
	member.setUsername("C");  
	member.setRoleType(RoleType.GUEST);
```

![](https://raw.githubusercontent.com/oasis791/blog-posting/3b4c9f37dcdbd1dbcad2fc73f72a553d1510cb09/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/4.%EC%97%94%ED%8B%B0%ED%8B%B0%20%EB%A7%A4%ED%95%91/2/3.png)

-> A 멤버와 C 멤버가 각각 USER, GUEST로 ROLE이 다르지만, 데이터베이스 상에선 같은 멤버로 저장되었다.

->  **운영상 굉장히 위험하고, 해결할 수 없는 버그가 초래될 수 있다!!!**

### EnumType.STRING

```java
@Enumerated(EnumType.STRING)  
private RoleType roleType;
```

```java
	Member member = new Member();  
	member.setId(1L);  
	member.setUsername("A");  
	member.setRoleType(RoleType.USER);
```

```sql
Hibernate: 
    create table Member (
       id bigint not null,
        age integer,
        createdDate timestamp,
        description clob,
        lastModifiedDate timestamp,
        roleType varchar(255),
        name varchar(255),
        primary key (id)
    )
```

-> roleType이 ORDINAL과 다르게 varchar 타입으로 DDL이 나가는 것을 확인할 수 있다.

![](https://raw.githubusercontent.com/oasis791/blog-posting/3b4c9f37dcdbd1dbcad2fc73f72a553d1510cb09/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/4.%EC%97%94%ED%8B%B0%ED%8B%B0%20%EB%A7%A4%ED%95%91/2/4.png)

-> ROLETYPE 컬럼에 ENUM의 문자열 그대로 저장됨을 확인할 수 있다.

### @Temporal
Java 8 이후로는 굳이 @Temporal을 사용하지 않고, 현재는 java.time 패키지의 LocalDate와 LocalDateTime 클래스를 이용한다.

- LocalDate : 연, 월, 일을 나타낼 수 있다.
- LocalDateTime : 연, 월, 일, 시간을 나타낼 수 있다.
 
```java
	@Temporal(TemporalType.TIMESTAMP)  
	private Date createdDate;  
	  
	@Temporal(TemporalType.TIMESTAMP)  
	private Date lastModifiedDate;  
	  
	private LocalDate testLocalDate;  
	  
	private LocalDateTime testLocalDateTime;
```

-> Date는 java.util 패키지의 클래스이고 LocalDate, LocalDateTime은 각각 java.time의 클래스이다.

```sql
Hibernate: 
    create table Member (
	    ...
        createdDate timestamp,
        lastModifiedDate timestamp,
        testLocalDate date,
        testLocalDateTime timestamp,
        ...
    )
```

-> LocalDate는 자동으로 date 타입으로, LocalDateTime은 자동으로 timestamp 타입으로 DDL이 나감을 확인할 수 있다.

## @Lob
- 데이터베이스의 BLOB, CLOB 타입과 매핑한다.
- `@Lob`에는 지정할 수 있는 속성이 없다.

```java
package javax.persistence;  
  
import java.lang.annotation.Target;  
import java.lang.annotation.Retention;  
import static java.lang.annotation.ElementType.FIELD;  
import static java.lang.annotation.ElementType.METHOD;  
import static java.lang.annotation.RetentionPolicy.RUNTIME;  

@Target({METHOD, FIELD})  
@Retention(RUNTIME)  
public @interface Lob {  
}
```

- 매핑하는 필드 타입이 문자면 CLOB 매핑, 나머지는 BLOB으로 매핑한다.
	- CLOB : String, char[], java.sql.CLOB
	- BLOB : byte[], java.sql.BLOB

### @Transient
- 필드 매핑을 하지 않는다.
- 따라서, 데이터베이스에 저장되지도 않고, 조회되지도 않는다.
- 주로 메모리상에서만 임시로 어떤 값을 보관하고 싶을 때 사용한다.


