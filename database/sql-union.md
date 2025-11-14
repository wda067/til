# UNION

- `UNION`: 두 결과 집합을 합친 후, 중복된 행을 제거한다.
- `UNION ALL`: 중복 제거 과정 없이, 두 결과 집합을 그대로 합친다.

## 성능

- `UNION`은 중복을 제거하기 위해, 보통 전체 결과를 정렬한 다음, 서로 인접한 행들을 비교하며 중복을 찾는다. 따라서 대규모 데이터일 수록 비용이 발생한다.
- `UNION ALL`은 첫 번째 `SELECT` 결과 아래에 두 번째 `SELECT` 결과를 붙히기만 하면 된다.
- 중복을 제거해야 하는 요구사항이 있을 떄만 `UNION`을 사용한다.

## 정렬 

- `ORDER BY`절은 전체 `UNION` 연산의 가장 마지막에 한 번만 사용해야 한다.
    ```sql
    SELECT name, email, created_at AS event_date FROM users
    UNION ALL
    SELECT name, email, retired_date AS event_date FROM retired_users
    ORDER BY event_date DESC; -- 별칭을 사용하여 정렬
    ```