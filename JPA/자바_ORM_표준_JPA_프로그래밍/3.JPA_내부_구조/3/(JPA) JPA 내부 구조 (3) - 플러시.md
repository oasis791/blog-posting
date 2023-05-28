![김영한-JPA](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/JPA%EB%A9%94%EC%9D%B8.png)

> 본 포스팅의 이미지 저작권은 [자바 ORM 표준 JPA 프로그래밍 - 기본편 (김영한)](https://www.inflearn.com/course/ORM-JPA-Basic) 강의에 있습니다.

## 플러시(Flush)
- 영속성 컨텍스트의 변경내용을 데이터베이스에 동기화하는 것
- 데이터베이스 트랜잭션이 `commit`되면 flush가 자동 발생

## 플러시 발생하면 어떤 일이?
1. [변경 감지 (Dirty Checking)](https://kimhyunwook.tistory.com/26)
2. 수정된 Entity를 쓰기 지연 SQL 저장소에 등록
3. 쓰기 지연 SQL 저장소의 쿼리를 DB에 전송 (SELECT, UPDATE, DELETE)

-> flush가 일어난다고, 데이터베이스 트랜잭션이 commit이 되는건 아니다. 트랜잭션 커밋 호출시, 커밋된다.

## 영속성 컨텍스트를 플러시하는 방법
다음 내용들을 직접 사용할 일은 거의 없지만, 테스트 등의 과정을 위해서 알아놔야 한다!
1. em.flush() - 직접 호출
```java
Member member = new Member("100L, "memberA");
em.persist(member);

em.flush();
System.out.println("================");
tx.commit();
```

```
Hibernate:
	/* insert hellojpa.Member
		*/ insert
		into
			Member
			(name, id)
		values
			(?, ?)
================
// tx.commit()에서 flush가 자동 호출 되기 전, SQL 쿼리들이 발생한다는 것을 알 수 있음
```
2. 트랜잭션 커밋 - 플러시 자동 호출
```java
Member member = new Member("100L, "memberA");
em.persist(member);
tx.commit();
```

```
Hibernate:
	/* insert hellojpa.Member
		*/ insert
		into
			Member
			(name, id)
		values
			(?, ?)
			
//tx.commit()으로 인해, flush()가 호출됨을 알 수 있다.
```

3. JPQL 쿼리 실행 - 플러시 자동 호출
```java
em.persist(memberA);
em.persist(memberB);
em.persist(memberC);
// 이 부분까지 실제로 데이터베이스 쿼리가 날아가지 않음

// 중간에 JPQL 실행
List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();
```
`memberA,B,C`를 `SELECT`할 때, 데이터베이스에 해당 데이터가 존재하지 않는다면 문제가 생길 수 있다.
따라서 JPA는 JPQL이 실행될 때 디폴트로 `flush`를 발생시키고, 쿼리문을 실행한다.

-> flush가 일어난다고, 1차 캐시의 내용이 지워지지 않는다. 쓰기 지연 SQL 저장소의 쿼리들이 데이터베이스에 반영되는 과정

## 플러시 모드 옵션
EntityManager의 `setFlushMode` 메소드를 이용하여, 플러시를 어떻게 발생시킬지 옵션을 지정할 수 있다.
```java
em.setFlushMode(플러시 모드 옵션);
```

- FlushModeType.AUTO
	- 커밋이나 쿼리를 실행할 때 플러시 (기본값)
- FlushModeType.COMMIT
	- 커밋할 때만 플러시
	- 의도치 않게 DB에 쿼리를 날리지 않게 하기 위함

## 결론
- 트랜잭션이라는 작업 단위가 중요하다.
- 커밋 직전에만 동기화 하면 된다.