![Java](https://raw.githubusercontent.com/oasis791/blog-posting/8600ab949c7250edd10f1bf7d4ea15c9338fa197/Java/java.png)

## 개요

유튜브를 보다가 [Short-Circuit Evaluation에 관한 재미있는 영상](https://youtu.be/5dm7GVsZJLw?si=E-UQlOP9aDC7GAya)을 보았다. 코드를 짜는 개발자라면 명백히 생각하고, 고려할만 한 부분이라고 생각되어 포스팅하기로 결심했다.

## 논리 연산 OR과 AND

모두들 아는 내용일테지만, 논리 연산 OR과 AND의 연산 결과는 다음과 같다.

**[OR 연산]**

| A     | B     | A OR B |
| ----- | ----- | ------ |
| false | false | false  |
| false | true  | true   |
| true  | false | true   |
| true  | true  | true   |



**[AND 연산]**

| A     | B     | A AND B |
| ----- | ----- | ------- |
| false | false | false   |
| false | true  | false   |
| true  | false | false   |
| true  | true  | true    |

## Java의 논리 연산자

Java에서는 **eager operator**와 **short-circuit operator**가 있다.

**[eager operator]**

- OR은 `|` AND는 `&`로 나타낸다.
- bit operator와 같은 형태를 띄고 있지만, 각 조건 결과가 boolean형이면, eager operator로 쓰인다.
- short-circuit operator와의 가장 큰 차이점은, **첫 번째 조건과 상관 없이, 두 번째 조건을 무조건 실행한다는 점이다.**

**[short-circuit operation]**

- OR은 `||` AND는 `&&`로 나타낸다.
- eager operator와는 다르게 **첫 번째 조건의 결과에 따라, 두 번째 조건을 실행하지 않을 수 있다.**

=> 이를 **Short-circuit Evaluation** 이라고 한다.

## Short-circuit Evaluation 예시

다음 코드를 보자.

```java
public class Test {
    static boolean A() {
        System.out.println("A");
        return true;
    }

    static boolean B() {
        System.out.println("B");
        return false;
    }
}
```

해당 코드는 단순히 true를 반환해주는 메소드 A와 false를 반환해주는 메소드 B이다.

각 메소드가 호출되는지의 여부를 확인하기 위해 각 메소드의 이름을 출력해주고 있다.

여기서, main 메소드 안에 `A()`와 `B()`를 short-circuit operator를 이용하여 출력값들을 비교할 것이다.

**[예제 1 - true && false]**

```java
public static void main(String[] args) {
  System.out.println(A() && B());
}
```

이렇게, `A() && B()` 의 결과값을 확인하면 어떻게 나올까?

```java
A
B
false
```

예상 했듯이, A와 B가 순서대로 출력되고, true와 false의 AND연산 결과 값인 false가 출력 될 것이다.

여기서, `A() && B()`가 아닌, `B() && A()`로 출력하면 어떤 결과가 나올까?

**[예제 2 - false && true]**

```java
public static void main(String[] args) {
  System.out.println(B() && A());
}
```

```java
B
false
```

**앞선 결과와는 다르게 A는 출력되지 않고, B와 false만 출력되는 것을 확인할 수 있다.**

왜 이런 결과가 나올까?

논리 연산 AND는 앞선 조건이 true이면 뒤 조건이 false이냐 true이냐에 따라 값이 false또는 true로 바뀔 수 있지만, **앞선 조건이 false이면 뒤 조건이 어떠한 값이든 상관없이 결과 값이 false이다.**

**즉, Short-circuit evaluation는 일부 프로그래밍 언어의 일부 부울 연산자의 의미론으로, 첫 번째 인수가 식의 값을 결정하기에 충분하지 않은 경우에만 두 번째 인수가 실행되거나 평가되는 것을 의미한다.**

OR 연산도 마찬가지이다.

**[예제 3 - false || true]**

```java
public static void main(String[] args) {
  System.out.println(B() || A());
}
```

```java
B
A
true
```

**[예제 4 - true || false]**

```java
public static void main(String[] args) {
  System.out.println(A() || B());
}
```

```java
A
true
```

OR 연산도 마찬가지로 앞선 조건이 true이면 뒤 조건이 false이냐 true이냐에 관계 없이 결과값이 true로 고정되므로, 뒤 조건을 검사 할 필요가 없다.

**[예제 5 - OR연산 예외 상황]**

```java
public class Test {
    public static void main(String[] args) {
        System.out.println(A() || B());
    }

    static boolean A() {
        System.out.println("A");
        return true;
    }

    static boolean B() {
        System.out.println("B");
      // 치명적인 ArithmeticException 발생!!
        int error = 4 / 0;
        return false;
    }
}
```

```java
A
true
```

Java의 대표적인 RuntimeError인 **ArithmeticException**이 발생하도록 정수를 0으로 나누는 연산을 추가하였으나, 무리 없이 true가 반환되는 것을 확인할 수 있다.

## 결론

이처럼 Short-circuit Evaluation을 이용하면 선행 된 조건에 따라 후행 조건의 연산을 통째로 skip 할 수 있어, 성능상 이점을 볼 수 있다.

따라서, short-circuit operator를 사용할 땐, **비용이 높은 코드는 나중에 평가 되도록 작성해야 성능상 이점을 볼 수 있다.**

그러나, **치명적인 RuntimeError가 발생할 수 있는 코드는, 먼저 평가 되게 하거나, eager operator를 이용하여 양 쪽의 조건을 모두 평가되게 작성해야 안정적인 코드가 될 것이다.**