# 핸즈온 랩: Microsoft Fabric Lakehouse 만들기

전통적으로 대규모 데이터 분석 솔루션은 **data warehouse**를 중심으로 구축되었습니다. 데이터 웨어하우스에서는 데이터가 관계형 테이블에 저장되고 SQL을 사용하여 쿼리됩니다. 그러나 "빅 데이터"(방대한 *양(volumes)*, *다양성(variety)*, *빠른 속도(velocity)*의 새로운 데이터 자산이 특징)의 성장과 저비용 스토리지 및 클라우드 규모의 분산 컴퓨팅 기술의 발전은 **data lake**라는 분석 데이터 저장소에 대한 대안적인 접근 방식을 이끌어냈습니다. 데이터 레이크에서는 데이터가 고정된 스키마 없이 파일로 저장됩니다.

최근 데이터 엔지니어와 분석가들은 이 두 가지 접근 방식의 장점을 결합한 **data lakehouse**를 통해 이점을 얻고자 합니다. 데이터 레이크하우스에서는 데이터가 데이터 레이크의 파일에 저장되고, 관계형 스키마가 메타데이터 계층으로 적용되어 전통적인 SQL 구문을 사용하여 쿼리할 수 있습니다.

Microsoft Fabric에서 **lakehouse**는 *OneLake* 저장소(Azure Data Lake Store Gen2 기반)에 확장성이 뛰어난 파일 스토리지를 제공하며, 오픈 소스 *Delta Lake* 테이블 형식을 기반으로 하는 테이블 및 뷰와 같은 관계형 객체를 위한 메타스토어를 제공합니다. Delta Lake를 사용하면 레이크하우스에 테이블 스키마를 정의하여 SQL로 쿼리할 수 있습니다.

이 실습을 완료하는 데 약 **30**분이 소요됩니다.

> **참고**: 이 실습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

---

## Workspace 만들기

Fabric에서 데이터 작업을 시작하기 전에 Fabric 평가판이 활성화된 Workspace(작업 영역)를 만들어야 합니다. 이 Workspace는 여러분의 프로젝트와 관련된 모든 Fabric 항목(Lakehouse, Notebook, 보고서 등)을 담는 컨테이너 역할을 합니다.

1.  웹 브라우저에서 [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com/home?experience=fabric) `https://app.fabric.microsoft.com/home?experience=fabric`로 이동하여 Fabric 자격 증명으로 로그인합니다.
2.  왼쪽 메뉴 모음에서 **Workspaces** 아이콘( izgled: &#128455;)을 선택합니다.
3.  원하는 이름으로 새 Workspace를 만듭니다. **Advanced** 섹션에서 Fabric 용량(*Trial*, *Premium*, 또는 *Fabric*)을 포함하는 라이선스 모드를 선택해야 합니다.
4.  새 Workspace가 열리면 처음에는 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷](./Images/new-workspace.png)

## Lakehouse 만들기

이제 Workspace가 준비되었으니, 데이터 파일을 위한 데이터 레이크하우스를 만들 차례입니다. Lakehouse는 파일 저장(Data Lake)과 테이블 구조(Data Warehouse)의 장점을 결합한 통합 데이터 플랫폼입니다.

1.  왼쪽 메뉴 모음에서 **Create**를 선택합니다. *New* 페이지의 *Data Engineering* 섹션에서 **Lakehouse**를 선택하고, 원하는 고유한 이름을 지정합니다.

    > **참고**: **Create** 옵션이 사이드바에 고정되어 있지 않다면, 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    약 1분 후, 새로운 Lakehouse가 생성됩니다.

    ![새로운 레이크하우스 스크린샷](./Images/new-lakehouse.png)

2.  새로운 Lakehouse를 살펴보면, 왼쪽의 **Lakehouse explorer** 창을 통해 Lakehouse의 테이블과 파일을 탐색할 수 있습니다.
    *   **Tables** 폴더: SQL 구문을 사용하여 쿼리할 수 있는 테이블이 포함됩니다. Microsoft Fabric Lakehouse의 테이블은 Apache Spark에서 흔히 사용되는 오픈 소스 *Delta Lake* 파일 형식을 기반으로 합니다.
    *   **Files** 폴더: 관리형 델타 테이블과 연결되지 않은 데이터 파일들이 Lakehouse의 OneLake 저장소에 포함됩니다. 이 폴더에 외부 데이터를 참조하는 *shortcuts*를 만들 수도 있습니다.

    현재 Lakehouse에는 테이블이나 파일이 없습니다.

## 파일 업로드

Fabric은 외부 소스에서 데이터를 복사하는 파이프라인과 Power Query 기반의 시각적 도구를 사용하여 정의할 수 있는 데이터 흐름(Gen 2)을 포함하여 Lakehouse로 데이터를 로드하는 여러 방법을 제공합니다. 하지만 소량의 데이터를 수집하는 가장 간단한 방법 중 하나는 로컬 컴퓨터에서 파일이나 폴더를 직접 업로드하는 것입니다.

**핸즈온의 의미**: 이 단계는 가장 직관적이고 기본적인 데이터 적재 방법을 직접 경험하게 하여, Fabric Lakehouse가 일반 파일 시스템처럼 쉽게 데이터를 담을 수 있다는 점을 이해시키는 데 목적이 있습니다.

1.  [`https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/sales.csv`](https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/sales.csv)에서 **sales.csv** 파일을 다운로드하여 로컬 컴퓨터에 저장합니다.
    > **참고**: 파일을 다운로드하려면 브라우저에서 새 탭을 열고 URL을 붙여넣습니다. 데이터가 포함된 페이지에서 마우스 오른쪽 버튼을 클릭하고 **다른 이름으로 저장**을 선택하여 CSV 파일로 저장하세요.
2.  Lakehouse가 열려 있는 웹 브라우저 탭으로 돌아가, **Explorer** 창의 **Files** 폴더에 있는 **...** 메뉴에서 **New subfolder**를 선택하고 **data**라는 이름의 하위 폴더를 만듭니다.
3.  새로운 **data** 폴더의 **...** 메뉴에서 **Upload**를 선택한 다음 **Upload files**를 선택하여 로컬 컴퓨터에서 **sales.csv** 파일을 업로드합니다.
4.  파일이 업로드된 후, **Files/data** 폴더를 선택하여 **sales.csv** 파일이 업로드되었는지 확인합니다.

    ![레이크하우스에 업로드된 sales.csv 파일 스크린샷](./Images/uploaded-sales-file.png)

5.  **sales.csv** 파일을 선택하여 내용 미리보기를 확인합니다.

## Shortcuts 탐색하기

많은 경우 Lakehouse에서 작업해야 할 데이터가 다른 위치에 저장되어 있을 수 있습니다. 데이터를 OneLake 저장소로 수집하는 여러 방법이 있지만, 또 다른 옵션은 **shortcut**을 만드는 것입니다. Shortcut을 사용하면 데이터를 복사하는 데 따르는 오버헤드와 데이터 불일치 위험 없이 외부 소스 데이터를 분석 솔루션에 포함할 수 있습니다.

1.  **Files** 폴더의 **...** 메뉴에서 **New shortcut**을 선택합니다.
2.  Shortcut에 사용할 수 있는 데이터 소스 유형을 살펴본 후, shortcut을 만들지 않고 **New shortcut** 대화 상자를 닫습니다.

## 파일 데이터를 테이블로 로드하기

업로드한 판매 데이터는 파일 형태이며, 데이터 분석가와 엔지니어는 Apache Spark 코드를 사용하여 직접 작업할 수 있습니다. 하지만 많은 시나리오에서 SQL을 사용하여 쿼리할 수 있도록 파일의 데이터를 테이블로 로드하는 것이 더 편리합니다.

**핸즈온의 의미**: 이 단계에서는 '스키마 온 리드(Schema-on-Read)' 개념을 체험합니다. 구조화되지 않은 CSV 파일을 업로드한 후, Fabric이 이 파일을 분석하여 스키마를 추론하고 구조화된 Delta Lake 테이블로 변환하는 과정을 직접 수행해봅니다.

1.  **Explorer** 창에서 **Files/data** 폴더를 선택하여 포함된 **sales.csv** 파일을 확인합니다.
2.  **sales.csv** 파일의 **...** 메뉴에서 **Load to Tables** > **New table**을 선택합니다.
3.  **Load to table** 대화 상자에서 테이블 이름을 **sales**로 설정하고 로드 작업을 확인합니다. 테이블이 생성되고 로드될 때까지 기다립니다.

    > **팁**: **sales** 테이블이 자동으로 나타나지 않으면, **Tables** 폴더의 **...** 메뉴에서 **Refresh**를 선택하세요.

4.  **Explorer** 창에서 생성된 **sales** 테이블을 선택하여 데이터를 확인합니다.

    ![테이블 미리보기 스크린샷](./Images/table-preview.png)

5.  **sales** 테이블의 **...** 메뉴에서 **View files**를 선택하여 이 테이블의 기반이 되는 실제 파일들을 확인합니다.

    ![델타 테이블 파일 스크린샷](./Images/delta-table-files.png)

    **개념 설명**: Delta 테이블의 파일은 고효율 압축 형식인 *Parquet* 형식으로 저장됩니다. 또한, 테이블에 적용된 모든 트랜잭션(삽입, 업데이트, 삭제 등)의 세부 정보가 기록되는 **_delta_log**라는 하위 폴더가 포함됩니다. 이 로그 덕분에 데이터 버전 관리(Time Travel)와 ACID 트랜잭션이 가능해집니다.


## SQL을 사용하여 테이블 쿼리하기

Lakehouse를 만들고 그 안에 테이블을 정의하면, SQL `SELECT` 문을 사용하여 테이블을 쿼리할 수 있는 **SQL analytics endpoint**가 자동으로 생성됩니다.

**핸즈온의 의미**: 이 단계는 Lakehouse의 핵심 가치 중 하나인 '단일 데이터 복사본(One Copy of Data)'으로 다양한 작업을 수행할 수 있음을 보여줍니다. 파일로 업로드하고 테이블로 변환한 동일한 데이터를, 이번에는 친숙한 SQL을 사용하여 즉시 분석해봄으로써 데이터 전문가(SQL 분석가)와 데이터 엔지니어 모두에게 원활한 작업 환경을 제공한다는 점을 체험합니다.

1.  Lakehouse 페이지의 오른쪽 상단에서 모드를 **Lakehouse**에서 **SQL analytics endpoint**로 전환합니다. 잠시 후 Lakehouse의 테이블을 쿼리할 수 있는 시각적 인터페이스가 열립니다.

2.  **New SQL query** 버튼을 사용하여 새 쿼리 편집기를 열고 다음 SQL 쿼리를 입력합니다.

    ```sql
    SELECT
        Item,
        SUM(Quantity * UnitPrice) AS Revenue
    FROM
        sales
    GROUP BY
        Item
    ORDER BY
        Revenue DESC;
    ```
    
    **코드 설명**: 이 SQL 쿼리는 `sales` 테이블에서 각 제품별 총 매출을 계산하고, 매출이 높은 순서대로 정렬하여 보여줍니다.

    *   `SELECT Item, SUM(Quantity * UnitPrice) AS Revenue`: 제품(`Item`)과, 수량(`Quantity`)과 단가(`UnitPrice`)를 곱한 값의 합계를 `Revenue`라는 이름의 열로 계산하여 선택합니다.
    *   `FROM sales`: `sales` 테이블에서 데이터를 가져옵니다.
    *   `GROUP BY Item`: 결과를 제품(`Item`)별로 묶어 집계(SUM)합니다.
    *   `ORDER BY Revenue DESC`: 계산된 `Revenue`를 기준으로 내림차순(가장 큰 값이 먼저 오도록)으로 결과를 정렬합니다.

3.  **&#9655; Run** 버튼을 사용하여 쿼리를 실행하고 결과를 확인합니다. 각 제품별 총 매출이 표시되어야 합니다.

    ![SQL 쿼리와 결과 스크린샷](./Images/sql-query.png)

---



## 시각적 쿼리 만들기

많은 데이터 전문가가 SQL에 익숙하지만, Power BI 경험이 있는 데이터 분석가는 Power Query 기술을 적용하여 시각적 쿼리를 만들 수 있습니다.

**핸즈온의 의미**: 이 실습을 통해 코드를 작성하지 않고도 마우스 클릭만으로 복잡한 데이터 변환(여기서는 그룹화 및 집계)을 수행할 수 있음을 경험합니다. 이는 SQL에 익숙하지 않은 사용자도 Fabric Lakehouse의 데이터를 쉽게 분석하고 가공할 수 있다는 것을 보여줍니다.

1.  도구 모음에서 **New SQL query** 옵션을 확장하고 **New visual query**를 선택합니다.
2.  **sales** 테이블을 새로 열린 시각적 쿼리 편집기 창으로 드래그하여 아래와 같이 Power Query를 만듭니다.

    ![시각적 쿼리 스크린샷](./Images/visual-query.png)

3.  **Manage columns** 메뉴에서 **Choose columns**를 선택합니다. 그런 다음 **SalesOrderNumber**와 **SalesOrderLineNumber** 열만 선택합니다.

    ![열 선택 대화 상자 스크린샷](./Images/choose-columns.png)

4.  **Transform** 메뉴에서 **Group by**를 선택합니다. 다음 **Basic** 설정을 사용하여 데이터를 그룹화합니다.
    *   **Group by**: `SalesOrderNumber`
    *   **New column name**: `LineItems`
    *   **Operation**: `Count distinct values`
    *   **Column**: `SalesOrderLineNumber`

    완료되면 시각적 쿼리 아래의 결과 창에 각 판매 주문의 라인 아이템 수가 표시됩니다.

    ![결과가 포함된 시각적 쿼리 스크린샷](./Images/visual-query-results.png)

## 리소스 정리

이 실습에서는 Lakehouse를 만들고 데이터를 가져왔습니다. Lakehouse가 OneLake 데이터 저장소에 저장된 파일과 테이블로 구성되어 있음을 확인했습니다. 관리형 테이블은 SQL을 사용하여 쿼리할 수 있으며, 데이터 시각화를 지원하기 위해 기본 의미 체계 모델에 포함됩니다.

Lakehouse 탐색을 마쳤다면 이 실습을 위해 만든 Workspace를 삭제할 수 있습니다.

1.  왼쪽 막대에서 Workspace 아이콘을 선택하여 포함된 모든 항목을 봅니다.
2.  도구 모음에서 **Workspace settings**를 선택합니다.
3.  **General** 섹션에서 **Remove this workspace**를 선택합니다.
