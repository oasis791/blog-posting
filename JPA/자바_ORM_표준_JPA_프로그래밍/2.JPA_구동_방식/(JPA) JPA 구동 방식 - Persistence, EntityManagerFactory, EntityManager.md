![김영한-JPA](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/JPA%EB%A9%94%EC%9D%B8.png)

> 본 포스팅의 이미지 저작권은 [자바 ORM 표준 JPA 프로그래밍 - 기본편 (김영한)](https://www.inflearn.com/course/ORM-JPA-Basic) 강의에 있습니다.

# JPA 구동 방식
![김영한-JPA구동](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/2.JPA_%EA%B5%AC%EB%8F%99_%EB%B0%A9%EC%8B%9D/Persistence%2CEntityManagerFactory%2CEntityManager.png)

## 설정 정보 조회
설정 정보를 조회하는 방법은, JPA 구현체의 따라 달라질 수 있다. 베이직한 Hibernate 구현체와, 많이 사용되고 있는 Spring Data JPA 구현체를 알아보자.

### Hibernate 구현체
[Hibernate 공식 문서의 4.1 항목](https://docs.jboss.org/hibernate/orm/5.6/quickstart/html_single/#hibernate-gsg-tutorial-jpa-config)에 의하면, `META-INF/persistence.xml` 파일로 부트스트랩 프로세스를 정의한다.
```xml
<persistence xmlns="http://java.sun.com/xml/ns/persistence"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/persistence http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd"
        version="2.0">
    <persistence-unit name="org.hibernate.tutorial.jpa">
        ...
    </persistence-unit>
</persistence>
```

### Spring Data JPA 구현체
`application.properties` 또는 `application.yml` 파일에서 JPA 설정을 진행할 수 있다.

## Persistence 클래스
[Object DB - javax.persistence.Persistence](https://www.objectdb.com/api/java/jpa/Persistence)에서 `Persistence` 클래스의 대한 정보에 대해 알 수 있었다.
- `EntityManagerFactory`를 획득하기 위한 **부트스트랩** 클래스이다.
- `persistence.xml`의 설정 정보를 읽어와서 EntityManagerFactory 클래스 생성

```java
// javax.persistence.Persistence.java
public static EntityManagerFactory createEntityManagerFactory(String persistenceUnitName) {  
	return createEntityManagerFactory(persistenceUnitName, (Map)null);  
}
```
- `createEntityManagerFactory` 메소드로 EntityManagerFactory 타입을 반환해줌으로써 EntityManagerFactory 클래스를 생성한다.
- 파라미터로 `persistenceUnitName`을 받는데, `persistence.xml`의 설정 정보를 통해 UnitName을 가져온다.

## EntityManagerFactory
- EntityManager를 생성한다.
- EntityManagerFactory는 하나만 생성해서 애플리케이션 전체에서 공유한다.
```java
// javax.persistence.EntityManagerFactory.java
public interface EntityManagerFactory { 
	public EntityManager createEntityManager();
	...
}
```
- `createEntityManager` 메소드를 통해, EntityManager를 생성한다.

## EntityManager
- Entity를 관리하는 역할을 수행하는 클래스이다.
- 영속성 컨텍스트(Persistence Context)를 통해 Entity를 관리한다.
- 쓰레드 간에 공유하지 않는다.