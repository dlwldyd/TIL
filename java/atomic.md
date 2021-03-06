# Atomic 클래스
Atomic 클래스는 해당클래스의 원자성을 보장한다. synchornized 키워드를 사용하면 해당 키워드가 걸린 메서드나 객체 전체에 락이 걸리기 때문에 비용이 매우 크다. 하지만 Atomic 클래스를 사용하면 sychronized 키워드를 사용하는 것보다 비용적으로 더 싸고 동기화 문제를 해결할 수도 있다. Atomic 클래스는 CAS(Compare And Swap) 알고리즘을 이용하여 동기화 문제를 해결하는데 CAS 알고리즘이란 변수 값을 교체할 때 메인 메모리에 저장된 값(volatile 키워드를 사용하기 때문에 캐시 사용X)과 해당 스레드가 처음 조회했던 당시의 값을 비교하여 만약 값이 일치하면 새로운 값으로 교체하고 일치하지 않으면 실패한 것으로 간주하고(다른 스레드가 중간에 끼어들어 값을 바꿨기 때문) 처음부터 다시 값을 교체하는 것을 재시도한다.
* java docs : https://docs.oracle.com/javase/8/docs/api/index.html
* __Atomic 클래스 종류__
  + AtomicBoolean
    - get() : 현재 값을 반환한다.
    - set(boolean update) : 값을 변경한다.(원자성 보장됨)
    - getAndSet(boolean update) : 현재 값을 반환하고 값을 변경한다.(원자성 보장됨)
    - compareAndSet(boolean expect, boolean update) : expect와 현재 값이 일치할 때만 업데이트가 되고 true가 반환된다. 일치하지 않으면 업데이트 되지 않고 false가 반환된다.
  + AtomicInteger
    - get()
    - set(int update)
    - getAndSet(int update)
    - compareAndSet(int expect, int update)
    - getAndUpdate(람다식) : 현재 값을 반환하고 람다식으로 리턴된 값으로 변경한다.
    - updateAndGet(람다식) : 람다식으로 리턴된 값으로 변경한 후 변경된 값을 반환한다.
    - getAndIncrement() : 현재 값 리턴하고 +1 증가시킨다.
    - incrementAndGet() : +1 증가시키고 변경된 값 리턴한다.
    - getAndDecrement() : 현재 값 리턴하고 -1 감소시킨다.
    - decrementAndGet() : -1 감소시키고 변경된 값 리턴한다.
    - getAndAdd(int value) : 현재 값 리턴하고 현재 값에 value를 더한다.
    - addAndGet(int value) : 현재 값에 value를 더하고 그 결과를 리턴한다.
  + AtomicIntegerArray
    - length : 길이 반환
    - get(int index)
    - set(int index, int update)
    - getAndSet(int index, int update)
    - compareAndSet(int index, int expect, int update)
    - getAndUpdate(int index, 람다식)
    - updateAndGet(int index, 람다식)
    - getAndIncrement(int index)
    - incrementAndGet(int index)
    - getAndDecrement(int index)
    - decrementAndGet(int index)
    - getAndAdd(int index, int value)
    - addAndGet(int index, int value)
  + AtomicReference
    - AtomicReference는 객체를 wrapping하는 클래스이다.(new AtomicReference(T object))
    - get()
    - set(T update)
    - getAndSet(T update)
    - compareAndSet(T expect, T update) : equalsAndHashcode가 오버라이드 되야한다.
    - getAndUpdate(람다식)
    - updateAndGet(람다식)
  + AtomicReferenceArray
    - AtomicReferenceArray는 객체의 배열을 wrapping하는 클래스이다.(new AtomicReferenceArray(T[] objects))
    - length : 길이 반환
    - get(int index)
    - set(int index, T update)
    - getAndSet(int index, T update)
    - compareAndSet(int index, T expect, T update)
    - getAndUpdate(int index, 람다식)
    - updateAndGet(int index, 람다식)