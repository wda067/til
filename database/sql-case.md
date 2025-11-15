# CASE 문

## 단순 CASE 문

- 특정 하나의 컬럼이나 표현식의 값에 따라 결과를 다르게 하고 싶을 때 사용한다.
    ```sql
    SELECT order_id,
        user_id,
        product_id,
        quantity,
        status,
        CASE status
            WHEN 'PENDING' THEN '주문 대기'
            WHEN 'COMPLETED' THEN '결제 완료'
            WHEN 'SHIPPED' THEN '배송'
            WHEN 'CANCELLED' THEN '주문 취소'
            ELSE '알 수 없음' -- 예상치 못한 상태 값 처리
            END AS status_korean
    FROM orders;
    ```

## 검색 CASE 문

- 각 `WHEN`절에 독립적인 조건식을 사용하여 복잡한 논리를 구현할 때 사용한다.
    ```sql
    SELECT name,
        price,
        CASE
            WHEN price >= 100000 THEN '고가'
            WHEN price >= 30000 THEN '중가'
            ELSE '저가'
            END AS price_label
    FROM products;
    ```

## 그룹핑

- **"고객들을 출생 연대에 따라 '1990년대생', '1980년대생', '그 이전 출생'으로 분류하고, 각 그룹에 고객이 총 몇 명씩 있는지 알고 싶다."**
- `GROUP BY`절이 `SELECT`절보다 먼저 처리되지만, 대부분의 데이터베이스는 이러한 별칭 사용을 예외적으로 허용한다.
    ```sql
    SELECT CASE
            WHEN YEAR(birth_date) >= 1990 THEN '1990년대생'
            WHEN YEAR(birth_date) >= 1980 THEN '1980년대생'
            ELSE '그 이전 출생'
            END  AS birth_decade,
        COUNT(*) AS customer_count
    FROM users
    GROUP BY birth_decade;  -- SELECT 절에서 정의한 별칭
    ```

## 조건부 집계

##### 패턴 1: `COUNT(CASE ...)`

- `COUNT` 함수는 `NULL`이 아닌 모든 값을 센다.
```sql
COUNT(CASE WHEN status = 'COMPLETED' THEN 1 END)
```
- 따라서 `status`가 `COMPLETED`인 행의 개수만 세개 된다.

##### 패턴 2: `SUM(CASE ...)`

```sql
SUM(CASE WHEN status = 'COMPLETED' THEN 1 ELSE 0 END)
```
- 결과는 `COUNT` 함수와 같다.
