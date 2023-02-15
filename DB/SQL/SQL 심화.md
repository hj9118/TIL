# 심화편

## 메인쿼리 / 서브쿼리

- 메인 쿼리(주 쿼리): 가장 바깥쪽에 위치한 SELECT 문. 보통은 메인쿼리만 작성한다.
- 서브 쿼리 (부 쿼리): SELECT문 내 다른 SELECT문을 사용</br>
  </br>
  서브 쿼리 사용이유: WHERE절에서는 집약 함수를 통해 값을 구할 수 없으며 서브 쿼리를 통해 값을 다시 가져온다면 사용할 수 있기 때문.</br>
- 서브쿼리는 메인 쿼리보다 먼저 실행된다.
- 메인쿼리 이후에 작성되며 주로 WHERE절에 위치한다. (이외에도 작성 가능)

```sql
SELECT 컬럼 FROM 테이블
WHERE 컬럼 연산자 (SELECT ~);
```

</br>
- 예시1. price의 평균이상 값을 구할 때 - WHERE 절

```sql
SELECT id, price
FROM A
WHERE price >= (
  SELECT AVG(price)
  FROM A
);
```

- 서브 쿼리인 price의 평균값을 구한뒤 해당 값이 평균이상인 컬럼 값만 표시한다. </br>

- 예시2. 전체 레코드 수 가져오기 - SELECT 내 SELECT 작성

```sql
SELECT id, price,
(
  SELECT COUNT(*) FROM A
) AS count
FROM A
ORDER BY price
LIMIT 3;
```

- 개수를 구하는 서브 쿼리 값을 구한뒤 각 컬럼을 표시하며 price 순으로 3개만 표시

- 예시3. price가 평균 이하인 그룹 구하기 - HAVING절

```sql
SELECT id, AVG(price) FROM A
GROUP BY id
HAVING AVG(price) < (
  SELECT AVG(price) FROM A
);
```

- HAVING절로 평균값을 구한뒤 해당 값에 대한 조건이 일치하는 그룹을 표시

- 예시4. 테이블 여러개 사용하기

```sql
SELECT idA, user FROM A
WHERE idB = (
  SELECT idB FROM B
  WHERE typeB='abc'
);
```

- B 테이블의 idB 중 typeB가 abc인 값의 idB값을 구한다.
- idB의 값이 동일하는 컬럼 값을 반환

### 단일행 / 복수행 서브 쿼리

1개의 값을 반환할 경우 단일행 서브 쿼리</br>
여러행을 반환할 경우 복수행 서브 쿼리 라고 한다.
</br>

여러행을 반환하는 서브 쿼리에 대해서는 비교연산자를 사용할 수 없다.

#### 복수행 서브 쿼리에서 사용 가능한 연산자

| 연산자 | 사용법                  | 의미                                                 |
| ------ | ----------------------- | ---------------------------------------------------- |
| IN     | a IN (서브쿼리)         | a가 서브 쿼리 결과중 뭐든 a와 일치하면 1반환         |
| NOT IN | a NOT IN (서브쿼리)     | a가 서브 쿼리 결과 중 어느것도 일치하지 않다면 1반환 |
| ANY    | a 연산자 ANY (서브쿼리) | 서브 쿼리의 결과 중 a 연산 값과 같으면 1반환         |
| ALL    | a 연산자 ALL (서브쿼리) | 서브 쿼리의 결과 전체, a의 연산값이 1일이라면 1 반환 |

- ANY 예제

```sql
SELECT * FROM A
WHERE stock < ANY
(
  SELECT SUM(x) FROM B
  GROUP BY id
);
```
- B테이블의 x 컬럼의 개수를 구하여 id순으로 정렬한다.
- A테이블 전체 컬럼을 반환. 단 stock이 서브쿼리 값보다 SUM(x)보다 작은 수 일 것.

#### 서브 쿼리 내 NULL 제거

```sql
SELECT * FROM 테이블
WHERE 컬럼 IN (
  SELECT 컬럼 FROM 테이블
  WHERE 컬럼 IS NOT NULL
)
```

#### 주의점
1. IN, ANY 연산자 사용시 결과 내 NULL이 있다면 NULL로 반환될 수 있다. </br>
단, 일치하는 값이 있다면 해당 값을 반환한다.

2. LIMIT과 IN은 함께 사용할 수 없다.
- 대처 방법
```sql
SELECT id, user FROM A
WHERE id IN (
  SELECT id FROM (
    SELECT id FROM B
    ORDER BY price DESC
    LIMIT 3
  ) AS tmp
)
```

- 가장 안쪽에 있는 서브 쿼리에서 LIMIT 실행 단 AS를 통한 별명을 사용해야함.
- 그 다음 서브 쿼리 실행하여 IN 연산자 사용시 충돌하지 않도록 한다.

#### 서브 쿼리를 테이블로 만들기

```sql
SELECT AVG(cnt)
FROM (
  SELECT id, COUNT(*) AS cnt
  FROM A
  GROUP BY id
) AS a;
```
- id를 기준으로 그룹화하여 그룹별 개수를 센다.
- 테이블을 기준으로 값의 평균을 산정

## 상관 서브 쿼리
서브 쿼리지만 메인 쿼리와 연계하여 함께 실행한다.
예시 - 기본 사용 법
```sql
SELECT a.id, a.name FROM A
WHERE 3 < (
  SELECT SUM(stock) FROM B
  WHERE a.id = b.id
);
```
- A 테이블의 각 레코드마다 서브 쿼리를 실행하여 서브쿼리 내 연산과 일치하는 값을 찾는다.

---

메인 쿼리와 서브 쿼리에서 테이블이 나오며 테이블의 주인을 찾기 위해 `테이블.컬럼` 형식으로 점으로 연결하여 작성한다. </br>
</br>
예시

```sql
SELECT 컬럼, (
  SELECT ~ FROM 테이블2 AS B
  WHERE A.컬럼C = B.컬럼C
) FROM 테이블1 AS A
```

예시2 - 고객별 구입액 구하기
```sql
SELECT a.id, a.name, (
  SELECT SUM(b.price) FROM product AS b
  WHERE a.id = b.id
) AS res
FROM customer AS a;
```

- customer 테이블을 a, product 테이블을 b로 별명 지정
- id값이 동일하여 같은 인물의 구매내역임을 파악하여 각 id별 구매액 합계를 계산
- id, name, sum_price 값을 계산한다.

### EXISTS
상관 서브 쿼리 결과에 사용하여, 서브 쿼리의 결과가 존재하면 1을 반환.

```sql
SELECT a.col1, a.col2 FROM table1 AS a
WHERE EXISTS (
  SELECT * FROM table2 AS b
  WHERE a.col1 = b.col2
);
```
- 값이 있다면 true를 반환하여 레코드를 가져온다.
- 값이 없다면 false를 반환하여 레코드를 가져오지 않는다.

