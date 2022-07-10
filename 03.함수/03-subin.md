# ch3. 함수

## #3.1 작게 만들어라!

"함수를 만드는 첫째 규칙은 '작게!'다. 함수를 만드는 둘째 규칙은 '더 작게!'다."

```java
//함수를 작게 만들어라

//설정페이지, 해제페이지를 테스트 페이지에 넣은 후 해당 테스트 페이지를 html로 렌더링
public static String renderPageWithSetupsAndTeardowns(
        PageData pageData, boolean isSuite) throws Exception {
    if (isTestPage(pageDatCa))
        includeSetupAndTeardownPages(pageDataf isSuite);
    return pageData.getHtml();
}
```

1. 블록과 들여쓰기

if문/else문/while문 등에 들어가는 블록은 한줄이여야 한다

## #3.2 한 가지만 해라!

"함수는 한 가지를 해야 한다. 그 한 가지를 잘 해야한다. 그 한 가지만을 해야 한다."

한가지 동작여부 판단 기준

- 저장된 함수 이름 아래에서 추상화 수준이 하나인 단계만 수행해야 한다.
- 의미 있는 다른 이름으로 다른 함수를 추출할 수 있다면 그 함수는 여러 작업을 하고 있는 것이다.

## #3.3 함수 당 추상화 수준은 하나로!

"함수가 확실히 '한가지' 작업만 하려면 함수 내 모든 문장의 추상화 수준이 동일해야 한다."

1. 위에서 아래로 코드 읽기 : 내려가기 규칙

- 위에서 아래로 이야기처럼 읽혀야 좋다.
- 함수 추상화 수준이 한 번에 한 단계씩 낮아진다.

## #3.4 Switch문

문제점 : 스위치문은 여러개의 동작을 관리하기 때문에 작게 만들기 어렵다.

```java
public Money calculatePay(Employee e)
        throws InvalidEmplyeeType {
    switch (e.type) {
        case COMMISSIONED:
            return caleulateCommissionedPay(e);
        case HOURLY:
            return calculateHourlyPay(e);
        case SALARIED:
            return calculateSalariedPay(e);
        default:
            throw new InvalidEmployeeType(e.type);
    }
}
```

예시의 문제점 : 

1. 함수가 너무 길다.

2. 한가지의 작업만 수행하지 않는다.

3. SRP(단일 책임 원칙)를 위반한다. (코드의 변경이유가 여럿이기 때문)

4. OCP(개방 폐쇄 원칙) 위반한다. (새 직원의 유형이 추가 될때마다 코드를 변경하기 때문)

해결책 : 스위치문을 저차원 클래스에 숨기고 절대 반복하지 않는방법이 있다. 

- 장황한 switch 문의 반복은 추상 팩토리 & 다형성 객체 생성 코드로 개선

예) 목록 3-5. Employee and Factory

```java
public abstract class Employee {
    public abstract boolean isPayday();
    public abstract Money calculatePay();
    public abstract void deliverPay();
}
---------------------------------------
public interface EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType;
}
----------------------------------------
public class EmployeeFactoryImpl implements EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
        switch(r.type) {
            case COMMISSIONED:
                return new CommissionedEmployee(r.type);
            case HOURLY:
                return new HourlyEmployee(r.type);
            case SALARIED:
                return new SalariedEmployee(r.type);
            default:
                return new InvalidEmployeeType(r.type);
        }
    }
} 
```

## #3.5 서술적인 이름을 사용하라!

"코드를 읽으면서 짐작했던 기능을 각 루틴이 그대로 수행한다면 깨끗한 코드라 불러도 되겠다."

## #3.6 함수인수

"이상적인 인수 개수는 0개(무항)다. 다음은 1개(단항)고, 다음은 2개(이항)다."

- 인수는 개념을 이해하기 어렵게 만든다.
- 별로 중요하지 않은 세부사항임에도 불구하고, 발견할 때마다 의미를 해석해야 하기 때문이다. 
ex) StringBuffer
- 인수가 생기면 모든 경우의 수를 검증하는 테스트 케이스를 작성하기 복잡해진다.

1. 많이 쓰는 단항 형식
- 인수에 질문을 던지는 경우
ex) `boolean fileExists("MyFile")`  //파일이 존재한가?
- 인수를 뭔가로 변환해 결과를 반환하는 경우
ex) `InputStream fileOpen("MyFile")`  //String형의 파일 이름을 InputStream으로 변환함
- 이벤트 (입력 인수로 시스템 상태를 바꾸는 경우)
ex) `void passwordAttemptFailedNtimes(int attempts)` //암호 시도 실패 N회

2. 플래그 인수는 금지
- 각 플래그 값에 따라 별도의 함수를 작성하라.
ex) `render(boolean isSuite)` → `renderForSuite()` & `renderForSingleTest()`

3. 이항 함수가 적절한 경우
- 인수 2개가 한 값을 표현하고, 자연적인 순서가 있는 경우
좋은 ex) `Point p = new Point(0,0)`
나쁜 ex) `assertEquals(expected, actual)` //자연적 순서X → expected, actual의 순서를 기억해야함

4. 삼항 함수가 적절한 경우
- 부동소수점 비교
ex) `assertEquals(1.0, amount, .001)` 
//*double expected, double actual, double delta (델타-두 숫자가 여전히 동일한 것으로 간주되는 예상과 실제 사이의 최대 델타.)*

5. 인수 객체

인수가 2~3개가 필요하다면 독자적인 클래스 변수로 선언할 가능성을 짚어본다.

```java
Circle makeCircle(double x, double y, double radius);
Circle makeCirele(Point center, double radius);
```

6. 동사와 키워드 
- 단항 함수는 함수와 인수가 동사/명사 쌍을 이뤄야 한다.
예) `write(name)`
- 함수 이름에 인수에 관한 키워드를 추가하라.
예) `write(name)` → `writeField(name)`//name이 field임을 표기예) `assertEquals()` → `assertExpectedEqualsActual(expected, actual)` //인수의 순서를 표기

## #3.7 부수 효과를 일으키지 마라!

- 예상치 못한 부수 효과는 시간적인 결합, 순서 종속성을 초래하게 된다.
- 함수는 한 가지 기능만 하도록 작성하라.

```java
public class UserValidator {
    private Cryptographer cryptographer;
    public boolean checkPassword(St ring userName, String password) {
        User user = UserGateway.findByName(userName);
        if (user != User.NULL) {
            String codedPhrase = user.getPhraseEncodedByPasswᄋrd();
            String phrase = cryptographer.decrypt(codedPhrase, password);
            if ("Valid Password".equals(phrase)) {
                Session.initialize(); //부수효과
                return true;
            }
        }
        return false;
    }
}
//사용자는 checkPassword를 통해 비밀번호를 확인하는데 기존 세션정보를 지워버릴 위험이 생긴다.
```

## #3.8 명령과 조회를 분리하라!

함수는 뭔가를 수행하거나, 뭔가에 답하거나 둘 중 하나만 해야 한다.

나쁜 예)

```java
//이름이 attribute인 속성을 찾아 value로 설정한 후 성공하면 true, 실패하면 false를 반환
boolean set(String attribute, String value)

if (set("username", "myname")) //명령&조회
  ...
```

좋은 예)

```java
if (attributeExists("username")) { //조회
  setAttribute("username", "myname");  //명령
  ...
}
```

## #3.9 오류 코드보다 예외를 사용하라!

- 오류 코드를 반환하는 방식은 여러 단계로 중첩되는 코드를 야기한다. ex)`if deletePage(page) == E_OK`
- 오류 코드를 반환하는 방식은 오류 코드를 곧바로 처리해야 한다는 문제가 발생한다.

### Try/Catch 블록

- `Try/Catch` 블록을 별도 함수로 뽑아내고, `try`문 안에서 실제 '작업' 메소드를 호출한다.
- 실제 '작업'을 하는 코드에서는 `thorws Exception`하여 모든 예외 처리를 한 곳에서 처리한다.

## #3.10 반복하지 마라!

"어쩌면 중복은 소프트웨어에서 모든 악의 근원이다."

- 코드 길이가 늘어난다.
- 알고리즘이 변하면 중복된 코드 모두 수정해야 한다.
- 오류가 발생할 확률도 몇 배로 높다.

## #3.11 구조적 프로그래밍

- 모든 함수와 함수 내 모든 블록에 입구와 출구가 하나만 존재해야 한다.

*루프 안에서 break나 continue를 사용해선 안되며 goto는 절대로 사용해선 안된다.

## #3.12 함수를 어떻게 짜죠?

step 1) 처음에는 길고 복잡한 코드

step 2)테스트하는 단위 테스트 케이스 만듬

step 3)코드를 줄임. 함수를 만듬. 이름을 바꿈. 중복을 제거

step 4)메서드를 줄임. 순서를 바꿈. 때로는 전체 클래스를 쪼갬. 이와중에도 코드는 항상 단위 테스트를 통과

>> 최종적으로 clean한 함수 완성

## #최종정리

- 작게 만들어라!
- 한 가지만 해라! : 함수는 한 가지를 해야 한다
- 함수 당 추상화 수준은 하나로! : 내려가기 규칙
- Switch문 : 장황한 switch 문의 반복은 추상 팩토리 & 다형성 객체 생성 코드로 개선
- 함수인수 : 이상적인 인수 개수는 0개, 인수가 2-3개 필요하다면 인수를 독자적인 클래스 변수로 선언한다.
- 부수 효과를 일으키지 마라!
- 명령과 조회를 분리하라!
- 오류 코드보다 예외를 사용하라!
- 반복하지 마라!
- 구조적 프로그래밍 : 모든 함수와 함수 내 모든 블록에 입구와 출구가 하나만 존재해야 한다.
- 함수를 어떻게 짜죠? : 1) 복잡한코드 -> 단위 테스트 -> 코드줄임,함수만듬,이름바꿈,중복제거 -> 메서드줄임,순서바꿈,전체 클래스 쪼갬 -> 깨끗한 함수 완성

## #문제

아래와 같은 부분이 가진 문제점이 뭔가요??

```java
//사용자가 checkPassword를 통해 비밀번호를 확인하는 함수
public class UserValidator {
    private Cryptographer cryptographer;
    public boolean checkPassword(St ring userName, String password) {
        User user = UserGateway.findByName(userName);
        if (user != User.NULL) {
            String codedPhrase = user.getPhraseEncodedByPasswᄋrd();
            String phrase = cryptographer.decrypt(codedPhrase, password);
            if ("Valid Password".equals(phrase)) {
                Session.initialize(); //세션초기화 > 기존 세션정보를 지워버릴 위험이 생긴다.
                return true;
            }
        }
        return false;
    }
}
```

- 정답
    
    부수효과를 일으키면 안된다(함수는 한 가지 기능만 하도록 작성하라.)
