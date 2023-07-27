![김영한-JPA](https://raw.githubusercontent.com/oasis791/blog-posting/main/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/JPA%EB%A9%94%EC%9D%B8.png)

> 본 포스팅의 이미지 저작권은 [자바 ORM 표준 JPA 프로그래밍 - 기본편 (김영한)](https://www.inflearn.com/course/ORM-JPA-Basic) 강의에 있습니다.

## 연관관계 매핑시 고려사항 3가지
1. 다중성
	1. 다대일(N:1) : `@ManyToOne`
	2. 일대다(1:N) : `@OneToMany`
	3. 일대일(1:1) : `@OneToOne`
	4. 다대다(N:M) : `@ManyToMany`
2. 단방향인지 양방향인지
	- 테이블
		- 외래키 하나로 양쪽 조인 가능
		- 사실 방향이라는 개념이 없다.
	- 객체
		- 참조용 필드가 있는 쪽으로만 참조 가능
		- 한쪽만 참조 가능하면 -> 단방향
		- 양쪽이 서로 참조하면 -> 양방향
3. 연관관계의 주인
	- 테이블은 **외래키 하나**로 두 테이블의 연관관계를 맺는다.
	- 객체 양방향 관계는 A->B, B->A 처럼 **참조가 2군데**이다.
	- 객체 양방향 관계에서 테이블 외래키를 관리할 곳을 지정해야한다.
	- **연관관계의 주인**: 외래키를 관리하는 참조
	- 주인의 반대편: 외래키에 영향을 주지않음, **단순 조회**만 가능하다.

## 다대일(N:1)
- 가장 많이 사용하는 연관관계이다.

![](https://raw.githubusercontent.com/oasis791/blog-posting/80a85b68a2c1293be37a0953e82fd91e3a285dcb/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/6.%EB%8B%A4%EC%96%91%ED%95%9C%20%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84%20%EB%A7%A4%ED%95%91/1.%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84%20%EB%A7%A4%ED%95%91%EC%8B%9C%20%EA%B3%A0%EB%A0%A4%EC%82%AC%ED%95%AD%20%EB%B0%8F%20%EB%8B%A4%EB%8C%80%EC%9D%BC/1.png)

- DB 테이블 설계할 때, 항상 다(N)쪽에 외래키가 존재한다.
- 객체의 연관관계도 다(N)쪽에 참조를 걸면 된다.

## 다대일 단방향 정리
- 가장 많이 사용하는 연관관계
- **다대일**의 반대는 **일대다**
- `@ManyToOne` 애노테이션과 `@JoinColumn(name = "컬럼 이름")` 애노테이션을 통해, 외래키와 매핑해준다.

## 다대일 양방향

![](https://raw.githubusercontent.com/oasis791/blog-posting/80a85b68a2c1293be37a0953e82fd91e3a285dcb/JPA/%EC%9E%90%EB%B0%94_ORM_%ED%91%9C%EC%A4%80_JPA_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/6.%EB%8B%A4%EC%96%91%ED%95%9C%20%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84%20%EB%A7%A4%ED%95%91/1.%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84%20%EB%A7%A4%ED%95%91%EC%8B%9C%20%EA%B3%A0%EB%A0%A4%EC%82%AC%ED%95%AD%20%EB%B0%8F%20%EB%8B%A4%EB%8C%80%EC%9D%BC/2.png)

- 일(1)쪽 객체에 컬렉션으로 참조를 생성한다.
- **객체에 참조를 생성해도, 테이블에는 전혀 영향이 없다.**
- `@OneToMany(mappedBy = "연관관계 주인 필드 변수명")` 애노테이션을 통해 참조를 생성한다.

## 다대일 양방향 정리
- **외래키가 있는 쪽이 연관관계 주인이다.**
- 양쪽을 서로 참조하도록 개발할 때 추가한다.