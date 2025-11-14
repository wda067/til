# 서브쿼리

## 개념

- 하나의 SQL 쿼리문 안에 포함된 또 다른 `SELECT` 쿼리
- 바깥쪽의 메인쿼리가 실행되기 전에 `()`안에 있는 서브쿼리가 먼저 실행된다.
- 데이터베이스는 서브쿼리의 실행 결과를 바깥쪽 메인쿼리에게 전달한다.

## 스칼라 서브쿼리

- 단일 행 반환
- **"특정 주문(`order_id = 1`)을 한 고객과 같은 도시 **/** 에 사는 모든 고객을 찾고 싶다."**
- 서브쿼리로 조건에 맞는 도시 조회하여 `=`로 비교
    ```sql
    SELECT name, address
    FROM users
    WHERE address = (SELECT u.address
                     FROM users u
                     INNER JOIN orders o ON u.user_id = o.user_id
                     WHERE o.order_id = 1);
    ```

## 다중 행 서브쿼리

- `IN` 연산자
- **"'전자기기' 카테고리에 속한 모든 상품 **/** 들을 주문한 주문 내역을 전부 보고 싶다."**
- 서브쿼리로 '전자기기'에 해당하는 상품 ID 목록 조회하여 `IN`로 필터링
    ```sql
    SELECT * FROM orders
    WHERE product_id IN (SELECT product_id
                         FROM products
                         WHERE category = '전자기기');
    ```
- `ANY`나 `ALL`은 잘 사용하지 않고, `IN` 연산자나 `MIN()`, `MAX()` 같은 집계 함수로 대체

## 다중 컬럼 서브쿼리

- **"우리 쇼핑몰의 특정 고객(`user_id = 2`)이 한 주문(`order_id = 3`)이 있다. 이 주문 **/** 과 동일한 고객이면서 주문 처리 상태(`status`)도 같은 모든 주문을 찾아보자."**
- `order_id = 3`인 고객 ID와 주문 상태 조회
    ```sql
    SELECT user_id, status
    FROM orders
    WHERE order_id = 3;  //PK이므로 단일행 반환
    ```
- 다중 컬럼 비교
    ```sql
    SELECT order_id, user_id, status
    FROM orders
    WHERE (user_id, status) = (SELECT user_id, status
                               FROM orders
                               WHERE order_id = 3);
    ```
- **"각 고객별로 가장 먼저 한 주문 **/** 의 주문ID, 사용자ID, 주문 날짜를 조회해라"**
- 각 고객별로 첫 주문 조회
    ```sql
    SELECT user_id ,MIN(order_date)
    FROM orders
    GROUP BY user_id;
    ```
- 다중 행 비교 -> `IN` 연산자
    ```sql
    SELECT order_id,
        user_id,
        order_date
    FROM orders
    WHERE (user_id, order_date) IN (SELECT user_id, MIN(order_date)
                                    FROM orders
                                    GROUP BY user_id);
    ```

## 상관 서브쿼리

- 메인쿼리와 서브쿼리가 서로 연관 관계를 맺고 동작하는 서브쿼리
- **"각 상품별로, 자신이 속한 카테고리의 평균 가격 이상의 상품들을 찾아라."**
    ```sql
    SELECT product_id,
           name,
           category,
           price
    FROM products p1
    WHERE price >= (SELECT AVG(price)
                    FROM products p2
                    WHERE p2.category = p1.category);
    ```
- **"한 번이라도 주문된 상품 조회하기"**
    ```sql
    SELECT *
    FROM products
    WHERE product_id IN (SELECT DISTINCT product_id FROM orders);
    ```
- `IN` 방식은 테이블이 매우 클 경우 성능 문제를 일으킬 수 있다.
- 서브쿼리가 반환한 목록 전체를 메모리에 저장한 뒤, 메인쿼리의 각 행과 비교해야 하기 때문이다.
- 이럴 때 `EXISTS`를 사용하면 효율적으로 쿼리를 실행할 수 있다.
    ```sql
    SELECT *
    FROM products p
    WHERE EXISTS(SELECT 1
                FROM orders o
                where o.product_id = p.product_id);
    ```

## SELECT 서브쿼리

- 서브쿼리를 `SELECT`절 안에 사용함으로써 하나의 '컬럼'으로 동작한다.
- **"전체 상품 목록을 조회하면서, 각 상품별로 총 몇 번의 주문이 있었는지 '총 주문 횟수'를 함께 보여주고 싶다."**
    ```sql
    SELECT product_id,
        (SELECT COUNT(*) FROM orders o WHERE o.product_id = p.product_id) as order_count
    FROM products p;
    ```
- 스칼라 서브쿼리는 `JOIN`으로는 표현하기 복잡한 로직을 직관적으로 표현할 수 있다.
- 하지만 **성능 저하의 가능성**이 있다.
- 상관 서브쿼리는 메인쿼리가 반환하는 행의 수만큼 서브쿼리가 반복 실행되기 때문이다.
- 따라서 적재적소에 사용해야 한다.
- 앞선 문제는 `JOIN`으로 해결할 수도 있다.
    ```sql
    SELECT p.product_id, p.name, COUNT(o.order_id) AS order_count
    FROM products p
    LEFT JOIN orders o ON p.product_id = o.product_id
    GROUP BY p.product_id, p.name;
    ```

## 테이블 서브쿼리

- `FROM`절에서 하나의 독립된 테이블로 사용하는 서브쿼리
- **"각 상품 카테고리별로, 가장 비싼 상품의 이름과 가격을 조회하고 싶다."**
- 카테고리별로 `GROUP BY`로 묶고 가장 비싼 가격은 `MAX()`를 사용하면 되지만 이름은 해당 그룹에서 다중 행이 존재하기 때문에 오류가 발생한다. 
- 따라서 카테고리별 최고가를 먼저 알아낸 뒤, 그 가격과 일치하는 상품을 찾는다.
    ```sql
    -- 카테고리별 최고가
    WITH cmp AS (SELECT category, MAX(price) as max_price
                FROM products
                GROUP BY category)

    SELECT p.category, p.name, p.price
    FROM products p
    JOIN cmp on p.category = cmp.category AND p.price = cmp.max_price;
    ```

## JOIN vs 서브쿼리

- 성능은 쿼리 옵티마이저에 의해 일반적으로 JOIN이 더 좋은 경우가 많다.
- 반면 가독성 측면에서는 서브쿼리가 더 좋은 경우가 많다.
- 따라서 다음 순서대로 고려한다.

  1. **JOIN을 우선적으로 고려한다.**
  2. J**OIN으로 표현하기 너무 복잡하거나, 서브쿼리의 가독성이 훨씬 좋다면 서브쿼리를 사용한다.**
  3. **IN 대신 EXISTS를 사용한다.**
  4. **성능이 의심될 때는 반드시 측정한다.**
