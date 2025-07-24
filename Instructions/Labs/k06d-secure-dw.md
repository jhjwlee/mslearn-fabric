

# 핸즈온 랩: Microsoft Fabric 데이터 웨어하우스 보안

Microsoft Fabric의 권한과 세분화된 SQL 권한은 함께 작동하여 Warehouse 액세스 및 사용자 권한을 제어합니다. 이 실습에서는 세분화된 권한, 열 수준 보안(Column-Level Security), 행 수준 보안(Row-Level Security) 및 동적 데이터 마스킹(Dynamic Data Masking)을 사용하여 데이터를 보호합니다.

> **참고**: 이 실습의 연습을 완료하려면 두 명의 사용자가 필요합니다. 한 사용자는 Workspace Admin 역할을, 다른 사용자는 Workspace Viewer 역할을 할당받아야 합니다. Workspace에 역할을 할당하려면 [Workspace에 대한 액세스 권한 부여](https://learn.microsoft.com/fabric/get-started/give-access-workspaces)를 참조하세요.

이 실습을 완료하는 데 약 **45**분이 소요됩니다.

## Workspace 만들기

Fabric에서 데이터 작업을 시작하기 전에 Fabric 평가판이 활성화된 Workspace를 만들어야 합니다.

1.  웹 브라우저에서 [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com/home?experience=fabric) `https://app.fabric.microsoft.com/home?experience=fabric`로 이동하여 Fabric 자격 증명으로 로그인합니다.
2.  왼쪽 메뉴 모음에서 **Workspaces** 아이콘( izgled: &#128455;)을 선택합니다.
3.  원하는 이름으로 새 Workspace를 만듭니다. **Advanced** 섹션에서 Fabric 용량(*Trial*, *Premium*, 또는 *Fabric*)을 포함하는 라이선스 모드를 선택해야 합니다.
4.  새 Workspace가 열리면 처음에는 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷](./Images/new-empty-workspace.png)

> **참고**: Workspace를 생성하면 자동으로 Workspace Admin 역할의 멤버가 됩니다. 이 연습에서 구성된 기능을 테스트하기 위해 환경의 두 번째 사용자를 Workspace Viewer 역할에 추가할 수 있습니다. 이는 Workspace 내에서 **Manage Access**를 선택한 다음 **Add people or groups**를 선택하여 수행할 수 있습니다. 이렇게 하면 두 번째 사용자가 Workspace 콘텐츠를 볼 수 있습니다.

## Data Warehouse 만들기

다음으로, 생성한 Workspace에 데이터 웨어하우스를 만듭니다.

1.  왼쪽 메뉴 모음에서 **Create**를 선택합니다. *New* 페이지의 *Data Warehouse* 섹션에서 **Warehouse**를 선택하고, 원하는 고유한 이름을 지정합니다.
2.  약 1분 후, 새로운 빈 웨어하우스가 생성됩니다.

    ![새로운 빈 데이터 웨어하우스 스크린샷](./Images/new-empty-data-warehouse.png)

## 테이블의 열에 동적 데이터 마스킹(Dynamic Data Masking) 규칙 적용하기

**개념 설명: Dynamic Data Masking (DDM)**
동적 데이터 마스킹은 중요한 데이터를 쿼리 결과에서 숨기거나 대체하여 권한 없는 사용자가 볼 수 없도록 하는 보안 기능입니다. 실제 데이터베이스에 저장된 데이터는 변경되지 않으며, 쿼리가 실행될 때 실시간으로 데이터가 마스킹됩니다. 예를 들어, 신용카드 번호의 마지막 4자리를 제외하고 모두 'X'로 표시하거나, 이메일 주소를 형식에 맞게 숨길 수 있습니다. 이는 민감한 데이터의 노출을 최소화하면서도 데이터 분석은 가능하게 하는 강력한 도구입니다.

1.  웨어하우스에서 **T-SQL** 타일을 선택하고, 다음 T-SQL 문을 사용하여 테이블을 생성하고 데이터를 삽입 및 조회합니다.

    ```T-SQL
    CREATE TABLE dbo.Customers
    (   
        CustomerID INT NOT NULL,   
        FirstName varchar(50) MASKED WITH (FUNCTION = 'partial(1,"XXXXXXX",0)') NULL,     
        LastName varchar(50) NOT NULL,     
        Phone varchar(20) MASKED WITH (FUNCTION = 'default()') NULL,     
        Email varchar(50) MASKED WITH (FUNCTION = 'email()') NULL   
    );
   
    INSERT dbo.Customers (CustomerID, FirstName, LastName, Phone, Email) VALUES
    (29485,'Catherine','Abel','555-555-5555','catherine0@adventure-works.com'),
    (29486,'Kim','Abercrombie','444-444-4444','kim2@adventure-works.com'),
    (29489,'Frances','Adams','333-333-3333','frances0@adventure-works.com');
   
    SELECT * FROM dbo.Customers;
    ```
    **코드 설명**:
    - `MASKED WITH (FUNCTION = 'partial(1,"XXXXXXX",0)')`: **사용자 지정 마스크**. `FirstName` 열에 적용됩니다. 문자열의 첫 `1`개 문자를 보여주고, 그 뒤에 `XXXXXXX`를 붙인 후, 마지막 `0`개 문자를 보여줍니다. (결과: CXXXXXXX)
    - `MASKED WITH (FUNCTION = 'default()')`: **기본 마스크**. `Phone` 열에 적용됩니다. 숫자 데이터는 `0`으로, 문자열 데이터는 `xxxx`로 마스킹합니다.
    - `MASKED WITH (FUNCTION = 'email()')`: **이메일 마스크**. `Email` 열에 적용됩니다. 이메일 주소의 첫 글자만 남기고 `XXX@XXX.com` 형식으로 마스킹합니다. (결과: cXXX@XXX.com)

2.  **&#9655; Run** 버튼을 사용하여 SQL 스크립트를 실행합니다.
3.  **Explorer** 창에서 **Schemas** > **dbo** > **Tables**를 확장하고 **Customers** 테이블이 생성되었는지 확인합니다. `SELECT` 문은 마스킹되지 않은 데이터를 반환하는데, 이는 Workspace 생성자로서 **Workspace Admin** 역할의 멤버이기 때문이며, 이 역할은 기본적으로 마스킹되지 않은 데이터를 볼 수 있는 권한을 가집니다.

4.  **핸즈온의 의미**: 이제 보안 규칙이 실제로 어떻게 동작하는지 확인하기 위해, 권한이 제한된 사용자로 접속하여 결과를 비교해 봅니다. **Workspace Viewer** 역할의 테스트 사용자로 연결하고 다음 T-SQL 문을 실행합니다.

    ```T-SQL
    SELECT * FROM dbo.Customers;
    ```
    테스트 사용자는 `UNMASK` 권한을 부여받지 않았기 때문에 `FirstName`, `Phone`, `Email` 열에 대해 마스킹된 데이터가 반환됩니다.

5.  다시 본인(Workspace Admin)으로 다시 연결하고, 다음 T-SQL을 실행하여 테스트 사용자에 대한 데이터 마스킹을 해제합니다. `<username>@<your_domain>.com` 부분을 **Workspace Viewer** 역할의 테스트 사용자 이름으로 바꾸세요.

    ```T-SQL
    GRANT UNMASK ON dbo.Customers TO [<username>@<your_domain>.com];
    ```
    **코드 설명**: `GRANT UNMASK`는 특정 사용자에게 마스킹 규칙을 우회하고 원본 데이터를 볼 수 있는 권한을 부여하는 명령입니다.

6.  다시 테스트 사용자로 연결하여 다음 T-SQL 문을 실행합니다.

    ```T-SQL
    SELECT * FROM dbo.Customers;
    ```
    이제 테스트 사용자가 `UNMASK` 권한을 부여받았으므로 데이터가 마스킹되지 않은 상태로 반환됩니다.

## 행 수준 보안(Row-Level Security) 적용하기

**개념 설명: Row-Level Security (RLS)**
행 수준 보안(RLS)은 쿼리를 실행하는 사용자의 ID나 역할에 따라 테이블의 특정 행에 대한 액세스를 제한하는 데 사용됩니다. 예를 들어, 영업 담당자는 자신이 담당하는 고객의 데이터 행만 볼 수 있고, 다른 담당자의 데이터는 볼 수 없도록 설정할 수 있습니다. 이는 동일한 `SELECT * FROM ...` 쿼리를 실행하더라도, 누가 실행하느냐에 따라 서로 다른 결과(행의 수)를 보게 만드는 강력한 보안 기능입니다. RLS는 보안 정책(Security Policy)과 보안 조건자(Security Predicate) 함수를 통해 구현됩니다.

1.  이전 실습에서 만든 웨어하우스에서 **New SQL Query**를 선택합니다.
2.  테이블을 생성하고 데이터를 삽입합니다. 나중에 행 수준 보안을 테스트할 수 있도록, `<username1>@your_domain.com`은 테스트 사용자 이름으로, `<username2>@your_domain.com`은 본인의 사용자 이름으로 바꾸세요.

    ```T-SQL
    CREATE TABLE dbo.Sales  
    (OrderID INT, SalesRep VARCHAR(60), Product VARCHAR(10), Quantity INT);
    
    INSERT dbo.Sales (OrderID, SalesRep, Product, Quantity) VALUES
    (1, '<username1>@<your_domain>.com', 'Valve', 5),   
    (2, '<username1>@<your_domain>.com', 'Wheel', 2),   
    (3, '<username1>@<your_domain>.com', 'Valve', 4),  
    (4, '<username2>@<your_domain>.com', 'Bracket', 2),   
    (5, '<username2>@<your_domain>.com', 'Wheel', 5),   
    (6, '<username2>@<your_domain>.com', 'Seat', 5);  
    
    SELECT * FROM dbo.Sales;  
    ```

3.  스크립트를 실행하여 **Sales** 테이블을 생성하고 데이터를 확인합니다.
4.  새로운 스키마, 함수로 정의된 보안 조건자(security predicate), 그리고 보안 정책(security policy)을 생성합니다.

    ```T-SQL
    -- RLS 객체를 담을 별도의 스키마 생성
    CREATE SCHEMA rls;
    GO
   
    /* 인라인 테이블 반환 함수로 보안 조건자 생성.
    이 함수는 SalesRep 열의 값이 쿼리를 실행하는 사용자와 같을 때 1(true)을 반환.
    즉, 행에 접근할 수 있음을 의미. */
    CREATE FUNCTION rls.fn_securitypredicate(@SalesRep AS VARCHAR(60)) 
        RETURNS TABLE  
    WITH SCHEMABINDING  
    AS  
        RETURN SELECT 1 AS fn_securitypredicate_result   
    WHERE @SalesRep = USER_NAME();
    GO   

    /* Sales 테이블에 쿼리가 실행될 때마다 함수를 호출하고 강제하는 보안 정책 생성.
    이 정책은 읽기 작업(SELECT, UPDATE, DELETE)에 사용할 수 있는 행을 조용히 필터링. */
    CREATE SECURITY POLICY SalesFilter  
    ADD FILTER PREDICATE rls.fn_securitypredicate(SalesRep)   
    ON dbo.Sales  
    WITH (STATE = ON);
    GO
    ```
    **코드 설명**:
    - `CREATE FUNCTION rls.fn_securitypredicate(...)`: "규칙 검사기" 역할을 하는 함수입니다. `WHERE @SalesRep = USER_NAME()` 부분이 핵심으로, `SalesRep` 열의 값과 현재 접속한 사용자 이름(`USER_NAME()`)이 일치하는지 비교합니다. 일치하면 `1`을 반환하여 해당 행을 보여주도록 허용합니다.
    - `CREATE SECURITY POLICY SalesFilter`: "규칙 집행관" 역할을 하는 정책입니다.
    - `ADD FILTER PREDICATE ... ON dbo.Sales`: `dbo.Sales` 테이블에 대해 `rls.fn_securitypredicate` 함수를 필터 규칙으로 적용하라고 지시합니다. 이제부터 `dbo.Sales` 테이블을 조회하는 모든 `SELECT` 문은 이 필터를 거치게 됩니다.

5.  스크립트를 실행하고 **Explorer**에서 `rls` 스키마 아래에 함수가 생성되었는지 확인합니다.

6.  **핸즈온의 의미**: `INSERT` 문에서 `<username1>@<your_domain>.com`으로 지정했던 테스트 사용자로 Fabric에 로그인합니다. 다음 T-SQL을 실행하여 현재 로그인한 사용자가 맞는지 확인합니다.
    ```T-SQL
    SELECT USER_NAME();
    ```

7.  **Sales** 테이블을 쿼리하여 행 수준 보안이 예상대로 작동하는지 확인합니다. 현재 로그인한 사용자에 대해 정의된 보안 조건자를 충족하는 데이터만 보여야 합니다. 즉, `SalesRep` 열이 현재 사용자 이름과 일치하는 3개의 행만 결과로 나타납니다.
    ```T-SQL
    SELECT * FROM dbo.Sales;
    ```

## 열 수준 보안(Column-Level Security) 구현하기

**개념 설명: Column-Level Security (CLS)**
열 수준 보안은 특정 사용자가 테이블의 특정 열에 접근할 수 없도록 지정하는 기능입니다. 이는 `GRANT` 또는 `DENY` 문을 사용하여 간단하게 구현할 수 있습니다. 예를 들어, 모든 사용자가 주문 ID와 고객 ID는 볼 수 있지만, 신용카드 번호 열은 특정 재무팀 역할만 볼 수 있도록 제한할 수 있습니다.

1.  웨어하우스에서 새 SQL 쿼리를 엽니다.
2.  테이블을 생성하고 데이터를 삽입합니다.

    ```T-SQL
    CREATE TABLE dbo.Orders
    (OrderID INT, CustomerID INT, CreditCard VARCHAR(20));   

    INSERT dbo.Orders (OrderID, CustomerID, CreditCard) VALUES
    (1234, 5678, '111111111111111'),
    (2341, 6785, '222222222222222'),
    (3412, 7856, '333333333333333');   

    SELECT * FROM dbo.Orders;
    ```
3.  테이블의 특정 열에 대한 조회 권한을 거부(DENY)합니다. 다음 T-SQL 문은 `<username>@<your_domain>.com` 사용자가 `Orders` 테이블의 `CreditCard` 열을 보지 못하게 합니다. `<username>@<your_domain>.com` 부분을 Workspace에서 **Viewer** 권한을 가진 테스트 사용자 이름으로 바꾸세요.

     ```T-SQL
    DENY SELECT ON dbo.Orders (CreditCard) TO [<username>@<your_domain>.com];
     ```

4.  **핸즈온의 의미**: 조회 권한을 거부한 테스트 사용자로 Fabric에 로그인하여 열 수준 보안을 테스트합니다.
5.  `Orders` 테이블을 쿼리하여 열 수준 보안이 예상대로 작동하는지 확인합니다. 다음 쿼리는 `CreditCard` 열에 대한 접근이 거부되었기 때문에 오류를 반환합니다.

    ```T-SQL
    SELECT * FROM dbo.Orders;
    ```
    하지만, `CreditCard` 열을 제외하고 `OrderID`와 `CustomerID` 필드만 선택하면 쿼리는 성공적으로 실행됩니다. 이를 통해 특정 열에 대한 접근만 선택적으로 차단되었음을 확인할 수 있습니다.
    ```T-SQL
    SELECT OrderID, CustomerID from dbo.Orders
    ```

## T-SQL을 사용하여 세분화된 SQL 권한 구성하기

**개념 설명: Granular SQL Permissions**
Fabric은 Workspace 수준과 Item 수준에서 데이터 접근을 제어하는 권한 모델을 가지고 있습니다. Fabric 웨어하우스의 보안 개체(securable)에 대해 사용자가 수행할 수 있는 작업을 더 세밀하게 제어해야 할 경우, 표준 SQL 데이터 제어 언어(DCL)인 `GRANT`, `DENY`, `REVOKE` 명령을 사용할 수 있습니다. 이는 전통적인 데이터베이스 보안 관리 방식과 동일합니다.

1.  웨어하우스에서 새 SQL 쿼리를 엽니다.
2.  저장 프로시저와 테이블을 생성합니다. 그런 다음 프로시저를 실행하고 테이블을 쿼리합니다.

     ```T-SQL
    CREATE PROCEDURE dbo.sp_PrintMessage
    AS
    PRINT 'Hello World.';
    GO   
    CREATE TABLE dbo.Parts(PartID INT, PartName VARCHAR(25));
   
    INSERT dbo.Parts (PartID, PartName) VALUES (1234, 'Wheel'), (5678, 'Seat');
    GO
   
    EXEC dbo.sp_PrintMessage;
    GO   
    SELECT * FROM dbo.Parts
     ```

3.  다음으로 **Workspace Viewer** 역할의 멤버인 사용자에게 테이블에 대한 `SELECT` 권한을 `DENY`하고, 동일한 사용자에게 프로시저에 대한 `EXECUTE` 권한을 `GRANT`합니다. `<username>@<your_domain>.com` 부분을 테스트 사용자 이름으로 바꾸세요.

     ```T-SQL
    DENY SELECT on dbo.Parts to [<username>@<your_domain>.com];

    GRANT EXECUTE on dbo.sp_PrintMessage to [<username>@<your_domain>.com];
     ```

4.  **핸즈온의 의미**: `DENY` 및 `GRANT` 문에서 지정한 테스트 사용자로 Fabric에 로그인합니다. 그런 다음 저장 프로시저를 실행하고 테이블을 쿼리하여 적용된 세분화된 권한을 테스트합니다.
     ```T-SQL
    EXEC dbo.sp_PrintMessage;
    GO
   
    SELECT * FROM dbo.Parts;
     ```
    **결과 분석:**
    - `EXEC dbo.sp_PrintMessage;`는 성공적으로 실행됩니다. 왜냐하면 `EXECUTE` 권한을 명시적으로 부여받았기 때문입니다.
    - `SELECT * FROM dbo.Parts;`는 실패합니다. 왜냐하면 테이블에 대한 `SELECT` 권한이 명시적으로 거부되었기 때문입니다.
    이를 통해 권한이 각 데이터베이스 객체(테이블, 저장 프로시저 등)에 따라 개별적으로 적용됨을 명확히 알 수 있습니다.

## 리소스 정리

이 실습에서는 테이블의 열에 동적 데이터 마스킹 규칙을 적용하고, 행 수준 보안을 적용하고, 열 수준 보안을 구현하고, T-SQL을 사용하여 SQL 세분화된 권한을 구성했습니다.

1.  왼쪽 탐색 모음에서 Workspace 아이콘을 선택하여 포함된 모든 항목을 봅니다.
2.  상단 툴바의 메뉴에서 **Workspace settings**를 선택합니다.
3.  **General** 섹션에서 **Remove this workspace**를 선택합니다.
