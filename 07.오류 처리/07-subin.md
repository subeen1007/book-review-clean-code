# ch7. 오류처리

## #7.1 오류 코드보다 예외를 사용하라

나쁜 예) 오류코드 사용예

- 함수를 호출한 즉시 오류를 확인해야함
- 오류확인을 잊기쉬움

```java
public class DeviceController {
	...
	public void sendShutDown() {
		DeviceHandle handle = getHandle(DEV1);
		// 디바이스 상태를 점검한댜.
		if (handle != DeviceHandle.INVALID) {
			// 레코드 필드에 디바이스 상태를 저장한다.
			retrieveDeviceRecord(handle);
			// 디바이스가 일시정지 상태가 아니라면 종료한다.
			if (record.getStatus() != DEVICE_SUSPENDED) {
				pauseDevice(handle);
				clearDeviceWorkQueue(handle);
				closeDevice(handle);
			} else {
				logger.log("Device suspended. Unable to shut down");
			}
		} else {
			logger.log("Invalid handle for: " + DEV1.toString());
		}
	}
	...
}
```

좋은 예) 예외사용

- 호출자 코드 깔끔

```java
public class DeviceController {
	...
	public void sendShutDown() {
		try {
			tryToShutDown();
		} catch (DeviceShutDownError e) {
			logger.log(e);
		}
	}

	private void tryToShutDown() throws DeviceShutDownError {
		DeviceHandle handle = getHandle(DEV1);
		DeviceRecord record = retrieveDeviceRecord(handle);
		pauseDevice(handle); 
		clearDeviceWorkQueue(handle); 
		closeDevice(handle);
	}

	private DeviceHandle getHandle(DeviceID id) {
		...
		throw new DeviceShutDownError("Invalid handle for: " + id.toString());
		...
	}
	...
}
```

## #7.2 Try-Catch-Finally문부터 작성하라

try-catch-finally 문에서 try 블록에 들어가는 코드를 실행하면 어느 시점에서든 실행이 중단된 후 catch 블록으로 넘어갈 수 있습니다.

- try 블록에서 무슨 일이 생기든지 catch 블록은 프로그램 상태를 일관성 있게 유지해야 함
- try-catch-finally 문을 시작으로 코드를 짜면 호출자가 기대하는 상태를 정의하기 쉬워짐

### *TDD 방식으로 메소드 구현*

1. *단위 테스트* 를 만든다.
    
    ```java
    @Test**(**expected **=** StorageException**.**class**)
    public** **void** **retrieveSectionShouldThrowOnInvalidFileName()** **{**
    	sectionStore**.**retrieveSection**(**"invalid - file"**);
    }**
    ```
    
2. 단위 테스트에 맞춰 *코드를 구현* 한다.
    
    ```java
    *// 실제로 구현할 때까지 비어 있는 더미를 반환한다.*
    **public** List**<**RecordedGrip**>** **retrieveSection(**String sectionName**)** **{**
    	****return** **new** ArrayList**<**RecordedGrip**>();
    }**
    ```
    
    예외가 발생하지 않기 때문에 *단위 테스트에서 실패* 한다.
    
3. 파일 접근을 시도하도록 구현한다.
    
    ```java
    **public** List**<**RecordedGrip**>** **retrieveSection(**String sectionName**)** **{
    	try** **{**
    		FileInputStream stream **=** **new** FileInputStream**(**sectionName**);
    	}** **catch** **(**Exception e**)** **{
    		throw** **new** **StorageException(**"retrieval error"**,** e**);
    	}
    	return** **new** ArrayList**<**RecordedGrip**>();
    }**
    ```
    
    테스트가 성공할 것이다.
    
4. *리펙터링*이 가능해졌다.
    
    ```java
    **public** List**<**RecordedGrip**>** **retrieveSection(**String sectionName**)** **{
    	try** **{**
    		FileInputStream stream **=** **new** FileInputStream**(**sectionName**);**
    		stream**.**close**();
    	}** **catch** **(**FileNotFoundException e**)** **{ //예외 리펙터링
    		throw** **new** **StorageException(**"retrieval error"**,** e**);
    	}
    	return** **new** ArrayList**<**RecordedGrip**>();
    }**
    ```
    

→ TDD방식으로 구현시 자연스럽게 try블록의 트랜잭션 범위부터 구현하게 되므로 범위 내에서 트랜잭션 본질을 유지하기 쉬워짐 

## #7.3 미확인(unchecked) 예외를 사용하라

1. *확인/미확인(checked/*unchecked*) 예외란?*

*checked 예외*  : 컴파일 단계에서 확인되며 반드시 처리해야 하는 예외입니다.

- IOException
- SQLException

*Unchecked 예외*  : 실행 단계에서 확인되며 명시적인 처리를 강제하지는 않는 예외입니다.

- NullPointerException
- IllegalArgumentException
- IndexOutOfBoundException
- SystemException
2. 확인된(checked) 예외 사용시 문제점

: 비용이 든다 > 확인된 예외는 OCP(Open Closed Principle)을 위반

- 하위단계(호출되는)에서 코드변경시 상위단계(호출하는) 연쇄적인 수정필요

## #7.4 예외에 의미를 제공하라

- 오류가 발생한 *원인과 위치를 찾기 쉽도록* 호출 스택만으로는 부족한 정보를 충분히 덧붙여야 함.
    - 오류 메시지에 정보를 담음
    - 실패한 연산 이름, 실패 유형 언급

## #7.5 호출자를 고려해 예외 클래스를 정의하라

“프로그래머에게 가장 중요한 관심사는 **오류를 잡아내는 방법**이 되어야한다.”

1. 아래 코드는 외부 라이브러리를 호출하고, 모든 예외를 호출자가 잡아내고 있습니다.

```java
ACMEPort port = new ACMEPort(12);

 try {
     port.open();
 } catch (DeviceResponseException e) {
     reportPortError(e);
     logger.log("Device response exception", e);
 } catch (ATM1212UnlockedException e) {
     reportPortError(e);
     logger.log("Unlock exception", e);
 } catch (GMXError e) {
     reportPortError(e);
     logger.log("Device response exception");
 } finally {
     ...
 }

```

2. 호출 라이브러리 API를 감싸 한가지 예외 유형을 반환하는 방식으로 단순화

위 경우는 예외에 대응하는 방식이 예외 유형과 무관하게 거의 동일함

```java
LocalPort port = new LocalPort(12);
 try {
     port.open();
 } catch (PortDeviceFailure e) {
     reportError(e);
     logger.log(e.getMessage(), e);
 } finally {
     ...
 }
```

```java
public class LocalPort {
     private ACMEPort innerPort;

     public LocalPort(int portNumber) {
         innerPort = new ACMEPort(portNumber);
     }

     public void open() {
         try {
             innerPort.open();
         } catch (DeviceResponseException e) {
             throw new PortDeviceFailure(e);
         } catch (ATM1212UnlockedException e) {
             throw new PortDeviceFailure(e);
         } catch (GMXError e) {
             throw new PortDeviceFailure(e);
         }
     }
     ...
 }
```

외부 API를 감싸면 장점

- 에러 처리가 간결해짐
- 외부 라이브러리와 프로그램 사이의 의존성이 크게 줄어듦
- 프로그램 테스트가 쉬워짐
- 외부 API 설계 방식에 의존하지 않아도 됨

## #7.6 정상 흐름을 정의하라

**중단이 적합하지 않은 때도 있다. "특수 사례 패턴"으로 클래스를 만들거나 객체를 조작해 특수사례를 처리한다.**

→ 클라이언트 코드가 예외적인 상황을 처리할 필요가 없어진다

특수 사례 객체를 반환하는 *특수 사례 패턴*의 예시)

1. 리팩토링 *before)* 총계를 계산하는 코드

```java
/**
* 설명 : ExpenseReportDAO를 고쳐 언제나 MealExpense 객체를 반환하게 한다.
* 청구한 식비가 없다면 일일 기본 식비를 반환하는 MealExpense 객체를 반환한다.
*/
try {
     MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
     m_total += expenses.getTotal();
 } catch(MealExpencesNotFound e) {
     m_total += getMealPerDiem();
 }
```

2. 리팩토링 *after) 클래스나 객체가 예외적인 상황을 캡슐화 해서 처리한다*

```java
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();
```

```java
public class PerDiemMealExpenses implements MealExpenses {
     public int getTotal() {
         // 기본값으로 일일 기본 식비를 반환한다.
         // (예외가 아닌)
     }
 }
```

## #7.7 Null을 반환하지 마라

메소드에서 null 반환하고 싶다면 :

- 예외를 던짐
- 특수 사례 객체를 반환

사용하려는 API가 null을 반환한다면 :

- 감싸기 메소드 구현 → 예외던짐
- 특수 사례 객체를 반환

null반환의 참사 : 

```java
public void registerItem(Item item) {
	if (item != null) { //null체크1
		//peristentStore가 null일시 NullPointerException발생
		ItemRegistry registry = peristentStore.getItemRegistry();
		if (registry != null) { //null체크2
			Item existing = registry.getItem(item.getID());
			if (existing.getBillingPeriod().hasRetailOwner()) {
				existing.register(item);
			}
		}
	}
}
```

null대신 Collections.emptyList() 반환 :

```java
// bad) null반환함..
List<Employee> employees = getEmployees();
if(employees != null) {
	for(Employee e : employees) {
		totalPay += e.getPay();
	}
}

// good) null대신 빈값반환
List<Employee> employees = getEmployees();
for(Employee e : employees) {
	totalPay += e.getPay();
}

public List<Employee> getEmployees() {
	if (..직원이 없다면..)
		return Collections.emptyList();
}
```

→ null반환X로 NullPointerException 발생 가능성 줄이자!

## #7.8 null을 전달하지 마라

메소드로 null을 전달하는 코드는 피해라!

## #7.9 최종정리

1. 오류 코드보다 예외를 사용하라
2. Try-Catch-Finally문부터 작성하라 : *TDD 방식으로 메소드 구현* 추천
3. 미확인(unchecked) 예외를 사용하라
4. 예외에 의미를 제공하라 : 오류 메시지에 정보, 실패한 연산 이름, 실패 유형 담기
5. 호출자를 고려해 예외 클래스를 정의하라 : 외부 API의 오류 감싸기
6. 정상 흐름을 정의하라 : 특수 사례 패턴
7. null을 반환하지 마라
- 예외를 던짐
- 특수 사례 객체를 반환
8. null을 전달하지 마라 : null인수X

## #Q&A

null대신 빈 빈리스트값을 반환하기 위해 무슨 함수를 사용하는 것이 좋을까요?

- 정답
