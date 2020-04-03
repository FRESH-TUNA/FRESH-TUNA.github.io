---
layout: post
title: "나머지 문제, 유클리드 호제법"
author: "DONGWON KIM"
meta: "Springfield"
categories: "algorithm"
---

## 1. 배경지식 
### 1. 최대공약수, 최소공배수
최대공약수란 주어진 n개의 자연수의 공통적인 약수중 가장큰값을 의미한다.
최대공배수는 주어진 n개의 자연수의 공통적인 배수중 가장 작은값을 의미한다.

### 2. 자료형
데이터를 얼마 만큼의 메모리공간에 적재하고 (메모리 용량) 메모리에 어떻게 적재할지 (메모리에 적재방법), 데이터에 대해 어떤 연산이 가능하지 정의한것을 '자료형' 이라고 한다.
자바언어에서는 자료형은 primitive type과 reference type으로 구분된다.
primitive type의 예를 들어보면 'int'가 있다. 'int'타입의 경우 메모리할당은 32bit 이고, 가능한 연산은 +, -, *, /, % 등이 있다.
같은 정수타입이라도 'int', 'long' 등의 메모리 용량에 차이가 있으므로 상황에 따라 다른 자료형을 활용하여 대응할수 있다.

## 2. 나머지 문제 (BOJ 10430)
### 문제
(A+B)%C는 (A%C + B%C)%C 와 같을까?
(A×B)%C는 (A%C × B%C)%C 와 같을까?
세 수 A, B, C가 주어졌을 때, 위의 네 가지 값을 구하는 프로그램을 작성하시오.

### 입력
첫째 줄에 A, B, C가 순서대로 주어진다. (2 ≤ A, B, C ≤ 10000)

### 출력
첫째 줄에 (A+B)%C, 둘째 줄에 (A%C + B%C)%C, 셋째 줄에 (A×B)%C, 넷째 줄에 (A%C × B%C)%C를 출력한다.

### 예제 입력
```bash
5 8 4
```

### 예제 출력
```bash
1
1
0
0
```

### 풀이
문제에 주어진 식을 그냥 그대로 가져다 쓰면 쉽게 해결할수 있는 문제이다.
우선 A, B, C의 범위는 2보다 크거나 같거나, 10000보다 작거나 같다. 문제를 보았을때, 요구되는 가장 큰값이 10000 * 10000 = 100,000,000
이므로 32bit 공간을 사용하는 int 자료형으로 커버할수 있다. (-2147483648 ~ 2147483647)
<br/>
또한 문제를 보면 'A%C', 'B%C'는 문제의 해답을 구할때 반복해서 쓰임을 알수 있다. 따라서 'A%C', 'B%C'는 따로 변수에 저장해두어서 재사용하면 연산을 아낄수 있다.
<br/>
함수호출이 발생하면 메모리 call stack에 push 되었다가, 함수 호출이 끝나면 call stack에서 pop 되는 과정을 거친다.
이문제의 해답을 구하기 위해서는 총 4가지 연산 결과를 출력해야 한다. 이때 4번 출력함수를 call 하는것보다는 한번 출력함수를 call 하도록 구현하는것이 좋다.


```java
import java.util.Scanner;

class Remain {
  private int a, b, c;

  public RemainService(int a, int b, int c) {
    this.a = a;
    this.b = b;
    this.c = c;
  }

  public void call() {
    int aDividedByC = this.a % this.c;
    int bDividedByC = this.b % this.c;

    // 출력함수 call 횟수를 1번으로 줄이기
    System.out.printf(
      "%d\n%d\n%d\n%d",
      this.add(this.a, this.b, this.c),
      this.add(aDividedByC, bDividedByC, this.c),
      this.multiply(this.a, this.b, this.c),
      this.multiply(aDividedByC, bDividedByC, this.c)
    );
  }

  private int add(int a, int b, int c) {
    return (a + b) % c;
  }

  private int multiply(int a, int b, int c) {
    return (a * b) % c;
  }
}

public class Main{
  public static void main(String[] args) {
    int a, b, c;
    Scanner in = new Scanner(System.in);
    
    a = in.nextInt();
    b = in.nextInt();
    c = in.nextInt();

    RemainService remain = new RemainService(a, b, c);
    remain.call();
    in.close();
  }
}
```
## 3. GCD, LCM 문제

## 4. GCD 합 문제

