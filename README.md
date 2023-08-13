# 재고시스템의 동시성 이슈

## Environment
- Mysql 8.x
- Java 17, Spring Boot 3.x
- 분산 환경이라는 가정 하에 실시

## Summary
1. 간단한 Stock 시스템 구현
2. 1개씩 감소시키는 테스트 코드로 정상 작동 확인
3. 멀티스레드 환경에서 100개의 감소 테스트 코드 작동 확인
   - Race Condition으로 인해 정확히 100개의 감소가 이뤄지지 않음
4. synchronized로 해결
5. database의 lock으로 해결
6. redis로 해결

## 동시성 문제 해결

### synchronized

> 분산 환경에서의 문제점

앱 내에서만 동시성 문제를 해결해주기 때문에 동일 DB를 사용하는 분산 환경에서는 동일하게 동시성 문제가 발생함

> 작동 시간의 지연

메서드에 하나의 스레드만 접근 가능하므로 100개의 스레드로 테스트 했을 때 많은 시간 차이가 나는 것을 알 수 있음
- Transactional: 300ms
- synchronized: 1100ms

### Pessimistic Lock (비관적 락)

- 데이터에 lock을 걸어 정합성을 맞추는 방법
- exclusive lock을 걸게되면 다른 트랜잭션에서는 lock이 해제되기 전까지 데이터를 가져갈 수 없게 됨
- 데드락이 걸릴 수 있기에 주의해야함

### Optimistic Lock (낙관적 락)

- Lock을 이용하지 않고 버전을 이용함으로써 정합성을 맞추는 방법
- 먼저 데이터를 읽고 update를 수행할 때 현재 내가 읽은 버전이 맞는지 확인하며 업데이트
- 내가 읽은 버전에서 수정사항이 생겼을 경우엔 애플리케이션에서 다시 읽은 후에 작업을 수행

### Named Lock

- 이름을 가진 metadata locking
- 이름을 가진 lock을 획득한 후 해제할 때까지 다른 세션은 이 lock을 획득할 수 없음
- Transaction이 종료될 때 lock이 자동 해제되지 않으므로 주의해야 함
- 별도의 명령어로 해제해주거나 선점 시간이 끝나야 함