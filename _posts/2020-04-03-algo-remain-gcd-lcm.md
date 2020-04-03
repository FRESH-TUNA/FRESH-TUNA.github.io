---
layout: post
title: "나머지 구하기, 최대공약수, 최소공배수 (BOJ 10430, BOJ 2609, BOJ 9613)"
author: "DONGWON KIM"
meta: "Springfield"
categories: "algorithm"
---
## 1. 배경지식 
### 1. 자료형
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

class RemainService {
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
## 3. GCD, LCM 문제 (BOJ 2609)
### 문제
두 개의 자연수를 입력받아 최대 공약수와 최소 공배수를 출력하는 프로그램을 작성하시오.

### 입력
첫째 줄에는 두 개의 자연수가 주어진다. 이 둘은 10,000이하의 자연수이며 사이에 한 칸의 공백이 주어진다.

### 출력
첫째 줄에는 입력으로 주어진 두 수의 최대공약수를, 둘째 줄에는 입력으로 주어진 두 수의 최소 공배수를 출력한다.

### 예제 입력
```bash
24 18
```

### 예제 출력
```bash
6
72
```

### 풀이 1
최대공약수와 최소공배수의 개념을 알고 있다면 쉽게 풀수 있는 문제이다.
최대공약수를 구하기 위해선 1부터 시작해서 두수의 가장 큰 약수를 구하면 된다. 
```python
# 최대공약수를 구하기 위한 의사코드
def gcd구하기
  최대공약수 = 1
  
  while 정수1 % 최대공약수 == 0  && 정수2 % 최대공약수 == 0:
    최대공약수 = 최대공약수 + 1

  return 최대공약수
```

최소공배수를 구하기 위해선 1부터 시작해서 두수의 가장 큰 배수를 구하면 된다. 다음은 최소공배수를 구하기 위한 의사코드이다.
```python
def lcm구하기
  최소공배수 = 1
  
  while 최소공배수 % 정수1 == 0  && 최소공배수 % 정수2 == 0:
    최소공배수 = 최소공배수 + 1

  return 최소공배수
```

이를 구현한 코드는 다음과 같다.
```java
import java.util.Scanner;

class Gcd {
  private int a, b;

  public Gcd(int a, int b) {
    this.a = a;
    this.b = b;
  }

  public int call() {
    int gcd = 1;
    while(this.a % gcd == 0 && this.b % gcd == 0) ++gcd;
    return gcd;
  }
}

class Lcm {
  private int a, b, gcd;

  public Lcm(int a, int b) {
    this.a = a;
    this.b = b;
  }

  public int call() {
    int lcm = 1;
    while(lcm % this.a == 0 && lcm % this.b == 0) ++lcm;
    return lcm;
  }
}

class Main{
  public static void main(String[] args) {
    int a, b;
    Scanner in = new Scanner(System.in);

    a = in.nextInt();
    b = in.nextInt();

    Gcd gcdCalculator = new Gcd(a, b);
    Lcm lcmCalculator = new Lcm(a, b);
    System.out.println(gcd.call());
    System.out.println(lcmCalculator.call());
    in.close();
  }
}
```

### 풀이 2
유클리드 호제법에 의하면 ,2개의 자연수(또는 정식) a, b에 대해서 a를 b로 나눈 나머지를 r이라 할때(단, a>b), a와 b의 최대공약수는 b와 r의 최대공약수와 같다. 
이를 활용하여 최대공약수를 구하는 알고리즘의 시간복잡도를 향상시킬수 있다. (  O(log(a+b))  )
<br/>
두수 a, b의 최대공약수를 gcd라고 가정하자, 그러면 a와 b는 다음과 같이 나타낼수 있다.
```bash
a = gcd * x
b = gcd * y
```
두수 a, b의 최소공배수는 'x * y * gcd' 이므로, 이를 아용하여 최대공약수를 알고 있다면 최소공배루를 다음과 같이 구할수 있다.
```bash
lcm = a * b / gcd
```
<br/>
다음과 같이 로직을 개선해보았다.

```java
import java.util.Scanner;

class Gcd {
  private int a, b;

  public Gcd(int a, int b) {
    this.a = a;
    this.b = b;
  }

  public int call() {
    return this.gcd(this.a, this.b);
  }

  public int gcd(int a, int b) {
    int remain = a % b;
    if(remain == 0)
      return b;
    else
      return this.gcd(b, remain);
  }

  public int gcd() {
    int gcd = 1;
    while (this.a % gcd == 0 && this.b % gcd == 0) ++gcd;
    return gcd;
  }
}

class Lcm {
  private int a, b, gcd;

  public Lcm(int a, int b, int gcd) {
    this.a = a;
    this.b = b;
    this.gcd = gcd;
  }

  public Lcm(int a, int b) {
    this.a = a;
    this.b = b;
  }

  public int call() {
    if(this.gcd != 0)
      return this.fastLcm();
    else
      return this.slowLcm();
  }

  public int fastLcm() {
    return this.a * this.b / this.gcd;
  }

  public int slowLcm() {
    int lcm = 1;
    while(lcm % this.a == 0 && lcm % this.b == 0)
      ++lcm;
    return lcm;
  }
}

class Main{
  public static void main(String[] args) {
    int a, b;
    Scanner in = new Scanner(System.in);  // Create a Scanner object
    int gcd;
    a = in.nextInt();
    b = in.nextInt();

    Gcd gcdCalculator = new Gcd(a, b);
    Lcm lcmCalculator = new Lcm(a, b, gcd = gcdCalculator.call());
    System.out.println(gcd);
    System.out.println(lcmCalculator.call());
    in.close();
  }
}
```

## 4. GCD 합 문제 (BOJ 9613)
### 문제
양의 정수 n개가 주어졌을 때, 가능한 모든 쌍의 GCD의 합을 구하는 프로그램을 작성하시오.

### 입력
첫째 줄에 테스트 케이스의 개수 t (1 ≤ t ≤ 100)이 주어진다. 각 테스트 케이스는 한 줄로 이루어져 있다. 각 테스트 케이스는 수의 개수 n (1 < n ≤ 100)가 주어지고, 다음에는 n개의 수가 주어진다. 입력으로 주어지는 수는 1,000,000을 넘지 않는다.

### 출력
각 테스트 케이스마다 가능한 모든 쌍의 GCD의 합을 출력한다.

### 예제 입력
```bash
3
4 10 20 30 40
3 7 5 12
3 125 15 25
```

### 예제 출력
```bash
70
3
35
```

### 풀이 
앞의 gcd 알고리즘문제를 잘풀었다면 쉽게 해결할수 있는 문제이다. 각각의 테스트케이스에 대해서 가능한 모든 쌍을 loop를 통해 얻어내어
최대공약수를 구한후 이를 모두 더해주면 된다.
입력으로 주어지는 수가 백만을 넘지 않는다고 했으므로, 백만의 숫자가 최대 100개로 하나의 테스트케이스에 주어질수 있다. 이때 가능한 쌍은 4450개 이고
4450개의 최대공약수는 모두 백만이 된다. 이때는 64bit 자료형인 'long'타입을 사용하는것이 바람직하다.

핵심 의사코드는 다음과 같다.
```python
def 모든gcd합구하기
  gcd합 = 0
  
  while 하나의쌍 in 모든쌍():
    gcd합 += gcd구하기(하나의쌍)

  return gcd합
```
이를 java로 다음과 같이 구현하였다.

```java
import java.util.Scanner;

class Gcd {
  private int a, b;

  public Gcd(int a, int b) {
    this.a = a;
    this.b = b;
  }

  public int call() {
    return this.gcd(this.a, this.b);
  }

  public int gcd(int a, int b) {
    int remain = a % b;
    if(remain == 0)
      return b;
    else
      return this.gcd(b, remain);
  }
}

class GcdSum {
  private int[] integers;

  public GcdSum(int[] integers) {
    this.integers = integers;
  }

  public long call() {
    long result = 0;
    for(int i = 1; i < this.integers.length; ++i){
      for(int j = i + 1; j < this.integers.length; ++j){
        Gcd gcdCalculator = new Gcd(this.integers[i], this.integers[j]);
        result += gcdCalculator.call();
      }
    }
    return result;
  }
}

class GcdSumService {
  private int caseCount;
  Scanner scanner;

  public GcdSumService(int caseCount, Scanner scanner) {
    this.caseCount = caseCount;
    this.scanner = scanner;
  }

  public void initIntegers(int[] integers, int numOfIntegers) {
    for(int i = 1; i < numOfIntegers + 1; ++i) {
      integers[i] = this.scanner.nextInt();
    }
  }

  public void call() {
    while(caseCount-- != 0) {
      int numOfIntegers = scanner.nextInt();
      int[] integers = new int[numOfIntegers + 1];
      this.initIntegers(integers, numOfIntegers);
      GcdSum gcdSum = new GcdSum(integers);
      System.out.println(gcdSum.call());
    }
  }
}

public class Main {
  public static void main(String[] args) {
    Scanner scanner = new Scanner(System.in);
    int caseCount = scanner.nextInt();
    GcdSumService gcdSumService = new GcdSumService(caseCount, scanner);
    gcdSumService.call();
    scanner.close();
  }
}
```
