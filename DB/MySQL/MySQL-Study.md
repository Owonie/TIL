### 데이터베이스 개념

테이블의 구조를 미리 설계하는 개념으로 설계도를 그리는 과정과 비슷하다. 프로젝트를 진행하기위해서 대표적으로 폭포수 모델(waterfall model) 을 사용한다.

프로젝트란 현실 세계에서 일어나는 업무를 컴퓨터 시스템으로 옮겨놓는 과정이다. 이는 객체지향이 하는 새로운 세계를 만드는 행위가 유사하다.

프로젝트의 규모가 커지면서 여러명이 같이 기능을 구현할 일이 많아진다. 이때 중요한 것이 바로 프로젝트의 설계다.

소프트웨어 개발 단계를 폭포수 모델로 표현할 수 있다.
![[Pasted image 20231205015256.png]]

데이터베이스 모델링은 폭포수 모델에서 프로그램 설계 쪽에 해당한다. 데이터베이스 모델링은 실세계에서 사용하는 사물 또는 작업을 DBMS의 데이터베이스 개체로 옮기기 위한 과정이다.

데이터베이스 모델링에는 정답이 없다. 하지만 좋은 설계와 안 좋은 설계는 분명 존재하니, 데이터베이스 모델링을 능숙하게 잘 다뤄야 튼튼한 프로젝트를 만들 수 있다.

DBMS(MySQL) 기준으로 하나의 데이터베이스 내부엔 행, 열, 기본키, 데이터, 데이터 형식, 열이름 등이 있다.

### 데이터베이스 생성 및 기본 조작

```sql
CREATE SCHEMA 'shop_db';
SELECT * FROM shop_db.member;
SELECT * FROM shop_db.member WHERE member_id = '고원';
```

### 데이터베이스 개체

테이블이 데이터베이스의 대표적인 개체다. 하지만 데이터베이스에는 테이블 외에 인덱스, 뷰, 스토어드 프로시저, 트리거, 함수, 커서 등의 개체도 사용된다.

- 인덱스
  빠르게 데이터를 조회할 수 있도록 도와주는 개체다. 인덱스는 찾아보기 기능 (책 뒤의 인덱스 부록) 과 유사하다. 인덱스는 없어도 데이터를 다룰 수는 있다, 다만 빠르게 조회를 하고 싶을 때 대용량 데이터 환경에선 인덱스를 사용하는 것이 훨씬 효율적이다.

조회를 직접 해보자.

```sql
SELECT * FROM member;
```

![[Pasted image 20231208012611.png]]
위의 테이블과 같이 현재 데이터가 저장되어있다. 테이블을 전부 조회하는 것은 Full Table Scan이라고 부른다.
![[Pasted image 20231208013952.png]]

여기서 PK를 기준으로 특정 값을 조회해보자:

```sql
SELECT * FROM member WHERE member_id = 'chu';
```

![[Pasted image 20231208012730.png]]
constant 로 조회를 하는 것을 볼 수 있다. 이 때 PK 조회는 이진 검색 알고리즘을 사용하기에 시간복잡도는 O(logn) 이다.

이번엔 PK 값이 아닌 컬럼 값을 기준으로 조회를 해보도록 하자:

```sql
SELECT * FROM member WHERE member_name = 'jiwoo';
```

![[Pasted image 20231208013040.png]]
이땐 Full Table Scan 방식 (테이블의 모든 row를 조회한다) 으로 시간복잡도는 O(n) 이며, 모든 레코드를 처음부터 끝까지 차례대로 확인해야한다. 이런 탐색 방식을 선형 검색이라고 부른다.

인덱스를 사용하면, 자주 사용되는 값들을 "표기" 해주기 때문에, 시간복잡도를 O(logn)을 갖을 수 있어 대용량 데이터베이스에선 훨씬 빠른 값 조회가 가능하다.

그럼 인덱스를 사용해보자:

```sql
CREATE INDEX idx_member_name ON member(member_name);
```

이렇게 인덱스를 표기해두고 다시 한번 직전의 PK가 아닌 값을 조회해보면 스캔 방식이 바뀐다는 것을 알 수 있다.

```sql
SELECT * FROM member WHERE member_name = 'jiwoo';
```

![[Pasted image 20231208013647.png]]
빨간색이었던 Full Table Scan 문구는 Non-Unique Key Lookup 으로 바뀌어있으며, Query cost 또한 0.65 에서 0.35로 낮아진 것을 알 수 있다.

데이터가 방대해질수록 인덱스의 역할은 매우 중요해진다. 하지만 이것 또한 트레이드 오프이기에, 항상 적용 대상을 잘 고려해서 데이터베이스를 설계해야한다.

### 뷰

뷰는 가상의 테이블이다. 사용자가 뷰 테이블에 접근하면 뷰는 실제 테이블에서 데이터를 가져와서 사용자에게 보여준다. 그렇기에 뷰는 사실상 테이블을 select 하는 구문으로 구성되어있다.

```sql
CREATE VIEW member_view
AS
	SELECT * FROM member;

```

뷰는 사실상 select 문의 덩어리다. 뷰를 실행하면 여러개의 select 문을 동시에 실행하는 것과 같다.

```sql
SELECT * FROM member_view;
```

만들어둔 뷰를 실행해보면 member 테이블을 모두 조회하는 것을 알 수있다.

### 스토어드 프로시저

프로그래밍 기능을 mysql에서 사용하고 싶을 때 프로시저를 사용할 수 있다.

```sql
DELIMITER //
CREATE PROCEDURE myProc()
BEGIN
	SELECT * FROM member WHERE member_name = 'jiwoo';
	SELECT * FROM product WHERE product_name = '응원봉';
END //
DELIMITER ;
```

프로시저를 사용할 떄는 예약어들이 좀 많다. 주의해서 사용하도록 하자.

```sql
CALL myProc();
```

지정해둔 프로시저 함수를 호출하면, 내부에 작성되어있는 쿼리문들을 실행할 수 있다.
