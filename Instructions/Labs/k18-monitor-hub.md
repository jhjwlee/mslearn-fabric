# 모니터링 허브에서 Fabric 활동 모니터링하기

Microsoft Fabric의 `monitoring hub`(모니터링 허브)는 활동을 모니터링할 수 있는 중앙 집중식 공간을 제공합니다. 모니터링 허브를 사용하여 볼 수 있는 권한이 있는 항목과 관련된 이벤트를 검토할 수 있습니다.

이 실습을 완료하는 데는 약 **30**분이 소요됩니다.

> **Note**: 이 실습을 완료하려면 [Microsoft Fabric 테넌트](https://learn.microsoft.com/fabric/get-started/fabric-trial)에 대한 액세스 권한이 필요합니다.

---
### **개념 설명: Monitoring Hub**

`Monitoring hub`는 Microsoft Fabric 내에서 발생하는 모든 활동을 추적하고 관리하기 위한 중앙 관제 센터입니다. 사용자는 이 허브를 통해 자신이 권한을 가진 모든 Workspace에서 실행되는 다양한 작업들의 상태를 한눈에 파악할 수 있습니다.

**주요 기능 및 특징:**
*   **중앙 집중식 모니터링**: Dataflow 실행, Notebook 작업, 파이프라인 실행 등 다양한 Fabric 항목의 활동을 한 곳에서 볼 수 있습니다.
*   **상태 추적**: 각 활동의 현재 상태(예: `In-progress`, `Succeeded`, `Failed`)를 실시간으로 확인할 수 있어, 작업이 정상적으로 진행되고 있는지, 혹은 문제가 발생했는지 신속하게 파악할 수 있습니다.
*   **실행 기록(Historical Runs)**: 특정 항목의 과거 실행 기록을 조회할 수 있어, 문제 발생 시 원인을 추적하거나 성능 변화를 분석하는 데 유용합니다.
*   **상세 정보 보기**: 각 실행에 대한 상세 정보(시작 시간, 종료 시간, 기간, 실행 주체 등)를 확인할 수 있습니다.
*   **필터링 및 사용자 정의**: 수많은 활동 중에서 원하는 항목만 쉽게 찾아볼 수 있도록 상태, 항목 유형, 시간 등으로 필터링하는 기능을 제공합니다.

`Monitoring hub`는 Fabric 환경의 운영 상태를 건강하게 유지하고, 문제가 발생했을 때 신속하게 대응하기 위한 필수적인 도구입니다.

---

## Workspace 생성

Fabric에서 데이터를 다루기 전에, Fabric 용량이 활성화된 테넌트에서 Workspace를 생성해야 합니다.

1.  브라우저에서 [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com/home?experience=fabric-developer) (`https://app.fabric.microsoft.com/home?experience=fabric-developer`)로 이동하여 Fabric 자격 증명으로 로그인합니다.
2.  왼쪽 메뉴 바에서 **Workspaces** (아이콘 모양 &#128455;)를 선택합니다.
3.  원하는 이름으로 새 Workspace를 만들고, **Advanced** 섹션에서 Fabric 용량을 포함하는 라이선스 모드(*Trial*, *Premium*, 또는 *Fabric*)를 선택합니다.
4.  새 Workspace가 열리면 비어 있어야 합니다.

    ![Fabric의 비어 있는 Workspace 스크린샷](./Images/new-workspace.png)

## Lakehouse 생성

이제 Workspace가 있으므로 데이터용 데이터 레이크하우스를 만들 차례입니다.

1.  왼쪽 메뉴 바에서 **Create**를 선택합니다. *New* 페이지의 *Data Engineering* 섹션에서 **Lakehouse**를 선택합니다. 원하는 고유한 이름을 지정합니다.

    >**Note**: **Create** 옵션이 사이드바에 고정되어 있지 않으면 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    약 1분 후, 새로운 Lakehouse가 생성됩니다.

    ![새로운 lakehouse 스크린샷](./Images/new-lakehouse.png)

1.  새로운 Lakehouse를 보고, 왼쪽의 **Lakehouse explorer** 창을 통해 Lakehouse의 테이블과 파일을 탐색할 수 있음을 확인합니다.

    현재 Lakehouse에는 테이블이나 파일이 없습니다.

## Dataflow 생성 및 모니터링

---
### **개념 설명: Dataflow Gen2**

`Dataflow Gen2`는 Microsoft Fabric에서 데이터를 수집, 변환, 로드(ETL)하는 데 사용되는 강력한 도구입니다. 이는 Power Query를 기반으로 하므로, 수백 개의 데이터 소스(CSV 파일, 데이터베이스, 웹 서비스 등)에 연결하고, 코드를 거의 또는 전혀 작성하지 않고도 그래픽 인터페이스를 통해 데이터를 정리하고 변형할 수 있습니다.

Dataflow를 통해 수행된 데이터 변환 작업은 재사용이 가능하며, 그 결과를 Lakehouse나 Warehouse 같은 다른 Fabric 항목에 로드하여 분석에 활용할 수 있습니다. 이 실습에서는 Dataflow를 사용하여 웹에 있는 CSV 파일로부터 제품 데이터를 가져와 Lakehouse 테이블로 로드합니다.
---

Microsoft Fabric에서는 `Dataflow (Gen2)`를 사용하여 다양한 소스에서 데이터를 수집할 수 있습니다. 이 실습에서는 데이터플로우를 사용하여 CSV 파일에서 데이터를 가져와 Lakehouse의 테이블로 로드합니다.

1.  Lakehouse의 **Home** 페이지에서 **Get data** 메뉴의 **New Dataflow Gen2**를 선택합니다.
2.  새로운 Dataflow의 이름을 `Get Product Data`로 지정하고 **Create**를 선택합니다.

    ![새로운 dataflow 스크린샷](./Images/new-data-flow.png)

3.  Dataflow 디자이너에서 **Import from a Text/CSV file**을 선택합니다. 그런 다음 Get Data 마법사를 완료하여 익명 인증을 사용하여 `https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/products.csv`에 연결하여 데이터 연결을 생성합니다. 마법사를 완료하면 다음과 같이 데이터플로우 디자이너에 데이터 미리보기가 표시됩니다.

    ![dataflow 쿼리 스크린샷](./Images/data-flow-query.png)

4.  Dataflow를 게시(Publish)합니다.
5.  왼쪽 탐색 바에서 **Monitor**를 선택하여 모니터링 허브를 보고, 데이터플로우가 진행 중(in-progress)인지 확인합니다(만약 보이지 않는다면 보일 때까지 뷰를 새로고침하세요).

    ![진행 중인 dataflow가 있는 모니터링 허브 스크린샷](./Images/monitor-dataflow.png)

6.  몇 초간 기다린 다음, Dataflow의 상태가 **Succeeded**가 될 때까지 페이지를 새로고침합니다.
7.  탐색 창에서 Lakehouse를 선택합니다. 그런 다음 **Tables** 폴더를 확장하여 **products**라는 이름의 테이블이 Dataflow에 의해 생성되고 로드되었는지 확인합니다(**Tables** 폴더를 새로고침해야 할 수 있습니다).

    ![lakehouse 페이지의 products 테이블 스크린샷](./Images/products-table.png)

## Spark Notebook 생성 및 모니터링

---
### **개념 설명: Spark Notebook**

Notebook은 데이터 엔지니어와 데이터 과학자가 코드를 작성하고 실행하며, 그 결과를 즉시 확인하고 문서를 함께 작성할 수 있는 대화형 환경입니다. Microsoft Fabric의 Notebook은 Apache Spark를 기반으로 동작하므로, 대규모 데이터 처리가 필요한 복잡한 데이터 엔지니어링 작업이나 머신러닝 모델 학습과 같은 데이터 과학 작업을 수행하는 데 이상적입니다.

Python(PySpark), Scala, Spark SQL, R 등 다양한 언어를 지원하며, 이 실습에서는 Notebook을 사용하여 Lakehouse에 저장된 데이터를 Spark를 이용해 간단히 쿼리하는 방법을 경험합니다.
---

Microsoft Fabric에서는 Notebook을 사용하여 Spark 코드를 실행할 수 있습니다.

1.  왼쪽 메뉴 바에서 **Create**를 선택합니다. *New* 페이지의 *Data Engineering* 섹션에서 **Notebook**을 선택합니다.

    **Notebook 1**이라는 이름의 새 Notebook이 생성되고 열립니다.

    ![새로운 notebook 스크린샷](./Images/new-notebook.png)

2.  Notebook 왼쪽 상단에서 **Notebook 1**을 선택하여 세부 정보를 보고, 이름을 `Query Products`로 변경합니다.
3.  Notebook 편집기의 **Explorer** 창에서 **Add data items**를 선택한 다음 **Existing data sources**를 선택합니다.
4.  이전에 생성한 Lakehouse를 추가합니다.
5.  **products** 테이블에 도달할 때까지 Lakehouse 항목을 확장합니다.
6.  **products** 테이블의 **...** 메뉴에서 **Load data** > **Spark**를 선택합니다. 이렇게 하면 다음과 같이 Notebook에 새 코드 셀이 추가됩니다.

    ![테이블을 쿼리하는 코드가 있는 notebook 스크린샷](./Images/load-spark.png)

    > **코드 설명:**
    > *   `df = spark.read.table("your_lakehouse_name.products")`: 이 코드는 Spark 세션을 사용하여 지정된 Lakehouse(`your_lakehouse_name`) 내의 `products` 테이블을 읽어옵니다. 읽어온 데이터는 `df`라는 이름의 Spark DataFrame 객체에 저장됩니다. DataFrame은 분산된 데이터 컬렉션을 나타내는 핵심적인 데이터 구조입니다.
    > *   `display(df)`: 이 함수는 DataFrame `df`의 내용을 표 형식으로 시각화하여 Notebook 출력에 표시합니다. 대량의 데이터를 다룰 때, `display` 함수는 샘플 데이터만을 보여주어 결과를 빠르게 확인할 수 있도록 도와줍니다.

7.  **&#9655; Run all** 버튼을 사용하여 Notebook의 모든 셀을 실행합니다. Spark 세션을 시작하는 데 잠시 시간이 걸리며, 그 후 쿼리 결과가 코드 셀 아래에 표시됩니다.

    ![쿼리 결과가 있는 notebook 스크린샷](./Images/notebook-output.png)

8.  툴바에서 **&#9723;** (*Stop session*) 버튼을 사용하여 Spark 세션을 중지합니다.
9.  탐색 바에서 **Monitor**를 선택하여 모니터링 허브를 보고, Notebook 활동이 나열되어 있는지 확인합니다.

    ![notebook 활동이 있는 모니터링 허브 스크린샷](./Images/monitor-notebook.png)

## 항목의 기록 모니터링

Workspace의 일부 항목은 여러 번 실행될 수 있습니다. 모니터링 허브를 사용하여 실행 기록을 볼 수 있습니다.

1.  탐색 바에서 Workspace 페이지로 돌아갑니다. 그런 다음 **Get Product Data** Dataflow의 **&#8635;** (*Refresh now*) 버튼을 사용하여 다시 실행합니다.
2.  탐색 창에서 **Monitor** 페이지를 선택하여 모니터링 허브를 보고 Dataflow가 진행 중인지 확인합니다.
3.  **Get Product Data** Dataflow의 **...** 메뉴에서 **Historical runs**를 선택하여 Dataflow의 실행 기록을 봅니다.

    ![모니터링 허브의 실행 기록 뷰 스크린샷](./Images/historical-runs.png)

4.  과거 실행 기록 중 하나의 **...** 메뉴에서 **View detail**을 선택하여 실행 세부 정보를 봅니다.
5.  **Details** 창을 닫고 **Back to main view** 버튼을 사용하여 주 모니터링 허브 페이지로 돌아갑니다.

## 모니터링 허브 뷰 사용자 정의하기

이 실습에서는 몇 가지 활동만 실행했으므로 모니터링 허브에서 이벤트를 찾는 것이 비교적 쉬워야 합니다. 그러나 실제 환경에서는 많은 수의 이벤트를 검색해야 할 수 있습니다. 필터 및 기타 뷰 사용자 정의를 사용하면 이 작업이 더 쉬워집니다.

1.  모니터링 허브에서 **Filter** 버튼을 사용하여 다음 필터를 적용합니다.
    *   **Status**: `Succeeded`
    *   **Item type**: `Dataflow Gen2`

    필터가 적용되면 성공적으로 실행된 Dataflow만 나열됩니다.

    ![필터가 적용된 모니터링 허브 스크린샷](./Images/monitor-filter.png)

2.  **Column Options** 버튼을 사용하여 다음 열을 뷰에 포함시킵니다(**Apply** 버튼을 사용하여 변경 사항을 적용).
    *   Activity name
    *   Status
    *   Item type
    *   Start time
    *   Submitted by
    *   Location
    *   End time
    *   Duration
    *   Refresh type

    모든 열을 보려면 수평으로 스크롤해야 할 수 있습니다.

    ![사용자 정의 열이 있는 모니터링 허브 스크린샷](./Images/monitor-columns.png)

## 리소스 정리

이 실습에서는 Lakehouse, Dataflow, Spark Notebook을 생성하고 모니터링 허브를 사용하여 항목 활동을 보았습니다.

Lakehouse 탐색을 마쳤다면, 이 실습을 위해 생성한 Workspace를 삭제할 수 있습니다.

1.  왼쪽 바에서 Workspace 아이콘을 선택하여 포함된 모든 항목을 봅니다.
2.  툴바의 **...** 메뉴에서 **Workspace settings**를 선택합니다.
3.  **General** 섹션에서 **Remove this workspace**를 선택합니다.
