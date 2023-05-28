![김영한-JPA](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/JPA%EB%A9%94%EC%9D%B8.png)

> 본 포스팅의 이미지 저작권은 [자바 ORM 표준 JPA 프로그래밍 - 기본편 (김영한)](https://www.inflearn.com/course/ORM-JPA-Basic) 강의에 있습니다.

## 영속성 컨텍스트의 이점
1. 1차 캐시
2. 영속 엔티티의 동일성(identity) 보장
3. 트랜잭션을 지원하는 쓰기 지연 (transactional write-behind)
4. 변경 감지(Dirty Checking)
5. 지연 로딩(Lazy Loading)

## 엔티티 조회
### 1차 캐시
![1차캐시.png](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/3.JPA_%EB%82%B4%EB%B6%80_%EA%B5%AC%EC%A1%B0/2/1%EC%B0%A8%EC%BA%90%EC%8B%9C.png)
다음 시나리오에 따라, 비영속 객체가 영속성 컨텍스트의 1차 캐시에 어떻게 저장되는지 알아보자.
```java
// 새로운 멤버 객체 생성
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");

// 1차 캐시에 저장됨
em.persist(member);

// 1차 캐시에서 조회
Member findMember = em.find(Member.class, "member1");
```
1. `new Member()`를 통해 비영속 객체인 `member`를 생성한다.
2.  `em.persist()`로 영속 상태로 전환되며, 영속성 컨텍스트 내부의 1차 캐시에 **DB PK로 매핑한 ID**가 **@Id**로, **Entity 객체 자체**가 **값**이 된다.
3. `em.find()`를 통해 Entity를 조회한다.
	1. JPA는 **1차 캐시**를 통해 해당 Entity를 찾는다.
	2. 1차 캐시에 해당 Entity가 존재하지 않는다면
		1. DB에서 조회한다.
		2. 찾은 Entity를 1차 캐시에 저장한다.
		3. 해당 Entity를 반환한다.

-> **큰 이점은 없다.**
-> EntityManager는 Database Transaction 단위로 만들고, **Database Transaction이 끝날 때, 같이 종료 시켜버린다.**
-> 고객의 요청이 하나 들어와서 비즈니스가 끝나버리면, **영속성 컨텍스트를 지워버리기 때문**에 찰나의 순간에서만 이득이고, 전체적인 구조에서는 큰 성능적 이득이 없다.

### 영속 엔티티의 동일성(identity) 보장
```java
Member a = em.find(Member.class, "member1");
Member b = em.find(Member.class, "member1");

System.out.println("result = ", (a == b)); // result = true
```
동일한 Entity를 조회하면, 같은 주소값을 보장하기 때문에 `==` 비교시, true가 반환 됨을 확인할 수 있다.
-> **1차 캐시**에 Entity가 존재하기 때문에 가능
-> 1차 캐시로 **반복 가능한 읽기(REPEATABLE READ)** 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공

## 엔티티 등록
### 트랜잭션을 지원하는 쓰기 지연 (transactional write-behind)
```java
// EntityManagerFactory에서, EntityManager 생성
EntityManager em = emf.createEntityManager();
// 데이터 변경을 위해 EntityManager에서 트랜잭션 시작
EntityTransaction tx = em.getTransaction();
tx.begin();

// memberA와 memberB 영속 상태 변경
em.persist(memberA);
em.persist(memberB);
// 그러나, 여기서 INSERT SQL을 데이터베이스에 보내지 않는다.

//트랜잭션 커밋
tx.commit();
// 커밋하는 순간 데이터베이스에 INSERT SQL을 보낸다.
```

![쓰기 지연 저장소](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/3.JPA_%EB%82%B4%EB%B6%80_%EA%B5%AC%EC%A1%B0/2/%EC%93%B0%EA%B8%B0%20%EC%A7%80%EC%97%B0%20%EC%A0%80%EC%9E%A5%EC%86%8C.png)
1. `em.persist()`를 통해 객체를 영속 상태로 전환
2. 1차 캐시에 해당 객체가 key-value 값으로 저장된다.
3. JPA가 해당 Entity를 분석하여 **INSERT** 쿼리를 생성한다.
4. **쓰기 지연 SQL 저장소**에 **INSERT 쿼리**들이 쌓인다.

![트랜잭션 커밋](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/3.JPA_%EB%82%B4%EB%B6%80_%EA%B5%AC%EC%A1%B0/2/%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98%20%EC%BB%A4%EB%B0%8B.png)
1. `tx.commit()`을 통해 트랜잭션 커밋 수행
2. **쓰기 지연 SQL 저장소**에 있던 쿼리들이 `flush`가 되면서 Database로 날아간다.
3. 실제 데이터베이스가 commit 된다.

*hibernate에선 `persistence.xml` 파일의 `batch_size` 옵션으로 해당 사이즈만큼 쿼리를 모아서 데이터베이스에 반영할 수 있다.*

## 엔티티 수정
### 변경 감지(Dirty Checking)
```java
Member member = em.find(Member.class, "memberA");

member.setName("ZZZZZ");
// 추가적인 persist() 호출을 해주지 않아도 된다.
tx.commit();
```

![변경 감지](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/3.JPA_%EB%82%B4%EB%B6%80_%EA%B5%AC%EC%A1%B0/2/%EB%B3%80%EA%B2%BD%EA%B0%90%EC%A7%80.png)
1. 트랜잭션 `commit`시, 내부적으로 `flush`가 발생한다.
2. Entity와 **스냅샷**을 비교한다.
	- **스냅샷** : 값을 읽어온 **최초 시점**의 상태를 저장해둔다.
3. Entity의 변경 감지시, `UPDATE` 쿼리를 **쓰기 지연 SQL 저장소**에 저장한다.
4. `UPDATE` 쿼리를 DB에 반영한 후, 커밋한다.

## 엔티티 삭제
```java
Member memberA = em.find(Member.class, "memberA");

em.remove(memberA); // 엔티티 삭제
```
앞서 언급한 매커니즘과 동일하다. `DELETE` 쿼리가 `commit` 시점에 나간다.

