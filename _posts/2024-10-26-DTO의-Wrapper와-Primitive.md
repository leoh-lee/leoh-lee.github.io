---
title: "DTO의 Wrapper와 Primitive"
date: "2024-10-26 18:36:11"
categories: [Java, Spring, Programming]
tags: [Java, Spring]
---

DTO를 만들다보면 `Integer`와 같은 Wrapper 클래스를 사용할 지, 원시타입(`int`) 그대로를 사용할 지 고민될 때가 있다. 둘은 어떤 차이가 있고 어떤 게 더 좋은 지 알아보자. 

### 원시타입과 Wrapper 클래스
원시타입과 Wrapper 클래스의 선택지를 알아보기 전에 각각이 무엇인 지 간단히 알아보자.

#### 원시타입 (Primitive Type)
자바에는 총 8개의 원시타입이 있다. 정수를 나타내는 `byte`,`short`, `int`, `long`, 문자를 나타내는 `char`,  실수를 나타내는 `float`, `double`, 논리(참과 거짓)를 나타내는 `boolean`이다. 
원시타입은 참조타입과 달리 Heap 메모리 주소를 참조하지 않고 스택 메모리에 값 그 자체가 저장된다.
이 특징으로 인해, 원시타입에서는 우리가 흔히 보는 `NullPointerException` 이 발생하지 않는다. 

> NullPointerException은  말 그대로 가리키고 있는 대상이 Null일 때 발생하는 예외이다. 참조타입의 경우 스택에서 Heap에 있는 객체 주소를 가리키게 되는데 가리키는 대상(주소)이 아무것도 없을 때 발생한다.

그러면 만약 원시타입에 아무것도 할당하지 않으면 어떻게 될까? NPE가 발생할까?
이는 두 가지 경우가 있다.
- 메서드 안에서 선언된 경우
```java
public void test() {
	int a;
	long b;
	char c;

	System.out.println(a); // >>> 예외 발생
}
```

이 경우에는 컴파일 시점에 예외가 발생한다. `Variable 'a' might not have been initialized`

- 클래스 에 선언된 경우
```java
static class PrimitiveTest {  
    static byte byteValue;  
    short shortValue;  
    int intValue;  
    long longValue;  
    float floatValue;  
    double doubleValue;  
    char charValue;  
    boolean booleanValue;  
  
    public void printAll() {  
        System.out.println("byteValue = " + byteValue);  
        System.out.println("shortValue = " + shortValue);  
        System.out.println("intValue = " + intValue);  
        System.out.println("longValue = " + longValue);  
        System.out.println("floatValue = " + floatValue);  
        System.out.println("doubleValue = " + doubleValue);  
        System.out.println("charValue = " + charValue);  
        System.out.println("booleanValue = " + booleanValue);  
    }}
```
외부에서 `printAll()`을 호출하면 결과는 다음과 같다.

![image](/assets/img/2024-10-26-DTO의-Wrapper와-Primitive/Pasted-image-20241026175539.png)

결과와 같이 선언할 때, 기본값들이 초기화된다.
| Type    | 기본값   |
| ------- | -------- |
| byte    | 0        |
| short   | 0        |
| int     | 0        |
| long    | 0        |
| float   | 0.0      |
| double  | 0.0      |
| char    | '\u0000' |
| boolean | false    |

> `char`에 초기화되는 문자는 null 문자이다.

이렇 듯 클래스 단위에서 원시타입을 선언하면 기본값이 초기화된다. 이는 변수를  `static` 으로 선언해도 마찬가지이다.

#### Wrapper Class
자바의 래퍼클래스는 위에서 본 원시타입을 객체로 감싸주는 참조 타입이다. 위에서 알아봤듯이 원시타입은 가리키는(참조하는) 객체가 없다는 특징을 가지고 있는데, 이 특징 때문에 단점 혹은 한계가 발생하는 경우가 있다. 예를 들어, 제네릭 타입으로 원시타입은 들어갈 수 없다.
```java
List<TestDto> list // OK
List<int> list // X! 불가능
```

제네릭 뿐만아니라 원시타입이 참조타입의 역할을 해야할 필요가 있는 경우를 위해 자바에서는 Wrapper Class를 제공한다.
| 원시타입 | Wrapper |
| -------- | ------- |
| byte     | Byte    |
| short    | Short   |
| int      | Integer |
| long     | Long    |
| float    | Float   |
| double   | Double  |
| char     | Char    |
| boolean  | Boolean |

> Wrapper Class 는 참조타입이므로, Heap에 Wrapper 객체를 생성하고 Stack에서 해당 객체 주소를 참조한다. 즉, null을 가질 수 있다는 뜻이다.

### 원시타입과 Wrapper의 차이
메서드 내부 생성이 아닌 클래스 단위의 선언에 대해서만 얘기하겠다.

#### Null의 할당 가능 여부
이 두 개의 가장 큰 차이점은 객체냐 아니냐이다. `Wrapper`는 객체이기 때문에 `null`을 가질 수 있고 원시타입은 그럴 수 없다.
의도적으로 `null`을 넣어주고 싶다면 `Wrapper`를 사용해야한다. 백엔드에서 DTO 클래스는 주로 컨트롤러에서 값 매핑, DB 조회 시 값 매핑 등에 사용되는데, 이 때 의도적으로 null이 필요할 수도 있다. 예를 들면 DB에는 컬럼에 대한 `Null`, `Not Null`이 있다. 만약 DB 조회 결과 A 컬럼이 `null`이고 DTO에서 A에 해당하는 타입이 원시타입이면 `NPE`가 발생한다.

#### 성능
`Wrapper`는 객체이기 때문에 Heap에 생성되고, GC의 관리를 받는다. 반면 원시타입은 Stack 에서만 사용되므로 현재 스코프가 종료되면 같이 제거된다. 
당연하게도 원시타입이 Wrapper보다 접근 속도, 메모리 측면에서 성능이 좋겠지만, 그렇게 유의미한 차이는 아닐 듯 하다.

#### 추가 기능 여부
원시타입은 객체가 아니기 떄문에 메서드라는 것이 없다. `Wrapper` 는 객체이므로 원시타입에 필요한 다양한 메서드가 지원된다. 

#### 제네릭 타입으로 사용 가능 여부
위에서도 언급했는데, 기본적으로 원시타입은 제네릭 타입으로 들어갈 수 없다. Wrapper는 객체이므로 가능하다. `int[]` 과 같은 원시타입의 배열도 Heap에서 관리하는 참조타입으로 간주되기 때문에 제네릭 타입으로 사용가능하다.
```java
List<int> list // X! 불가능
List<Integer> list // OK
List<int[]> list // OK
```

### 결론
DTO, 엔티티를 생성할 때, 항상 원시타입과 Wrapper 중 무엇을 사용할 지에 대해 고민했었는데 이제는 **Wrapper**를 주로 사용하려고 한다. 앞서 알아봤듯이 `Wrapper`를 사용하는 데에는 많은 이점이 있으며, 성능적인 부분에서도 신경써야할 만큼 크게 중요한 차이는 없어보인다. 객체 지향을 공부하다보면 좀 더 객체지향적인 코드를 위해 컬렉션을 일급컬렉션으로 `Wrapping`하고 여러 원시타입을 값 객체로  `Wrapping` 하기도 한다. 원시타입과 Wrapper 클래스 역시 이에 해당하는 좋은 Practice가 아닐까 생각한다.
마지막으로 하나 얘기할 것은, 이 포스팅은 클래스 단위에 선언된 변수(멤버 혹은 static)에 대한 얘기였다. 메서드 내부에서는 `원시타입과 Wrapper의 차이`에서 본 것 처럼 상황에 맞게 잘 사용하면 될 것이다.
## Reference
