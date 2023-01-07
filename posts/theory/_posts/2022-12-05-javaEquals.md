---
layout: post
title: "Java Object class의 메소드들 (이펙티브자바 3장 독후감)"
tags: [java, Effective java]
comments: true
---

## Equals 메소드를 꼭 오버라이딩 해야할까?
반드시 할필요가 없으며, 하더라도 제대로 안하면 찾기 힘든 버그를 발생시킬수 있다. 다음 사항에 속한다면 equals 메소드를 오버라이딩을 하지 않는것이 좋다.
- 클래스의 인스턴스가 unique 하다면 (equals를 통해 비교할일이 없기때문에 )
- 그렇다면, 그냥 equals를 통해 비교할일이 없다면 하지 않는것이 좋다. (ex: 정규식)
- 이미 클래스의 부모 클래스가 equals를 오버라이딩 했고, 그 equals 메소드가 적절할때
- 클래스가 private, protected 면서, equals 메소드가 사용되지 않을것으로 예상된다면

## 언제, 어떻게 Equals 메소드를 오버라이딩 해야할까?
객체의 값의 비교를 통해 같고, 다름을 비교하고자 할때 (Integer, Long) 오버라이딩 해야 한다.
- **Reflexive** (x.equals(x) == true) 를 만족해야 한다.
- **Symmetric** (a.equals(b) == b.equals(a)) 를 만족해야 한다.
    - 어떤 상황에 발생할수있을까? 만약 비교하고자 하는 두객체가 다른 타입의 클래스라면 문제가 발생할수 있다. 책에서는 대소문자구분없는 문자열 클래스 (UnsensitiveString)와 String클래스를 비교하는것을 예시로 들었다. 만약 UnsensitiveString 에서 String 인스턴스와 equals 메소드를 실행한다면 대소문자구별을 하지 않으므로 true가 나오겠지만, 반대의 상황에서는 false가 나올수 있다.
    - 그래서 애초에 다른 클래스간의 비교를 할때는 false로 반환하는것이 좋다.
- **Transitivity (추이성을 만족해야 한다.)**
- **consistency** (항상 일관된 결과가 나와야 한다, 결과에 변경을 줄수 있는 요인(ip주소)을 비교하는것은 적절치 못하다.
- **not null: 모든 객체가 null과 같아선 안된다ㅏ.**

## 어떻게 추이성을 만족시킬것인가?
- 추이성을 만족시키는것은 쉽지 않았다.
    
    ```java
    /**
     * Thanks to effective java
     */
    public class Point {
    	private final int x;
    	private final int y;
    
    	public Point(int x, int y) {
    		this.x = x;
    		this.y = y;
    	}
    
    	@Override public boolean equals(Object o) {
    		if(!o instanceof Point)
    			return false;
    		Point p = (Point) o;
    		return p.x == x && p.y == y;
    	}
    }
    
    /**
     * Point를 상속한 ColorPoint 객채 생성
     */
    public class ColorPoint extends Point {
    	private final Color color;
    	
    	public ColorPoint(int x, int y, Color color) {
    		super(x, y);
    		this.color = color;
    	}
    
       /**
        * symmetric 문제가 있다.
        */
    	@Override 
        public boolean equals(Object o) {
            if(!o instanceof ColorPoint)
                return false;
            return super.equals(o) && ((ColorPoint) o).color == color;
        }
    
       /**
        * 이렇게 해도 다음과 같은 사항에서 추이성의 위반이 있다.
        * public static void main() {
            ColorPoint p1 = new ColorPoint(1,2, Color.RED);
            Point p2 = new Point(1,2);
            ColorPoint p3 = new ColorPoint(1,2, Color.BLUE);
            p1.equals(p2);    // true 
            p2.equals(p3);    // true 
            p1.equals(p3);    // false
          } 
        */
    	@Override 
        public boolean equals(Object o){
          if(!(o instanceof Point))
            return false;
          if(!(o instanceof ColorPoint))
            return o.equals(this);
          return super.equals(o) && ((ColorPoint) o).color == color;
        }
    
       /**
        * 그렇다고 해서 getClass를 사용하면 리스코프 치환 원칙을 위반하게 된다.
        * 왜냐하면 부모타입으로 교체했을때도 사용이 가능해야 하는데 class가 다르므로 사용할수가 없다!
        */
        @Override 
        public boolean equals(Object o){
            if(o == null || o.getClass() != getClass())
              return false;
            Point p = (Point) o;
            return p.x == x && p.y == y;
        }
    }
    ```
    

- 컴포지션을 활용하는 방법이 있다!
    
    ```java
    public class ColorPoint{
      /**
       * Point를 상속하는것이 아니라 컴포지션으로 조합한다.
       */
      private final Point point;
      private final Color color;
    
    	public ColorPoint(int x, int y, Color color) {
    		point = new Point(x, y);
    		this.color = Objects.requireNonNull(color);
    	}
    
      @Override 
      public boolean equals(Object o){
        /**
         * symmetric
         */
        if(!(o instanceof ColorPoint)){
          return false;
        }
        ColorPoint cp = (ColorPoint) o;
        
        /**
         * 각각의 equals를 통해 equals 여부를 판단한다.
         */
    	return cp.point.equals(point) && cp.color.equals(color);
      }
    }
    ```


## Equals 오버라이딩시 권장사항
- **기본 타입**: `==` 연산자 비교
- **참조 타입**: `equals` 메서드로 비교
- **float, double 필드:** 정적 메서드 `Float.compare(float, float)`와 `Double.compare(double, double)`로 비교
    - `Float.equals(float)`나 `Double.equals(double)`은 Object를 double이나 float로 변환하는 형변환(오토박싱) 때문에 성능상 좋지 않다.
- **배열 필드**: 원소 각각을 비교한다. 모두가 핵심 필드라면 `Arrays.equals()`를 사용한다.
- **null 정상값 취급 방지**:
    - `Object.equals(object, object)`로 비교하여 `NullPointException` 발생을 예방한다.
- 비교하기 **복잡한 필드를 가진 클래스**
    - 필드의 표준형(canonical form)을 저장한 후 표준형끼리 비교
- **필드의 비교 순서는 `equals` 성능을 좌우한다**
    - 다를 가능성이 크거나 비교하는 비용이 싼 필드부터 비교파생 필드가 객체 전체 상태를 대표하는 경우, 파생 필드부터 비교
- `equals`를 재정의할 땐 `hashCode`도 반드시 재정의한다.
- **Object 외의 타입을 매개변수로 받는 `equals` 메서드는 선언하지 말자.**
    - `public boolean equals(Otherclass o)`: 입력 타입이 Object가 아니므로 오버로딩한 것이다.
- 무엇보다 위에 적은 5가지 원칙을 잘지키는지 잘 확인하도록 하자

## equals를 오버라이딩 하게 되면 hashcode도 오버라이딩 한다.
hashcode는 HashMap(해시테이블) 등의 자료구조에 동일한 bucket에 담기는지 판별하는데에 쓰인다. 따라서 equals를 통해 다른 객체임이 판별되면 hashcode역시 다르게 나오도록 구현하여 해시테이블의 성능을 높일수 있다.

## 이미 있는 clone 메소드를 오버라이드 해야할지 판단해보자
```java
class MyObject {
    MyObject2[] datas;
}
```

MyObject 인스턴스를 그냥 clone 하게 되면 어떻게 될까? MyObject2 배열역시 recursive한 방식으로 clone 메소드가 호출되어서 복사되겠지만, 배열의 각각이 가리키는 원소는 같은 MyObject2 인스턴스를 포인팅 하게 되서 문제가 발생할수 있다. 그래서 아래와 같이 datas의 원소를 일일히다 복사해줘야 한다.

```java
class MyObject {
    MyObject2[] datas;

    @Override
    MyObject clone() {
        MyObject clonedMyObject = new MyObject();
        clonedMyObject.datas = new MyObject2[len(datas)];


        for(int i = 0; i < len(datas); ++i)
            clonedMyObject.datas[i] = datas[i].clone();

        return clonedMyObject;
    }
}
```

## 참고자료
- [https://velog.io/@lychee/이펙티브-자바-아이템-10.-equals는-일반-규약을-지켜-재정의-하라](https://velog.io/@lychee/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%EC%9E%90%EB%B0%94-%EC%95%84%EC%9D%B4%ED%85%9C-10.-equals%EB%8A%94-%EC%9D%BC%EB%B0%98-%EA%B7%9C%EC%95%BD%EC%9D%84-%EC%A7%80%EC%BC%9C-%EC%9E%AC%EC%A0%95%EC%9D%98-%ED%95%98%EB%9D%BC)
- 이펙티브자바
