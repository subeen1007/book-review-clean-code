# ch5. 형식맞추기

## #5.1 형식을 맞추는 목적

- "코드 형식은 의사소통의 일환이다."
오늘 구현한 코드의 가독성은 앞으로 바뀔 코드의 품질에 지대한 영향을 미친다.

## #5.2 적절한 행 길이를 유지하라

1. 신문 기사처럼 작성하라

: 표제 - 기사를 몇마디로 요약, 첫문단 - 전체 기사 내용 요약, 이후 - 세부사항이 나옴

- 소스파일의 첫 부분은 고차원 개념과 알고리즘을 설명
    
    
- 아래로 내려갈수록 의도를 세세히 묘사
- 마지막에는 가장 저 차원 함수와 세부내역이 나옴
2. 개념은 빈 행으로 분리하라

“생각 사이는 빈 행을 넣어 분리해야 마땅하다”

- 빈 행은 새로운 개념을 시작한다는 시각적 단서!

```java
package fitnesse.wikitext.widgets;

import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class BoldWidget extends ParentWidget {
	public static final String REGEXP = "'''.+?'''";
	private static final Pattern pattern = Pattern.compile("'''(.+?)'''",
		Pattern.MULTILINE + Pattern.DOTALL
	);

	public BoldWidget(ParentWidget parent, String text) throws Exception {
		super(parent);
		Matcher match = pattern.matcher(text);
		match.find();
		addChildWidgets(match.group(1));
	}

	public String render() throws Exception {
		StringBuffer html = new StringBuffer("<b>");
		html.append(childHtml()).append("</b>");

	    return html.toString();
	}
}
```

3. 세로 밀집도
- 서로 밀접한 코드행은 세로로 가까이 놓아야함

4. 수직거리
- 변수선언 : 변수는 사용하는 위치에 최대한 가까이 선언
- 인스턴스 변수 : 클래스 맨 처음에  선언
- 종속 함수 : 두 함수는 세로로 가까이 배치, 호출하는 함수를 호출되는 함수보다 먼저 배치
- 개념적 유사성 : 친화도가 높을수록 가까이 배치
    
    ** 개념적인 친화도 높은 예시
    
    - 한 함수가 다른 함수를 호출해 생기는 직접적 종속성
    - 변수와 그 변수를 사용하는 함수
    - 비슷한 동작을 수행
    
    ```java
    public class Assert {
      static public void assertTrue(String message, boolean condition) {
        if(!condition) {
          fail(message)
        }
      }
      
      static public void assertTure(boolean condition) {
        asertTrue(null, condition);
      }
      
      static public void assertFalse(String message, boolean condition) {
        assertTrue(message, !condition);
      } 
      
      static public void assertFalse(boolean condition) {
        assertFalse(null, condition)
      }
    }
    ```
    
5. 세로 순서
- 호출되는 함수를 호출하는 함수보다 나중에 배치한다

6. 가로 형식 맞추기
- 짧은 행이 바람직. 저자 개인적으론 120자 정도의 행길이를 제한

7. 가로 공백과 밀집도
- 가로로는 공백을 사용해 밀접한 개념과 느슨한 개념을 표현합니다.

```java
private void measureLine(String line) { 
    lineCount++;

	// 할당 연산자가 강조되어 왼쪽/오른쪽 요소가 나뉨
    int lineSize = line.length();
    totalChars += lineSize; 
	
	// 함수와 인수는 밀접하기에 공백을 넣지 않는다.
	// 인수들은 공백으로 분리(별개라는 점을 보여줌)
    lineWidthHistogram.addLine(lineSize, lineCount);
    recordWidestLine(lineSize);
}
```

8. 가로정렬

아래와 같은 정렬은 별로 유용하지 않습니다.

```java
publicclassFitNesseExpediterimplements ResponseSender{
private     Socket         socket;
private     InputStream    input;
private     OutputStream   output;
private     Reques         request;
private     Response       response;
private     FitNesseContex context;
protected   long           requestParsingTimeLimit;
private     long           requestProgress;
private     long           requestParsingDeadline;
private     boolean        hasError;
```

- 코드가 엉뚱한 부분을 강조해 진짜 의도가 가려짐
- 위 선언부를 읽다 보면 변수 유형은 무시하고 이름만 읽게 됨
- 할당 연산자는 보이지 않고 오른쪽 피연산자에 눈이 감
- 코드 정렬을 자동으로 해주는 도구가 위와 같은 정렬을 무시함

9. 들여쓰기

범위로 이뤄진 계층을 표현하기 위해 들여쓰기를 한다.

- 짧은 if/while문이나 함수에서 들여쓰기를 무시하지 않도록 한다.

```java
//안좋은 예
public class CommentWidget extends TextWidget{
	  public static final String REGEXP= "^#[^\r\n]*(?:(?:\r\n)|\n|\r)?";
	
    public CommentWidget(ParentWidget parent, String text){super(parent, text);}
	  public Stringrender()throws Exception{return "";}
}
```

```java
//좋은 예
public class CommentWidget extends TextWidget{
    public static final String REGEXP= "^#[^\r\n]*(?:(?:\r\n)|\n|\r)?";
 
    public CommentWidget(ParentWidget parent, String text){
	      super(parent, text);
	  }
	
    public Stringrender()throws Exception{
		    return "";
	  }
}
```

10. 가짜범위

빈 while/for문 사용지양.

- *만약 쓰게 된다면 세미콜론(;)을 새 행에 제대로 들여써서 넣어준다.*

```java
while(dis.read(buf, 0, readBufferSize)!=-1)
;
```

## #5.2 팀 규칙

팀은 한가지 규칙에 합의하고, 모든 팀원이 그 규칙을 따라야한다.

## +intelliJ 단축어

Ctrl + Alt + O : import문 형식정리

Ctrl + Alt + L : 소스코드 형식정리

## #Q & A

종속 함수관계 일때는 호출하는 함수와 후출되는 함수의 위치를 어떻게 해야 바람직?

- 정답
    
     두 함수는 세로로 가까이 배치, 호출하는 함수를 호출되는 함수보다 먼저 배치
