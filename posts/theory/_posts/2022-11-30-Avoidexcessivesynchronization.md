---
layout: post
title: "Effective java 11장, 78절, Avoid excessive synchronization"
tags: [java]
comments: true
---

## 문제가 있는 예제 1 (예외발생)
```java
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import java.util.Set;

/**
 * 옵저버 패턴을 통해
 * set에 함수가 추가되면 콜백함수를 호출한다.
 */
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserver<E>> observers = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }

    /**
     * 문제의 함수
     * add된 숫자가 23일때를 가정해보자
     * 어찌됫건 콜백함수가 호출된다.
     * 그런데 synchronized 안에 있으므로 락이 풀린것이 아니다
     * 그러므로 added 함수의 removeObserver 함수를 실행하는데 문제가 없다.
     * 리스트를 순회하는도중에 삭제했으므로 exception이 터진다!
     * 이런 동기화블록 안에서 호출되는 클라이언트의 메소드는 무서운 외계인메소드가 될 가능성이 높다.
     */
    private void notifyElementAdded(E element) {
        synchronized (observers) {
            for (SetObserver<E> observer : observers) {
                observer.added(this, element);
            }
        }
    }

    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element);
        return result;
    }
}

/**
 * 옵져버 인터페이스
 */
@FunctionalInterface
public interface SetObserver<E> {
    void added(ObservableSet<E> set, E element);
}

public class Main {
    public static void main(String[] args) {
        ObservableSet<Integer> set =
                new ObservableSet<>(new HashSet<>());

        /**
         * 콜백함수를 추가한다.
         */
        set.addObserver(new SetObserver<Integer>() {
            @Override
            public void added(ObservableSet<Integer> set, Integer element) {
                System.out.println(element);
                if (element == 23)
                    set.removeObserver(this);//문제 발생
            }
        });

        for (int i = 0; i < 100; i++) {
            set.add(i);
        }
    }
}
```


## 문제가 있는 예제 2 (데드락)
```java
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import java.util.Set;

/**
 * 옵저버 패턴을 통해
 * set에 함수가 추가되면 콜백함수를 호출한다.
 */
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserver<E>> observers = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }

    /**
     * 문제의 함수
     * add된 숫자가 23일때를 가정해보자
     * 어찌됫건 콜백함수가 호출된다.
     * 그런데 synchronized 안에 있으므로 락이 풀린것이 아니다
     * 그러므로 added 함수의 removeObserver 함수를 실행하는데 문제가 없다.
     * 리스트를 순회하는도중에 삭제했으므로 exception이 터진다!
     */
    private void notifyElementAdded(E element) {
        synchronized (observers) {
            for (SetObserver<E> observer : observers) {
                observer.added(this, element);
            }
        }
    }

    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element);
        return result;
    }
}
@FunctionalInterface
public interface SetObserver<E> {
    // ObservableSet에 원소가 추가되면 호출된다.
    void added(ObservableSet<E> set, E element);
}

public class Main {
    public static void main(String[] args) {
        ObservableSet<Integer> set =
                new ObservableSet<>(new HashSet<>());

        /**
         * 콜백함수를 추가한다.
         * 새로운 쓰레드가 실행된다.
         * 새로운 쓰레드는 락이 없는 상태이다.
         * 그런데 notifyElementAdded 함수에 의해 이미 락이 걸려 있다.
         * 결국 데드락 상태에 빠지게 된다. 
         */
        set.addObserver(new SetObserver<Integer>() {
            @Override
            public void added(ObservableSet<Integer> set, Integer element) {
                System.out.println(element);
                if (e == 23) {
                    ExecutorService exec =
                         Executors.newSingleThreadExecutor();
                    try {
                        exec.submit(() ->
                        s.removeObserver(this)).get();
                    } catch (ExecutionException | InterruptedException ex) {
                        throw new AssertionError(ex);
                    } finally {
                        exec.shutdown();
                    } 
                }
            }
        });

        for (int i = 0; i < 100; i++) {
            set.add(i);
        }
    }
}
```


## 스냅샷을 통한 해결
```java
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import java.util.Set;

/**
 * 옵저버 패턴을 통해
 * set에 함수가 추가되면 콜백함수를 호출한다.
 */
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserver<E>> observers = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }

    /**
     * 일단 스냅샷을 딴 다음에
     * 스냅샷에 대해서만 콜백함수를 돌려주면
     * 쓰레드를 새로 생성하거나, 기존 쓰레드를 통해 removeObserver가 호출되더라도
     * 데드락이 걸리거나 예외가 터지지 않는다!
     */
    private void notifyElementAdded(E element) {
        List<SetObserver<E>> snapshot = null;
        synchronized(observers) {
            snapshot = new ArrayList<>(observers);
        }
        for (SetObserver<E> observer : snapshot)
            observer.added(this, element);
    }

    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element);
        return result;
    }
}
@FunctionalInterface
public interface SetObserver<E> {
    // ObservableSet에 원소가 추가되면 호출된다.
    void added(ObservableSet<E> set, E element);
}

public class Main {
    public static void main(String[] args) {
        ObservableSet<Integer> set =
                new ObservableSet<>(new HashSet<>());

        /**
         * 콜백함수를 추가한다.
         */
        set.addObserver(new SetObserver<Integer>() {
            @Override
            public void added(ObservableSet<Integer> set, Integer element) {
                System.out.println(element);
                if (e == 23) {
                    ExecutorService exec =
                         Executors.newSingleThreadExecutor();
                    try {
                        exec.submit(() ->
                        s.removeObserver(this)).get();
                    } catch (ExecutionException | InterruptedException ex) {
                        throw new AssertionError(ex);
                    } finally {
                        exec.shutdown();
                    } 
                }
            }
        });

        for (int i = 0; i < 100; i++) {
            set.add(i);
        }
    }
}
```


## CopyOnWriteArrayList 를 통한 해결
```java

/**
 * concurrent + snapshot feature를 모두 제공해준다!
 */
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
    observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}

private void notifyElementAdded(E element) {
    for (SetObserver<E> observer : observers)
        observer.added(this, element);
}

```


## 코멘트
- 과도한 동기화는 정합성을 지키기 위해 시스템 성능 저하를 불러 일으킨다.
- 높은 동시성이 필요하다면 thread-safe하게 설계하는것이 바람직하다.
- 동기화영역에서 외부 클라이언트의 메소드의 호출 (외계인메소드)은 안된다!

## 참고자료

Effective java 11장, 79절
