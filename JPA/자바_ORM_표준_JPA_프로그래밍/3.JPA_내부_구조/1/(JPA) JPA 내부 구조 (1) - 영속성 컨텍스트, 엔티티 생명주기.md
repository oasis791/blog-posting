![김영한-JPA](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/JPA%EB%A9%94%EC%9D%B8.png)

> 본 포스팅의 이미지 저작권은 [자바 ORM 표준 JPA 프로그래밍 - 기본편 (김영한)](https://www.inflearn.com/course/ORM-JPA-Basic) 강의에 있습니다.

## EntityManagerFactory와 EntityManager
![emf와_em.png](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/3.JPA_%EB%82%B4%EB%B6%80_%EA%B5%AC%EC%A1%B0/1/emf%EC%99%80_em.png)
1. 고객이 서비스를 요청함
2. 고객이 요청 할 때마다, EntityManagerFactory는 EntityManager를 생성
3. EntityManager는 내부적으로 Database Connection을 사용해서 DB를 사용한다.

## 영속성 컨텍스트 (Persistence Context)
- **엔티티를 영구 저장하는 환경**
- 영속성 컨텍스트는 눈에 보이지 않는, **논리적인 개념**이다.
- 엔티티 매니저(EntityManager)를 통해 영속성 컨텍스트에 접근한다.

## 엔티티의 생명주기 (Entity Lifecycle)
![entity_생명주기.png](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/3.JPA_%EB%82%B4%EB%B6%80_%EA%B5%AC%EC%A1%B0/1/entity_%EC%83%9D%EB%AA%85%EC%A3%BC%EA%B8%B0.png)

### 비영속 (new/transient)
![비영속.png](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/3.JPA_%EB%82%B4%EB%B6%80_%EA%B5%AC%EC%A1%B0/1/%EB%B9%84%EC%98%81%EC%86%8D.png)
영속성 컨텍스트와 전혀 관계가 없는 **새로운** 상태
```java
// 비영속 상태
Member member = new Member();  
member.setId(101L);  
member.setName("HelloJPA");
```
new 키워드를 통해 새로운 객체를 생성한 상태가 이에 해당한다.

### 영속 (managed)
![영속.png](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/3.JPA_%EB%82%B4%EB%B6%80_%EA%B5%AC%EC%A1%B0/1/%EC%98%81%EC%86%8D.png)
영속성 컨텍스트에 **관리** 되는 상태
```java
// 영속 상태 (managed)  
em.persist(member);
```
EntityManager의 `persist` 메소드로 객체를 영속 상태로 전환한다.

**!!주의할 점!!**
`persist`를 한 시점에서 DB에 직접 저장되지는 않는다.
-> `persist`를 한 시점부터, 영속성 컨텍스트 내부의 **1차 캐시**에 객체가 저장된다.
	트랜잭션을 `commit`한 시점에 `insert` 쿼리로 DB에 직접 반영하게 된다.
	자세한 내용은 후술 할 [1차 캐시](https://kimhyunwook.tistory.com/26) 참조!

### 준영속 (detached)
영속성 컨텍스트에 저장되었다가 **분리**된 상태
```java
// 준영속 상태 (detached)
em.detach(member);
```
EntityManager의 `detach` 메소드로 객체를 준영속 상태로 전환한다.

### 삭제 (removed)
실제 DB에서 **삭제**를 요청하는 상태
```java
// 삭제 상태 (removed)
em.remove(member);
```
EntityManager의 `remove` 메소드로 객체를 삭제 상태로 전환한다.

