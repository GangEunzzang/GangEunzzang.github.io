---
title: Redis Lettuce SETNX 정리
date: 2025-01-09 23:30:00 +09:00
categories: [spring]
tags:
  [
    java,
    redis,
    lettuce,
    SETNX,
    스프링
  ]
---

* * *

분산락을 구현하기 위해 Redis의 SETNX는 자주 사용된다.
어떤 상황에서 적용하면 좋은지, 어떤식으로 사용하는지 정리해보겠다.

<br><br>

## ✅ SETNX란?

`SETNX`는 Redis의 명령어 중 하나로, `SET Not Exists`의 약자이며, 키가 존재하지 않을 때만 값을 설정하는 명령어이다.  
즉, 키가 존재하지 않을 때만 값을 설정하고, 키가 존재할 경우 아무런 작업도 수행하지 않는다.  

아래와 같은 커맨드로 SETNX를 사용할 수 있다.  
`SETNX {key} {value}`

### 📌 예제
```redis
SETNX key1 value1
```

![img.png](../../assets/img3/img.png)
![img_1.png](../../assets/img4/img_1.png)

위와같이 key1로 lock 생성을 했으면 true를 반환해주고,  
이미 key1이 존재하면 false를 반환해준다.

즉, SETNX는 위와 같은 개념을 바탕으로 분산락을 구현할 때 사용된다.

## ✅ 분산락 구현

Redis 클라이언트 라이브러리인 `Lettuce`를 기반으로 `Spring Data Redis` 를 사용하여 분산락을 구현해보자

<br>

### 📌 Redis 분산락의 주요 로직
`락 획득 (Acquire Lock)`  
SETNX를 통해 키가 없을 경우 락을 생성
락에 만료 시간을 설정하여 무한 대기 방지

`비즈니스 로직 실행`  
락이 성공적으로 획득된 경우, 해당 작업을 실행

`락 해제 (Release Lock)`  
작업이 끝나면 락을 해제한다.
다른 클라이언트가 락을 사용 가능하도록 만든다.

<br>

### 📌 RedisLockService 구현
아래는 Redis의 SETNX 명령어를 사용하여 분산락을 구현한 간단한 예제이다.

```java
@RequiredArgsConstructor
@Service
public class RedisLockService {

    private final StringRedisTemplate redisTemplate;

    /**
     * 락 획득
     * @param key 락의 키
     * @param value 락의 값 (보통 고유한 UUID 사용)
     * @param timeout 락의 만료 시간 (초 단위)
     * @return 락 획득 성공 여부
     */
    public boolean acquireLock(String key, String value, long timeout) {
        Boolean result = redisTemplate.opsForValue()
                .setIfAbsent(key, value, timeout, TimeUnit.SECONDS);
        return Boolean.TRUE.equals(result);
    }

    /**
     * 락 해제
     * @param key 락의 키
     * @param value 락의 값 (락을 소유한 클라이언트만 해제할 수 있도록)
     */
    public void releaseLock(String key, String value) {
        String currentValue = redisTemplate.opsForValue().get(key);
        if (value.equals(currentValue)) {
            redisTemplate.delete(key);
        }
    }
}

```

<br>

### 📌 서비스에 락 적용하기
위에서 구현한 RedisLockService를 활용하여, 특정 작업(예: 티켓 구매)에 락을 적용할 수 있다.

```java
@RequiredArgsConstructor
@Service
public class TicketService {

    private final TicketRepository ticketRepository;
    private final RedisLockService redisLockService;

    public void purchaseTicket(Long ticketId, int quantity) {
        String lockKey = "ticket:" + ticketId;
        String lockValue = UUID.randomUUID().toString();

        // 락 획득
        boolean lockAcquired = redisLockService.acquireLock(lockKey, lockValue, 10);
        if (!lockAcquired) {
            throw new RuntimeException("Could not acquire lock for ticket purchase");
        }

        try {
            // 티켓 구매 로직
            Ticket ticket = ticketRepository.findById(ticketId)
                    .orElseThrow(() -> new RuntimeException("Ticket not found"));

            if (ticket.getQuantity() < quantity) {
                throw new RuntimeException("Not enough tickets available");
            }

            ticket.decreaseQuantity(quantity);
            ticketRepository.save(ticket);

        } finally {
            // 락 해제
            redisLockService.releaseLock(lockKey, lockValue);
        }
    }
}
```

<br>

### 📌 어떤 상황에서 위 코드를 적용하면 좋을까?

위 코드는 `락 획득 실패 시 즉시 종료`하는 방식으로 설계되었다. 이러한 방식은 다음과 같은 상황에서 유용하다.

`1.중요도가 낮은 작업`
* 작업이 반드시 수행될 필요는 없고, 실패해도 큰 문제가 없는 경우
* 예를 들어 티켓 구매 시도가 실패해도 사용자가 다시 요청을 시도하거나, 다른 티켓을 선택 할 수 있는 상황

`2.다른 대안이 있는 경우`
* 특정 자원(티켓, 상품 등)에 대한 작업이 실패해도, 다른 자원으로 대체할 수 있는 경우
* 예를 들어 특정 좌석의 티켓 예매에 실패했을 때, 사용자가 다른 좌석을 선택할 수 있는 구조

`3.실시간 응답 속도가 중요한 경우`
* 빠른 응답이 중요한 시스템에서 락 획득 실패 시 재시도를 하지않고 바로 종료함으로써 전체적인 부하 감소

`4. 락 경쟁이 드문 경우`
* 락 경쟁이 드문 경우에는 락 획득 실패가 발생할 확률이 낮아, 락 획득 실패 시 즉시 종료해도 큰 문제가 없는 경우

### 📌 재시도가 필요한 경우

`1. 작업 성공이 중요한 경우`
* 특정 작업이 반드시 수행되어야 하며, 락을 얻기 위해 재시도할 가치가 있는 경우
* 예를 들어 VIP 고객의 티켓 예매 요청은 실패할 수 없고, 몇 초 동안이라도 락을 재시도하여 반드시 완료해준다. (비즈니스 관점으로 적절한 예는 아니지만 알잘딱깔센..)

`2. 락 경쟁이 치열한 경우`
* 락 경쟁이 치열한 경우에는 락 획득 실패 시 재시도를 통해 락을 얻을 확률을 높일 수 있음
* 예를 들어 특정 상품의 재고를 감소시키는 작업이 많은 클라이언트에서 동시에 발생하는 경우

`3. 비즈니스 이벤트의 중요성`
* 특정 이벤트가 실패하면 비즈니스적으로 큰 손실을 초래하는 경우
* 예를 들어 한정된 수량의 상품을 판매하는 이벤트에서, 락 획득 실패 시 재시도를 통해 이벤트를 성공적으로 완료

### 📌 재시도 로직 추가

```java
public void purchaseTicketWithRetry(Long ticketId, int quantity, int maxRetries, long retryDelayMillis) {
    String lockKey = "ticket:" + ticketId;
    String lockValue = UUID.randomUUID().toString();
    int attempts = 0;

    while (attempts < maxRetries) {
        attempts++;
        boolean lockAcquired = redisLockService.acquireLock(lockKey, lockValue, 10);
        if (lockAcquired) {
            try {
                processPurchase(ticketId, quantity);
                return; // 성공 시 종료
            } finally {
                redisLockService.releaseLock(lockKey, lockValue);
            }
        }

        // 재시도 전 대기
        try {
            Thread.sleep(retryDelayMillis);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new RuntimeException("Thread interrupted during retry delay", e);
        }
    }

    // 재시도 실패 시 예외 발생
    throw new RuntimeException("Could not acquire lock for ticket purchase after " + maxRetries + " retries");
}
```
위와 같이 적절한 재시도 횟수, 딜레이 시간을 설정하여 락 획득에 실패한 경우 재시도할 수 있다.

<br>

### 📌 즉시 실패 vs 재시도

|구분 | 즉시 실패 | 재시도
|---|---|---|
|특징 | 실패 시 빠르게 종료 | 제한된 횟수만큼 재시도
|적용 사례 | 응답 속도가 중요한 경우 | 작업 성공률이 중요한 경우
|장점 | 처리 속도가 빠르고 시스템 부담이 적음 | 성공률 증가, 중요한 작업의 안정성 보장
|단점 | 실패율이 높아질 가능성 | 재시도로 인해 지연 발생

<br>

### 📌 분산락 동작 원리

`1.락 생성`  
* acquireLock 메서드는 SETNX 명령어를 호출하여 특정 키(lockKey)에 대한 락을 생성
* 동시에 다른 클라이언트가 동일한 키로 락을 생성하려고 하면, false를 반환하여 실패

`2.만료 시간 설정`  
* 락 생성 시 timeout(10초)과 함께 설정
* 설정된 시간 내에 락이 해제되지 않으면, 자동으로 락이 만료

`3.작업 수행`  
* 락을 성공적으로 획득한 클라이언트만 티켓 구매 작업을 진행

`4.락 해제`  
* 작업이 끝난 후, 락의 값(lockValue)을 확인하여 현재 클라이언트가 락 소유자인 경우에만 해제
* 다른 클라이언트가 락을 잘못 해제하지 못하도록 방지할 수 있다.

<br>

## ✅ 문제점

### 📌 Spin Lock 부하 문제

```java
    @Test
    void testConcurrentTicketPurchase() throws InterruptedException {
        // Given
        Long ticketId = 1L;
        int threads = 50; // 50개의 스레드에서 동시에 티켓 구매 시도
        int purchaseQuantity = 1;

        // 스레드 풀과 CountDownLatch 설정
        ExecutorService executorService = Executors.newFixedThreadPool(threads);
        CountDownLatch latch = new CountDownLatch(threads);

        // When
        for (int i = 0; i < threads; i++) {
            executorService.execute(() -> {
                try {
                    ticketService.purchaseTicketWithRetry(ticketId, purchaseQuantity);
                } catch (RuntimeException | InterruptedException ignored) {
                } finally {
                    latch.countDown();
                }
            });
        }

        // 모든 스레드가 작업을 끝낼 때까지 대기
        latch.await();
        executorService.shutdown();

        // Then
        // 티켓 수량이 정확히 감소되었는지 확인
        Ticket ticket = ticketRepository.findById(ticketId).orElseThrow();
        assertThat(ticket.getQuantity()).isEqualTo(100 - threads);
    }

```

실제로 위와같은 테스트코드를 실행하며 Redis 내부에 얼마나 많은 요청이 가나 확인해봤더니  
평균적으로 850번 정도의 부하가 걸린다.

이는 싱글스레드로 동작하는 Redis의 특성상 많은 문제를 야기할 수 있다.

<br> 

#### 해결방법

1. `백 오프 알고리즘(Exponential Backoff)`을 적용하여 재시도 간격을 점차 증가시키는 방법

```java
int attempts = 0;
int maxAttempts = 10;
long delay = 100; // 초기 딜레이 100ms

while (attempts < maxAttempts) {
    if (redisLockService.acquireLock(key, value, timeout)) {
        try {
            // 작업 수행
        } finally {
            redisLockService.releaseLock(key, value);
        }
        break;
    }

    // 지수 백오프
    Thread.sleep(delay);
    delay *= 2; // 딜레이 증가
    attempts++;
}

```

위와 같이 재시도 간격을 점점 증가시키면 Redis에 대한 부하를 줄일 수 있다.

2. `Redisson`과 같은 Redis 분산락 라이브러리 사용

* Redisson은 Redis의 Lua 스크립트를 사용하여 락의 원자성과 만료 시간 관리를 개선한 구현체를 제공한다.
* 락의 만료 시간을 주기적으로 갱신하거나, 락의 소유 여부를 확인하는 로직을 개선할 수 있다.

위 글은 Redisson 라이브러리가 아닌 Lettuce을 설명하기 위한 글이라 자세한 설명은 생각하겠슴당..

<br>


### 📌 SETNX 동작 한계

* SETNX는 단순히 키가 존재하지 않을때만 값을 설정하기 때문에  `락의 TTL(Time To Live)`와 같은 기능을 직접 구현해야 한다.
* 락 만료 시간이 설정되지 않을 경우, 락을 획득한 클라이언트가 작업을 완료하지 않고 종료될 경우, 락이 영원히 유지될 수 있다.

Redis 모니터링 후 실제 나간 명령어를 보면 다음과 같다.

```text 
 "SET" "ticket:1" "81e45ede-8410-410b-8f9f-712569d0e085" "EX" "10" "NX"
```

```bash
 SET {KEY} {VALUE} EX {TIMEOUT} NX
```

* `NX`: 키가 존재하지 않을 때만 설정
* `EX`: 만료 시간을 초 단위로 설정

위 명령어를 기반으로 락 설정과 TTL 설정을 하나의 명령으로 처리하므로 SETNX보다 안전하게 관리할 수 있다.

<br>

### 📌 락 해제시 Race Condition 문제
현재 `releaseLock` 메서드는 락의 소유 여부를 확인한 후 키를 삭제하는 두 단계로 이루어져 있다.

```java
    public void releaseLock(String key, String value) {
        String currentValue = redisTemplate.opsForValue().get(key);
        if (value.equals(currentValue)) {
            redisTemplate.delete(key);
        }
```
현재 Redis 명령어를 두 번 호출(`GET`, `DEL`)하므로, 락 해제 과정에서 다른 클라이언트가 락을 획득하는 `Race Condition` 문제가 발생할 수 있다.    

<br>

예를 들어:
>1. 클라이언트 A가 락을 소유하고 있고, 작업이 끝나서 락을 해제하려고 함
>2. 클라이언트 A가 `GET` 명령어로 현재 값을 확인하는 순간, 락 만료로 인해 다른 클라이언트(B)가 같은 락을 생성함
>3. 클라이언트 A는 이전에 확인했던 값을 기준으로 락 소유 여부가 확인되어 `DELETE` 명령어를 실행하지만, 이미 클라이언트 B가 생성한 락을 삭제하게됨

#### 해결방법

위와 같은 상황을 방지하기 위해 다중 명령어의 원자성을 보장해주는 Lua 스크립트를 적용할 수 있다.

```java
    String luaScript = "if redis.call('GET', KEYS[1]) == ARGV[1] then " +
      "return redis.call('DEL', KEYS[1]) " +
      "else return 0 end";
    
    redisTemplate.execute((RedisCallback<Object>) connection ->
      connection.eval(luaScript.getBytes(), ReturnType.INTEGER, 1,
      key.getBytes(), value.getBytes())
      );
```

위와 같이 Lua 스크립트를 사용하면, 실행 중간에 다른 명령이 개입할 수 없으므로, 다중 명령어의 실행 순서를 보장받을 수 있다.

### 📌 TTL 갱신 로직(Watchdog)
현재 로직에서는 TTL 갱신 로직이 없어, 처리 중일경우 락의 만료 시간을 갱신하지 않는다.  
Watchdog은 작업이 진행 중일 경우 주기적으로 TTL을 갱신하여 락이 만료되지 않도록 보장하는 방식이다.

```java
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);
scheduler.scheduleAtFixedRate(() -> {
  if (redisLockService.isLockHeld(lockKey, lockValue)) {
  redisLockService.extendTTL(lockKey, 10); // TTL 10초 연장
    }
      }, 0, 5, TimeUnit.SECONDS);

```

<br>

###  📌 락 획득 실패 시 처리 방식 문제

현재는 락 획득 실패 시 즉시 종료하거나 재시도하는 간단한 방식만 구현되어 있다.    
하지만, 락 획득 실패 시 어떻게 처리할지는 비즈니스 요구사항에 따라 다르다.  
예를 들어, 락 획득 실패 시 대기하거나, 다른 작업을 수행하거나, 예외를 발생시키는 등 다양한 방식으로 처리할 수 있다.  

  
#### 해결방법

* `Fallback 매커니즘`: 락 획득 실패 시, 해당 요청을 즉시 처리하지 못하는 상황에 대비하는 방식을 의미한다.  


* `Fallback 매커니즘 설계 예시`  
> 1. `요청을 큐에 넣기`: 작업을 비동기 방식으로 처리할 수 있도록 요청을 큐(예: RabbitMQ, Kafka)에 저장
> 2. `대체 동작 수행`: 실패한 작업 대신 대체 작업을 수행(예: 다른 좌석을 선택하세요 등)을 사용자에게 제공
>3. `작업 우선순위 조정`: 락 획득 실패 시, 해당 요청을 우선순위 큐에 저장하여 나중에 처리

<br>

```java
@Service
public class TicketService {

    private final RedisLockService redisLockService;
    private final BlockingQueue<Task> fallbackQueue = new LinkedBlockingQueue<>();

    public TicketService(RedisLockService redisLockService) {
        this.redisLockService = redisLockService;

        // 비동기적으로 큐의 작업을 처리하는 쓰레드
        new Thread(() -> {
            while (true) {
                try {
                    Task task = fallbackQueue.take(); // 큐에서 작업 가져오기
                    processTask(task);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        }).start();
    }

    public void purchaseTicket(Long ticketId, int quantity) {
        String lockKey = "ticket:" + ticketId;
        String lockValue = UUID.randomUUID().toString();

        boolean lockAcquired = redisLockService.acquireLock(lockKey, lockValue, 10);
        if (!lockAcquired) {
            // 락 획득 실패 시 큐에 작업 추가
            log.info("Failed to acquire lock. Adding to fallback queue...");
            fallbackQueue.add(new Task(ticketId, quantity));
            return;
        }

        try {
            // 비즈니스 로직 수행
            processPurchase(ticketId, quantity);
        } finally {
            redisLockService.releaseLock(lockKey, lockValue);
        }
    }

    private void processTask(Task task) {
        log.info("Processing fallback task: {}", task);
        purchaseTicket(task.getTicketId(), task.getQuantity());
    }
}

// 또는 아래와 같이 응답 핸들링
if (!lockAcquired) {
  log.warn("Failed to acquire lock for ticket purchase. Ticket ID: {}", ticketId);
    throw new TicketException("현재 요청을 처리할 수 없습니다. 잠시 후 다시 시도해주세요.");
}

```

위와 같은 방식은 여러 이점이 있다.
* `작업 실패를 줄임`: 요청을 큐에 저장함으로써 작업 실패를 줄일 수 있고, 경우에 따라 처리를 보장해줄수도 있다.
* `부하 완화`: 락 획득 실패 시, 재시도하지 않고, 큐를 통해 작업을 처리함으로써 부하를 완화할 수 있다.
* `사용자 경험 개선`: 즉시 처리하지 못하는 작업에 대해 적절한 피드백을 제공함으로써 사용자 경험을 개선할 수 있다.

<br><br>

## ✅ 분산락 주의사항
`1. 락 만료 시간 관리`
   * 작업 시간이 설정된 락의 만료 시간을 초과할 경우, 다른 클라이언트가 락을 획득할 수 있어 데이터 불일치가 발생할 수 있다.
   * 이를 방지하려면 작업 진행 중 주기적으로 락의 TTL을 갱신하는 방식(예: Watchdog)을 고려할 수 있다.  

`2. 락 해제 검증`
   * releaseLock에서 현재 클라이언트가 락의 소유자인지 확인하는 로직이 필요하다. Redis 트랜잭션을 사용하여 원자성을 보장하는 것이 좋다.

`3. Redis 분산락 대안`
   * 보다 안전한 분산락 구현을 위해 Redisson이나 ZooKeeper와 같은 라이브러리를 사용할 수 있다.
   * 특히 Redisson은 Redis의 Lua 스크립트를 사용하여 락의 원자성과 만료 시간 관리를 개선한 구현체를 제공한다.

<br><br>

## ✅ 마치며
* 락 만료 시간 관리나 복잡한 시나리오에 대해서는 추가적인 구현이 필요하다.
 * 비즈니스 요구사항에 맞는 분산락 구현 방식을 선택하는 것이 중요하다.
 * SETNX는 순서 보장이 되지 않으므로, 순서가 중요한 작업에는 적합하지 않다.
