# ch10. 클래스

## #10.1 클래스 체계

- 클래스 안에서의 순서 :
    1. static public 상수
    2. static private 변수
    3. private instance 변수(public 변수가 필요한 경우는 거의 없음)
    4. public method
        - private method는 자신을 호출하는 public method 직후에 위치
- 캡슐화
    - *테스트를 위해 private method나 변수를 protected로 공개해야 할 경우가 있다. 이런 경우 공개하지 않을 방법을 충분히 생각한 이후에 캡슐화를 푸는 것이 좋다.*

## #10.2 클래스는 작아야 한다!

- 클래스를 만들 때 가장 중요한 규칙은 함수와 마찬가지로 **작은 크기**이다
- 클래스는 *맡은 책임* 를 기준으로 측정한다.
    - 함수는 물리적인 행 수로 측정
- 아래처럼 method가 몇개만 포함되면?
    - 이 또한 책임이 너무 많아서 좋지 않음
        
        ```java
        public class SuperDashboard extends JFrame implements MetaDataUser {
            public Component getLastFocusedComponent()
            public void setLastFocused(Component lastFocused)
            public int getMajorVersionNumber()
            public int getMinorVersionNumber()
            public int getBuildNumber() 
        }
        ```
        
1. 단일 책임 원칙
: SRP는 클래스나 모듈을 변경할 이유가 하나 뿐이어야 한다는 원칙
    - SuperDashboard 클래스를 리팩터링
    - 버전 정보를 다루는 메서드 3개를 따로 빼내 Version이라는 독자적인 클래스를 만들어 책임을 줄임
        
        ```java
        public class Version {
        	public int getMajorVersionNumber()
        	public int getMinorVersionNumber()
        	public int getBuildNumber()
        }
        ```
        
    - 큰 클래스 몇개가 아니라 작은 클래스 여럿으로 이뤄진 시스템이 더 바람직
2. 응집도
    - 클래스는 인스턴스 변수 수가 작아야한다.
    - 각 클래스 메서드는 클래스 인스턴스 변수를 하나 이상 사용해야 함
    - 일반적으로 메서드가 변수를 더 많이 사용할수록 메소드와 클래스는 응집도가 더 높다.
    - 모든 인스턴스 변수를 메서드마다 사용하는 클래스는 응집도가 가장 높습니다.
    
    응집도 높은 클래스 예시) stack
    
    ```java
    public class Stack {
        private int topOfStack = 0;
        List<Integer> elements = new LinkedList<Integer>();
    
        public int size() { 
            return topOfStack;
        }
    
        public void push(int element) { 
            topOfStack++; 
            elements.add(element);
        }
    
        public int pop() throws PoppedWhenEmpty { 
            if (topOfStack == 0)
                throw new PoppedWhenEmpty();
            int element = elements.get(--topOfStack); 
            elements.remove(topOfStack);
            return element;
        }
    }
    ```
    
    - 응집도 높이기 위해? 변수와 메서드 적절히 분리 → 새로운 클래스로 쪼개줌
    - 새로운 클래스로 쪼개야할 때? 몇몇 메서드만이 사용하는 인스턴스 변수가 아주 많아졌을때(응집력을 잃을때)

1. 응집도를 유지하면 작은 클래스 여럿이 나온다
    - 큰 함수를 여럿으로 쪼개기만 해도 클래스 수가 많아짐
        - 함수 쪼갬 → 분리된 함수에서 필요한 변수를 클래스 인스턴스 변수로 승격 → 클래스가 응집력을 잃음 → 클래스가 응집력을 잃으면 쪼개라! → 클래스 수 많아짐
    

## #10.3 변경하기 쉬운 클래스

- 깨끗한 시스템은 클래스를 체계적으로 정리해 변경에 수반하는 위험을 낮춘다
- 예시)
    - (before) 변경이 필요한 클래스 : 주어진 메타 자료로 적절한 SQL 문자열을 만드는 Sql 클래스
        - 새로운 SQL문을 지원하려면 반드시 Sql클래스에 추가해야 함
        - 기존 SQL문 수정시 Sql 클래스 수정해야함
        
        ⇒ 변경할 이유가 2가지 → SRP위반
        
        ```java
        public class Sql {
            public Sql(String table, Column[] columns)
            public String create()
            public String insert(Object[] fields)
            public String selectAll()
            public String findByKey(String keyColumn, String keyValue)
            public String select(Column column, String pattern)
            public String select(Criteria criteria)
            public String preparedInsert()
            private String columnList(Column[] columns)
            private String valuesList(Object[] fields, final Column[] columns)
        	  private String selectWithCriteria(String criteria)
            private String placeholderList(Column[] columns)
        }
        ```
        
    - (after) 재구성한 Sql 클래스 ⇒ SRP지원, OCP 지원
        - 공개 인터페이스를 각각 Sql 클래스에서 파생하는 클래스로 만듦.
        - 비공개 메서드는 해당하는 파생 클래스로 이동 ex) valueList
        - 모든 파생 클래스가 공통으로 사용하는 비공개 메서드는 Where과  ColumnList라는 두 유틸리티 클래스로 이동
        
        ```java
        abstract public class Sql {
            public Sql(String table, Column[] columns) 
            abstract public String generate();
        }
        public class CreateSql extends Sql {
            public CreateSql(String table, Column[] columns) 
            @Override public String generate()
        }
        
        public class SelectSql extends Sql {    //selectAll()
            public SelectSql(String table, Column[] columns) 
            @Override public String generate()
        }
        
        public class InsertSql extends Sql {
            public InsertSql(String table, Column[] columns, Object[] fields) 
            @Override public String generate()
        		//valuesList(Object[] fields, final Column[] columns)
            private String valuesList(Object[] fields, final Column[] columns)
        }
        
        public class SelectWithCriteriaSql extends Sql {   //select(Criteria criteria)
            public SelectWithCriteriaSql(String table, Column[] columns, Criteria criteria) 
            @Override public String generate()
        }
        
        public class SelectWithMatchSql extends Sql {   //select(Column column, String pattern)
            public SelectWithMatchSql(String table, Column[] columns, Column column, String pattern) 
            @Override public String generate()
        }
        
        public class FindByKeySql extends Sql 
        		public FindByKeySql(String table, Column[] columns, String keyColumn, String keyValue) 
            @Override public String generate()
        }
        
        public class PreparedInsertSql extends Sql {
            public PreparedInsertSql(String table, Column[] columns) 
            @Override public String generate() {
        		//placeholderList(Column[] columns)
            private String placeholderList(Column[] columns)
        }
        
        public class Where {   //selectWithCriteria(String criteria)
            public Where(String criteria) public String generate()
        }
        
        public class ColumnList {   //columnList(Column[] columns)
            public ColumnList(Column[] columns) public String generate()
        }
        ```
        
    
    ⇒ 새 기능을 수정하거나 기존 기능을 변경할 때 건드릴 코드가 최소인 시스템 구조가 바람직.(OCP)
    

1. 변경으로부터 격리
    - ***시스템의 결합도를 낮추면 유연성과 재사용성도 더욱 높아진다.***
    - 결합도가 낮다는 뜻? 각 시스템 요소가 다른 요소로부터 그리고 변경으로부터 잘 격리되어 있다는 의미.
    - 결합도를 최소로 줄이면 DIP(Dependency Inversion Principle)를 따르게 된다.
    (클래스가 상세한 구현이 아니라 추상화에 의존해야 한다는 원칙)
    - 예시
        - TokyoStockExchange라는 상세한 구현 클래스가 아닌 StockExchange Interface에 의존
        - StockExchange Interface는 주식 기호를 받아 현재 주식 가격을 반환한다는 추상적인 개념을 포함
        - 이와 같은 추상화로 실제 주가를 얻어오는 출처나 얻어오는 방식 등과 같은 구체적인 사실을 모두 숨긴다.
        
        ```java
        public interface StockExchange { 
            Money currentPrice(String symbol);
        }
         
        public Portfolio {
            private StockExchange exchange;   //고정된 주가를 반환
            public Portfolio(StockExchange exchange) {
                this.exchange = exchange; 
            }
            // ... 
        }
         
        public class PortfolioTest {
            private FixedStockExchangeStub exchange;
            private Portfolio portfolio;
            
        		//테스트 Stub Set
            @Before
            protected void setUp() throws Exception {
                exchange = new FixedStockExchangeStub(); 
                exchange.fix("MSFT", 100);
                portfolio = new Portfolio(exchange);
            }
         
        		//마이크로소프트 주식 5주 구입시 전체 포트폴리오 총계가 500불인지 확인
            @Test
            public void GivenFiveMSFTTotalShouldBe500() throws Exception {
                portfolio.add(5, "MSFT");
                Assert.assertEquals(500, portfolio.value()); 
            }
        }
        ```
        
    

## #Q&A

클래스는 작을수록 좋은데 언제가 새로운 클래스로 쪼개야할 때인가? 

- 정답
    - 몇몇 메서드만이 사용하는 인스턴스 변수가 아주 많아졌을때(응집력을 잃을때)
    - 맡은 책임이 너무 많을때
