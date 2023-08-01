![김영한-JPA](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/JPA%EB%A9%94%EC%9D%B8.png)

> 본 포스팅의 이미지 저작권은 [자바 ORM 표준 JPA 프로그래밍 - 기본편 (김영한)](https://www.inflearn.com/course/ORM-JPA-Basic) 강의에 있습니다.

## 다대다 (N:M)
- 관계형 데이터베이스는 정규화된 테이블 2개로 다대다(N:M) 관계를 표현할 수 없다.
- **연결 테이블**을 추가해서 일대다, 다대일 관계로 풀어내야 한다.

![](https://raw.githubusercontent.com/oasis791/blog-posting/4637418bebb49ec00bc9cea1eeb7de3cd246ddd8/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/6.%EB%8B%A4%EC%96%91%ED%95%9C%20%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84%20%EB%A7%A4%ED%95%91/4.%EB%8B%A4%EB%8C%80%EB%8B%A4%20%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84/1.png)

- **객체는 컬렉션을 사용해서 객체 2개로 다대다 관계를 표현할 수 있다.**

![](https://raw.githubusercontent.com/oasis791/blog-posting/4637418bebb49ec00bc9cea1eeb7de3cd246ddd8/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/6.%EB%8B%A4%EC%96%91%ED%95%9C%20%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84%20%EB%A7%A4%ED%95%91/4.%EB%8B%A4%EB%8C%80%EB%8B%A4%20%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84/2.png)

- **`@ManyToMany`**를 사용한다.
- **`@JoinTable`**로 연결 테이블을 지정할 수 있다.
- 다대다 매핑은 단방향, 양방향 모두 가능하다.

## 다대다 매핑 예시

```java
@Entity 
public class Member {

	// id, name, team 속성
	
    @ManyToMany
    @JoinTable(name = "MEMBER_PRODUCT")
    private List<Product> products = new ArrayList<>();

	// 생성자 및 Getter, Setter
}
```

`@ManyToMany` 애노테이션을 통해 Product 엔티티와 다대다 연관관계를 맺어주었고, 두 테이블의 조인 테이블을 `MEMBER_PRODUCT`로 지정하였다.

```sql
Hibernate: 
    
    create table MEMBER_PRODUCT (
       Member_MEMBER_ID bigint not null,
        products_PRODUCT_ID bigint not null
    )
```

두 테이블의 주키를 갖는 관계 테이블이 생성되는 것을 확인할 수 있다.

```sql
Hibernate: 
    
    alter table MEMBER_PRODUCT 
       add constraint FKfmfxdrleengm9fi0691plhcwa 
       foreign key (products_PRODUCT_ID) 
       references Product
Hibernate: 
    
    alter table MEMBER_PRODUCT 
       add constraint FK4ibylolqmostllrjdc147aowv 
       foreign key (Member_MEMBER_ID) 
       references Member
```

MEMBER_PRODUCT 테이블은 MEMBER와 PRODUCT의 주키를 외래키로 갖게 된다.

```java
@Entity  
public class Product {  
  
	@Id  
	@GeneratedValue  
	@Column(name = "PRODUCT_ID")  
	private Long id;  
	  
	private String name;  
	  
	@ManyToMany(mappedBy = "products")  
	private List<Member> members = new ArrayList<>();

	// 생성자 및 Getter, Setter
}
```

양방향 관계를 맺고 싶다면, 반대 방향 엔티티에서 `mappedBy`를 걸어준다.

## 다대다 매핑의 한계
- **편리해 보이지만 실무에서 사용하지 않는다.**
- 연결 테이블이 단순히 연결만 하고 끝나지 않는다.
- 연결 테이블에 주문 시간, 수량 같은 데이터가 들어올 수 있다.
- 연결 테이블에 외래키 정보를 제외한 추가적인 데이터를 구성할 수 없다.

![](https://raw.githubusercontent.com/oasis791/blog-posting/4637418bebb49ec00bc9cea1eeb7de3cd246ddd8/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/6.%EB%8B%A4%EC%96%91%ED%95%9C%20%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84%20%EB%A7%A4%ED%95%91/4.%EB%8B%A4%EB%8C%80%EB%8B%A4%20%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84/3.png)

## 다대다 매핑의 한계 극복
- **연결 테이블용 엔티티를 추가한다. (연결 테이블을 엔티티로 승격시킨다.)**
- **`@ManyToMany` 를 `@OneToMany`, `@ManyToOne`으로 분리한다.**

```java
@Entity  
public class MemberProduct {  
  
	@Id @GeneratedValue  
	@Column(name = "MEMBER_PRODUCT")  
	private Long id;  
	  
	@ManyToOne  
	@JoinColumn(name = "MEMBER_ID")  
	private Member member;  
	  
	@ManyToOne  
	@JoinColumn(name = "PRODUCT_ID")  
	private Product product;  
	}  
}
```

MemberProduct를 엔티티로 생성하고, `@ManyToOne` 애노테이션으로 Member와 Product 간의 연관관계를 맺어준다.

```java
@Entity  
public class Member {  
  
	@OneToMany(mappedBy = "member")  
	private List<MemberProduct> memberProducts = new ArrayList<>();  
}
```

```java
@Entity  
public class Product {  

	@OneToMany(mappedBy = "product")  
	private List<MemberProduct> memberProducts = new ArrayList<>();
}
```

Member와 Product에 각각 `@OneToMany` 애노테이션으로 일대다 연관관계를 맺어주고, mappedBy를 통해 양방향 연관관계를 맺어준다.

**전통적인 방식에선 조인 테이블의 주키는 두 외래키를 합쳐 주키로 설정하지만, 운영상 편의를 위해 조인 테이블에서도 GeneratedValue를 통해 비즈니스 적으로 의미없는 값을 주키로 사용하는 것이 좋다.**