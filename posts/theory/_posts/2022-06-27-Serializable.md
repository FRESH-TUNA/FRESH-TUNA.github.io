---
layout: post
title: "Serializable, Jackson"
tags: [Java]
comments: true
---

## 1. Serializable ?
java.io 패키지에서 제공하는 직렬화/역직렬화 기능을 제공하는 인터페이스다. 이 인터페이스는 구현해야할 메소드는 없지만 marker의 역활을 한다. 이 인터페이스를 
상속 받은 클래스는 JVM을 통해 직렬화/역직렬화가 가능하다. Serializable을 구현한 클래스의 subtype들은 모두 직렬화/역직렬화가 가능하다.
생성한 객체를 파일로 저장할 때, 저장한 객체를 읽을 때, 다른 서버에서 생성한 객체를 받을 때 유용하게 사용할수 있다.

## 2. 직렬화/역직렬화
프로그래밍언어상의 객체를 byte[]로 변환하는것을 직렬화, 그 반대의 과정을 역직렬화라고 한다.

```java
public class AlgorithmResponse implements Serializable{
    private Long id;
    private String name;

    public AlgorithmResponse(Algorithm algorithm) {
        this.id = algorithm.getId();
        this.name = algorithm.getName();
    }
}
```
직렬화/역직렬화가 가능한 AlgorithmResponse 클래스이다. java.io.ObjectOutputStream 클래스를 이용하여 AlgorithmResponse 클래스를
아래와 같이 직렬화할수 있다.

```java
Algorithm algorithm;
AlgorithmResponse response = new AlgorithmResponse(algorithm);

byte[] serializedMember;
try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
    try (ObjectOutputStream oos = new ObjectOutputStream(baos)) {
        oos.writeObject(response);
        // serializedMember -> 직렬화된 member 객체 
        serializedMember = baos.toByteArray();
    }
}
// 바이트 배열로 생성된 직렬화 데이터를 base64로 변환
System.out.println(Base64.getEncoder().encodeToString(serializedMember));
```

역직렬화할때는 주의할점이 있는데, 직렬화와 역직렬화를 진행하는 시스템이 서로 다를 수 있다. 따라서 두 시스템에 같은 클래스가 있어야 한다!
역직렬화는 java.io.ObjectInputStream 를 사용하여 아래와 같이 진행할수 있다.
```java
// 직렬화 예제에서 생성된 base64 데이터 
String base64Member = "adfasdfawdfasf345qwsd";
byte[] serializedMember = Base64.getDecoder().decode(base64Member);
try (ByteArrayInputStream bais = new ByteArrayInputStream(serializedMember)) {
    try (ObjectInputStream ois = new ObjectInputStream(bais)) {
            // 역직렬화된 Member 객체를 읽어온다.
        Object objectMember = ois.readObject();
        AlgorithmResponse member = (AlgorithmResponse) objectMember;
    }
}
```

## 3. 역직렬화 주의점

```java
public class AlgorithmResponse implements Serializable{
    private Long id;
    private String name;
    private String difficulty;

    public AlgorithmResponse(Algorithm algorithm) {
        this.id = algorithm.getId();
        this.name = algorithm.getName();
        this.difficulty = algorithm.get(difficulty);
    }
}
```
직렬화를 한후 위와 같이 기존 클래스에 필드를 추가했다고 가정해보자.
아마도 역직렬화를 진행하게 되면 java.io.InvalidClassException 예외가 발생할것이다.

Serializable를 상속한 클래스에는 serialVersionUID 설정이 필요한테, 이를 설정하지 않을경우
클래스의 해시값을 사용하게 된다. 필드가 추가될경우 해쉬값이 달라지기 때문에 위와 같은 에러가 발생하는것이다.
아래와 같이 클래스를 수정해보자.

```java
public class AlgorithmResponse implements Serializable{
    private static final long serialVersionUID = 1004L;
    private Long id;
    private String name;
    private String difficulty;

    public AlgorithmResponse(Algorithm algorithm) {
        this.id = algorithm.getId();
        this.name = algorithm.getName();
        this.difficulty = algorithm.get(difficulty);
    }
}
```

## 4. Serializable 사용시 주의점
[우아한형제들 블로그글](https://techblog.woowahan.com/2551/) 을 참조하여 주의점을 아래와 같이 정리해보았다.
#### 1. 특별한 문제없으면 자바 직렬화 버전 serialVersionUID의 값은 개발 시 직접 관리

#### 2. serialVersionUID의 값이 동일하면 멤버 변수 및 메서드 추가는 크게 문제가 없다.
 
#### 3. 멤버 변수 제거 및 이름 변경은 오류는 발생하지 않지만 데이터는 누락된다.

#### 4. 역직렬화 대상의 클래스의 멤버 변수 타입 변경을 지양. 자바 역직렬화시에 타입에 엄격하여 예외 발생.

#### 5. 외부(DB, 캐시 서버, NoSQL 서버 등)에 장기간 저장될 정보는 자바 직렬화 사용을 지양한다. 역직렬화 대상의 클래스가 언제 변경이 일어나면 그동안 직렬화된 데이터는 쓸모가 없어진다.

## 5. 참고자료
[https://velog.io/@whitebear/%EC%9E%90%EB%B0%94-%EC%A7%81%EB%A0%AC%ED%99%94-%ED%99%95%EC%8B%A4%ED%9E%88-%EC%95%8C%EA%B3%A0-%EA%B0%80%EA%B8%B0](https://velog.io/@whitebear/%EC%9E%90%EB%B0%94-%EC%A7%81%EB%A0%AC%ED%99%94-%ED%99%95%EC%8B%A4%ED%9E%88-%EC%95%8C%EA%B3%A0-%EA%B0%80%EA%B8%B0)
