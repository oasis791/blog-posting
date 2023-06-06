![김영한-JPA](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/JPA%EB%A9%94%EC%9D%B8.png)

> 본 포스팅의 이미지 저작권은 [자바 ORM 표준 JPA 프로그래밍 - 기본편 (김영한)](https://www.inflearn.com/course/ORM-JPA-Basic) 강의에 있습니다.

## 엔티티 매핑 소개
1. 객체와 테이블 매핑 : `@Entity`, `@Table`
2. 필드와 컬럼 매핑 : `@Column`
3. 기본 키 매핑 : `@Id`
4. 연관관계 매핑: `@ManyToOne`, `@joinColumn`

## 객체와 테이블 매핑
### @Entity
```java
@Entity
public class Member {
	...
}
```
- `@Entity`가 붙은 클래스는 JPA가 관리하며, 엔티티라 한다.
- 파라미터가 없는 public 또는 protected인 **기본 생성자**가 필수이다.
	- JPA 스펙상 그렇다.
	- 동적인 작업이나, Reflection, Proxy하는 작업 등을 할 때 필요하다.
- final class, enum, interface, inner class에서는 사용 불가
- DB에 저장하고 싶은 필드에는 final을 사용할 수 없다.

### @Entity의 속성
```java
package javax.persistence;  
  
import ...
@Documented  
@Target(TYPE)  
@Retention(RUNTIME)  
public @interface Entity {  
	String name() default "";  
}
```
- `@Entity(name = "")` 속성으로 Entity의 이름을 지정할 수 있다.

### @Table
- `@Table`은 엔티티와 매핑할 테이블을 지정한다.

### @Table의 속성
```java

package javax.persistence;  
  
import ...  
  
@Target(TYPE)  
@Retention(RUNTIME)  
public @interface Table {  
  
	String name() default "";  
	
	String catalog() default "";  
	  
	String schema() default "";  
	  
	UniqueConstraint[] uniqueConstraints() default {};  
	  
	Index[] indexes() default {};  
}
```
- `@Table(name = "")` 속성으로 Table의 이름을 지정할 수 있다.
- `@Table(catalog = "")` 속성으로 데이터베이스 catalog를 매핑할 수 있다.
- `@Table(schema = "")` 속성으로 데이터베이스 schema를 매핑할 수 있다.
- `@Table(uniqueConstraints = "")` 속성으로 DDL 생성 시에 유니크 제약 조건을 생성할 수 있다.

### @Id
- 엔티티의 주키를 지정할 수 있다.
```java

package javax.persistence;  
  
import ...
  
@Target({METHOD, FIELD})  
@Retention(RUNTIME)  
public @interface Id {}
```

## 데이터베이스 스키마 자동 생성
- DDL을 **애플리케이션 실행 시점**에 자동 생성한다.
- 테이블보다 객체에 집중할 수 있다.
- Dialect를 통해, 데이터베이스에 맞는 적절한 DDL을 생성한다.
	- `<property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>`
	- <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>`<property name="hibernate.dialect" value="org.hibernate.dialect.Oracle12cDialect"/>`
- 이렇게 **생성된 DDL은 개발 장비에서만 사용해야 한다**

### 데이터베이스 스키마 자동 생성의 속성
- **hibernate.hbm2ddl.auto**
![데이터베이스 스키마 자동 생성 - 속성](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/4.%EC%97%94%ED%8B%B0%ED%8B%B0%20%EB%A7%A4%ED%95%91/1/%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4%20%EC%8A%A4%ED%82%A4%EB%A7%88%20%EC%9E%90%EB%8F%99%20%EC%83%9D%EC%84%B1%20-%20%EC%86%8D%EC%84%B1.png)

### 데이터베이스 스키마 자동 생성의 주의점
- **운영 장비에는 절대 create, create-drop, update를 사용하면 안된다.**
	- create와 create-drop은 자동으로 drop 하기 때문에 데이터가 다 날아간다.
	- update를 잘못 수행하면, 데이터베이스가 lock이 걸려 서비스가 중단될 수 있다.
- 개발 초기 단계는 create 또는 update
- 테스트 서버는 update 또는 validate
- 스테이징과 운영 서버는 validate 또는 none

## DDL 생성 기능
- DDL 생성 기능은 DDL을 자동 생성할 때만 사용되고, JPA의 실행 로직에는 영향을 주지 않는다.
