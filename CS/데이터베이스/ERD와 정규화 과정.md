- ERD(Entity Relationship Diagram)
	- 릴레이션 간의 관계들을 정의한 다이어그램
## ERD의 중요성
- ERD는 **시스템의 요구 사항을 기반**으로 작성됨
- ERD를 기반으로 **데이터베이스를 구축**함
- 디버깅 또는 비즈니스 프로세스 재설계가 필요한 경우에 설계도 역할을 담당
- ERD는 **관계형 구조로 표현할 수 있는 데이터**를 구성하는 데 유용하지만, 비정형 데이터를 나타내기는 어렵다.
	- 비정형 데이터 : 비구조화 데이터, 미리 정의된 데이터 모델이 없거나 미리 정의된 방식으로 정리되지 않은 정보
## 정규화 과정
- 정규화 과정
	- 정규형 원칙을 기반으로 정규형을 만들어가는 과정
## 정규형(NF, Normal Form) 원칙
- 자료의 중복성 감소
- 독립적인 관계는 별개의 릴레이션으로 표현
- 각각의 릴레이션은 독립적인 표현이 가능
### 기본 정규형
<디폴트 테이블>
![기본 이미지](https://dotnettrickscloud.blob.core.windows.net/img/sqlserver/unf.png)

- 제1정규형(1NF)
	- **모든 도메인이 더 이상 분해될 수 없는 원자 값**만으로 구성되어야 함.
	- 릴레이션의 속성 값 중에서 한 개의 기본키에 대해 두 개 이상의값을 가지는 반복 집합이 있어서는 안된다.
	![제1정규형](https://dotnettrickscloud.blob.core.windows.net/img/sqlserver/1nf.png)
- 제2정규형(2NF)
	- 릴레이션이 제1정규형이며, **부분 함수의 종속성을 제거**한 형태
		- 부분 함수의 종속성 제거 : 기본키가 아닌 모든 속성이 기본키에 완전 함수 종속적인 것을 말함.
	- 주의: 릴레이션을 분해할 때 동등한 릴레이션으로 분해해야 하고, 정보 손실이 발생하지 않는 무손실 분해로 분해되어야 한다.
	![제2정규형](https://dotnettrickscloud.blob.core.windows.net/img/sqlserver/2nf.png)

- 제3정규형(3NF)
	- 제2정규형이고 기본키가 아닌 모든 속성이 **이행적 함수 종속**(transitive FD)을 만족하지 않는 상태
	![제3정규형](https://dotnettrickscloud.blob.core.windows.net/img/sqlserver/3nf.png)
	- 
- 보이스/코드 정규형(BCNF)
### 고급 정규형
- 제4정규형
- 제5정규형


