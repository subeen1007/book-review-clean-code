# ch12. 창발성

## #12.1 창발적 설계로 깔끔한 코드를 구현하자

1. 창발?
    
    : 하위 계층(구성 요소)에는 없는 특성이나 행동이 상위 계층(전체 구조)에서 자발적으로 돌연히 출현하는 현상
    
    → 어떤 규칙과 원칙에 따라 설계를 하게 되면, 그것들이 모여 아주 좋은 거시적 설계가 된다는 말
    
2. 켄트 벡이 제시한 단순한 설계 규칙 4가지는 소프트웨어 설계 품질을 크게 높여준다
    
    <aside>
    💡 **단순한 설계 규칙 4가지(중요도순)**
    
    1. 모든 테스트를 실행한다. 
    2. 중복을 없앤다 
    3. 프로그래머 의도를 표현한다. 
    4. 클래스와 메서드 수를 최소로 줄인다. 
    
    ⇒ 위 4가지를 따르는 설계는 "**단순하다"**
    
    </aside>
    
    효과 → 코드 구조와 설계 파악 쉬워짐, SRP나 DIP 과 같은 원칙 적용이 쉬워짐, 우수한 설계의 창발성 촉진
    

## #12.2 단순한 설계 규칙 1 : 모든 테스트를 실행하라

- **테스트가 가능한 시스템**
    - 테스트를 철저히 거쳐 모든 테스트 케이스를 항상 통과하는 시스템
        
        (테스트가 불가능한 시스템 → 검증 불가. 출시 X)
        
    - 테스트가 가능한 시스템을 만들 때의 장점
        1. 설계 품질 ↑
        2. 크기 작고 목적 하나만 수행하는 클래스(SRP 준수하는 클래스) 나옴 → 테스트가 쉽다
    
    → 더 나은 설계 
    
- 테스트 케이스를 만들고 계속 돌리게 되면 객체 지향 방법론의 목표(낮은 결합도와 높은 응집력) 달성 가능
    - 결합도 ↑ → 테스트 케이스 작성 어렵
    - 테스트 케이스 많이 작성할 수록 DIP와 같은 원칙 적용하고 의존성 주입, 인터페이스, 추상화 등의 도구 사용해 결합도를 낮춤
    
    → 테스트 케이스를 작성하면 설계 품질이 높아진다.
    

## #12.3 단순한 설계 규칙 2~4 : 리팩터링

테스트 케이스 작성 완료 시 → 점진적으로 코드, 클래스 정리(코드 리펙터링)

- 리펙터링 단계
    - 소프트웨어 품질 높이는 기법 적용
    : 응집도 ↑, 결합도 ↓, 관심사 분리, 시스템 관심사 모듈로 나눔, 함수와 클래스 크기 ↓, 더 나은 이름 선택 등
    - 단순한 설계 규칙 적용
    : 중복 제거, 프로그래머의 의도 표현, 클래스와 메서드를 최소로 줄이는 단계

## #12.4 중복을 없애라

중복은 추가 작업, 추가 위혐, 불필요한 복잡도를 의미

1. **똑같은 코드**
    
    - 비슷한 코드를 더 비슷하게 고치면 리팩터링이 쉬워진다
    
2. **구현 중복**
    - 예시1
        - 중복 제거 전(각 메서드를 따로 구현)
            
            ```clike
            int size() {} // 개수 반환
            boolean isEmpty() {} // 부울 값 반환
            ```
            
        - 중복 제거 후
            
            ```clike
            boolean isEmpty() {
            	return 0 == size()
            }
            ```
            
    - 예시2
        - 중복 제거 전
            
            ```clike
            public void scaleToOneDimension(float desiredDimension, float imageDimension) {
              if (Math.abs(desiredDimension - imageDimension) < errorThreshold)
                return;
              float scalingFactor = desiredDimension / imageDimension;
              scalingFactor = (float)(Math.floor(scalingFactor * 100) * 0.01f);
              
              RenderedOpnewImage = ImageUtilities.getScaledImage(image, scalingFactor, scalingFactor);
              image.dispose();
              System.gc();
            	image = newImage;
            }
            
            public synchronized void rotate(int degrees) {
              RenderedOpnewImage = ImageUtilities.getRotatedImage(image, degrees);
              image.dispose();
              System.gc();
              image = newImage;
            }
            ```
            
        - 중복 제거 후
            
            ```clike
            public void scaleToOneDimension(float desiredDimension, float imageDimension) {
              if (Math.abs(desiredDimension - imageDimension) < errorThreshold)
                return;
              float scalingFactor = desiredDimension / imageDimension;
              scalingFactor = (float) Math.floor(scalingFactor * 10) * 0.01f);
              replaceImage(ImageUtilities.getScaledImage(image, scalingFactor, scalingFactor));
            }
            
            public synchronized void rotate(int degrees) {
              replaceImage(ImageUtilities.getRotatedImage(image, degrees));
            }
            
            private void replaceImage(RenderedOp newImage) {
              image.dispose();
              System.gc();
              image = newImage;
            }
            ```
            
            → 새 메서드를 만들어 중복을 줄였지만 클래스가 SRP를 위반하므로 replaceImage를 다른 클래스로 옮기자
            
            ⇒ 새 클래스의 가시성이 높아지고 재사용성을 늘려 시스템 복잡도를 줄여줌.
            
    - TEMPLATE METHOD 패턴 : 고차원 중복을 제거할 목적으로 자주 사용
        - 중복 제거 전
            
            ```clike
            /** 
            	최소 법정 일수 계산 코드만 제외하고 거의 동일한 두 메서드
            **/
            
            public class VacationPolicy {
              public void accrueUSDDivisionVacation() {
                // 지금까지 근무한 시간을 바탕으로 휴가 일수를 계산하는 코드
                // ...
                // 휴가 일수가 미국 최소 법정 일수를 만족하는지 확인하는 코드
                // ...
                // 휴가 일수를 급여 대장에 적용하는 코드
                // ...
              }
              
              public void accrueEUDivisionVacation() {
                // 지금까지 근무한 시간을 바탕으로 휴가 일수를 계산하는 코드
                // ...
                // 휴가 일수가 유럽연합 최소 법정 일수를 만족하는지 확인하는 코드
                // ...
                // 휴가 일수를 급여 대장에 적용하는 코드
                // ...
              }
            }
            ```
            
        - TEMPLATE METHOD 패턴 사용하여 중복 제거 후
            
            ```clike
            abstract public class VacationPolicy {
              public void accrueVacation() {
                caculateBseVacationHours();
                alterForLegalMinimums();
                applyToPayroll();
              }
              
              private void calculateBaseVacationHours() { /* ... */ };
              abstract protected void alterForLegalMinimums();
              private void applyToPayroll() { /* ... */ };
            }
            
            public class USVacationPolicy extends VacationPolicy {
              @Override protected void alterForLegalMinimums() {
                // 미국 최소 법정 일수를 사용한다.
              }
            }
            
            public class EUVacationPolicy extends VacationPolicy {
              @Override protected void alterForLegalMinimums() {
                // 유럽연합 최소 법정 일수를 사용한다.
              }
            }
            ```
            
            →  하위 클래스는 중복되지 않는 정보만 제공
            
        

## #12.5 표현하라

- 개발자의 의도를 분명히 표현해야 한다. 개발자가 명백하게 코드를 짤수록 다른 사람이 코드를 이해하기 쉬워짐 → 결함 ↓**,** 유지보수 비용 ↓
- 개발자의 의도를 분명히 표현한 코드를 짜는 방법
    1. **좋은 이름을 선택한다**
    2. **함수와 클래스 크기를 가능한 줄인다** 
        
        → 이름 짓기 쉽고, 구현 쉽고, 이해 쉽다.
        
    3. **표준 명칭을 사용한다.** 
        
        → ex) 디자인 패턴은 의사소통과 표현력 강화가 주요 목적이므로 클래스가 표준 패턴을 사용하여 구현된다면 클래스 이름에 패턴 이름을 넣으므로써 다른 개발자가 클래스 설계 의도를 이해하기 쉽게 만든다. 
        
    4. **단위 테스트 케이스를 꼼꼼히 작성한다.**
        
        → 테스트 케이스 = 예제로 보여주는 문서 
        
        ∴ 클래스 기능을 한 눈에 볼 수 있음
        

⇒ 가장 중요한 것은 노력!

## #12.6 클래스와 메서드 수를 최소로 줄여라

- 중복 제거, 의도 표현, SRP 준수하는 것에 너무 치중하면 X (∵ 너무 많은 클래스와 메서드가 생성될 수 있음)
    
    ∴ 함수와 클래스 수를 최대한 줄여라
    
- but, 테스트 케이스를 만들고 중복을 제거하고 의도를 표현하는 작업이 더 중요

## #12.7 핵심내용

<aside>
💡 **단순한 설계 규칙 4가지(중요도순)**

1. 모든 테스트를 실행한다. 
2. 중복을 없앤다 
3. 프로그래머 의도를 표현한다. 
4. 클래스와 메서드 수를 최소로 줄인다.

</aside>

## #Q&A

단순한 설계 규칙 4가지중 2가지 말해보기

- 정답
    
    1. 모든 테스트를 실행한다. 
    2. 중복을 없앤다 
    3. 프로그래머 의도를 표현한다. 
    4. 클래스와 메서드 수를 최소로 줄인다.
