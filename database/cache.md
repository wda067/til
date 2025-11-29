# 캐시

## 데이터 지역성

- **시간적 지역성**: 최근에 접근한 데이터는 곧 또 접근될 가능성이 높다.
- **공간적 지역성**: 어떤 데이터에 접근했다면, 그 주변의 데이터도 곧 접근될 가능성이 높다.

## 캐싱에 적합한 데이터

- **시간적 지역성이 높고 변경이 적은 데이터**
  - 네이버 뉴스 메인 페이지
- **지역성은 낮지만 생성 비용이 높은 데이터**
  - 관리자용 통계 대시보드
- **공간적 지역성이 나타나는 연관 데이터**
  - 상품 상세 정보 + 관련 태그/옵션
- **공유 가능성이 높은 데이터**
  - 공유일 정보 

## 캐시 만료 정책

- **감지 가능한 변경**
  - **명시적 무효화**를 통해 즉시 캐시 갱신
- **감지할 수 없는 변경**
  - TTL을 설정하여 주기적으로 자동 만료되도록 처리

## 캐싱 전략

### Cache Aside

- 데이터가 필요한 시점에만 데이터를 캐시에 적재 (Lazy Loading)
- 구현이 간단하고 캐시 효율이 뛰어나다.
- 데이터가 변경되었을 때 실시간으로 반영되지 않아 데이터 불일치 가능성이 있다.

![](https://fern-freeze-290.notion.site/image/attachment%3A96499d9b-6e26-4047-942a-6a763cde1ebc%3Aimage.png?table=block&id=1dcade11-8e36-80f8-ab24-f679668ece53&spaceId=7eed1a7c-29f5-4e4f-b69c-06355da852f2&width=1360&userId=&cache=v2)

### Read Through

- 캐시에 데이터가 없을 때, 캐시 시스템이 자동으로 DB에서 데이터를 조회하고 캐시에 적재
- 세부 제어가 어렵다.

![](https://fern-freeze-290.notion.site/image/attachment%3Ac9b2a084-e6d4-449a-aac0-e0739e04002b%3Aimage.png?table=block&id=1dfade11-8e36-8090-8d11-db67b86f07e8&spaceId=7eed1a7c-29f5-4e4f-b69c-06355da852f2&width=1360&userId=&cache=v2)

#### Cache Aside vs Read Through

- **Cache Aside**는 흐름을 애플리케이션에서 직접 제어할 수 있기 때문에 현업에서 많이 사용

### Write Around

- 데이터를 DB에만 저장

![](https://fern-freeze-290.notion.site/image/attachment%3Af5dcaf40-ac8b-4e39-a3a1-b154310a2eb4%3Aimage.png?table=block&id=1dcade11-8e36-806b-9063-d6454be33606&spaceId=7eed1a7c-29f5-4e4f-b69c-06355da852f2&width=1360&userId=&cache=v2)

### Write Through

- 캐시와 DB 모두에 데이터를 저장
- 쓰기 성능이 떨어진다.

![](https://fern-freeze-290.notion.site/image/attachment%3A920600d0-9282-40dc-964a-20e52d43ea2b%3Awrite_through.png?table=block&id=1dcade11-8e36-805d-9358-c3444c9fa094&spaceId=7eed1a7c-29f5-4e4f-b69c-06355da852f2&width=1360&userId=&cache=v2)

### Write Back

- 데이터를 캐시에만 저장하고, DB에는 비동기적으로 저장
- 쓰기 성능은 뛰어나지만, 캐싱 시스템 장애 시 데이터 유실 가능성이 존재한다.

![](https://fern-freeze-290.notion.site/image/attachment%3Aec474fd6-1a60-40d7-bba2-d0e3b6cc5cd8%3Awrite-back.png?table=block&id=1dcade11-8e36-8007-aab5-efbfb871661c&spaceId=7eed1a7c-29f5-4e4f-b69c-06355da852f2&width=1360&userId=&cache=v2)

### 읽기 위주 + 변경이 드문 데이터

- Cache Aside + Write Around + TTL
- 국가별 공휴일 목록

1. 읽기 시 캐시에서 먼저 조회
2. 캐시에 없으면 DB에서 조회 후, 캐시에 저장
3. 쓰기 시에는 DB에만 쓰고, 캐시는 그대로
4. 캐시는 TTL에 의해 일정 시간 후 자동 만료

### 읽기 위주 + 변경이 드문 데이터 + 실시간 변경 반영

- Cache Aside + Write Around + TTL + 명시적 무효화
- 이벤트 상세 정보

1. 읽기 시 캐시에서 먼저 조회
2. 캐시에 없으면 DB에서 조회 후, 캐시에 저장
3. 쓰기 시, DB에 먼저 저장
4. 이후 같은 키를 가진 캐시를 즉시 명시적으로 무효화
5. 다음 읽기 시점에 다시 최신 데이터를 DB에서 불러와 캐시에 저장

### 한계

- 하지만 캐시 전략의 단일한 해답은 불가능에 가깝다.
- 서비스가 다루는 데이터 성격과 우선순위(성능, 일관성)에 따라서 서로 보완되는 방식으로 조합하여 설계되어야 한다.
- 또한, 여러 전략들을 조합하더라도 동시성 상황에서 일관성이 훼손되는 사례는 발생하며, 이를 방지하기 위해서는 락과 같은 동시성 제어가 함께 고려되어야 한다.
