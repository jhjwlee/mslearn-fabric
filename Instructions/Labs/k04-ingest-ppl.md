
# 핸즈온 랩: Microsoft Fabric에서 파이프라인으로 데이터 수집하기

데이터 레이크하우스는 클라우드 규모 분석 솔루션을 위한 일반적인 분석 데이터 저장소입니다. 데이터 엔지니어의 핵심 작업 중 하나는 여러 운영 데이터 소스에서 레이크하우스로 데이터를 수집하는 것을 구현하고 관리하는 것입니다. Microsoft Fabric에서는 **파이프라인(pipelines)** 생성을 통해 데이터 수집을 위한 *추출, 변환, 로드*(**ETL**) 또는 *추출, 로드, 변환*(**ELT**) 솔루션을 구현할 수 있습니다.

또한 Fabric은 Apache Spark를 지원하므로 코드를 작성하고 실행하여 대규모 데이터를 처리할 수 있습니다. Fabric의 파이프라인과 Spark 기능을 결합하면 외부 소스에서 레이크하우스의 기반이 되는 OneLake 저장소로 데이터를 복사한 다음, Spark 코드를 사용하여 사용자 지정 데이터 변환을 수행하고 분석을 위해 테이블에 로드하는 복잡한 데이터 수집 로직을 구현할 수 있습니다.

이 실습을 완료하는 데 약 **45**분이 소요됩니다.

> **참고**
> 이 실습을 완료하려면 [Microsoft Fabric 테넌트](https://learn.microsoft.com/fabric/get-started/fabric-trial)에 대한 액세스 권한이 필요합니다.

## Workspace 만들기

Fabric에서 데이터 작업을 시작하기 전에 Fabric 평가판이 활성화된 Workspace를 만들어야 합니다.

1.  웹 브라우저에서 [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com/home?experience=fabric-developer) `https://app.fabric.microsoft.com/home?experience=fabric-developer`로 이동하여 Fabric 자격 증명으로 로그인합니다.
2.  왼쪽 메뉴 모음에서 **Workspaces** 아이콘( izgled: &#128455;)을 선택합니다.
3.  원하는 이름으로 새 Workspace를 만듭니다. **Advanced** 섹션에서 Fabric 용량(*Trial*, *Premium*, 또는 *Fabric*)을 포함하는 라이선스 모드를 선택해야 합니다.
4.  새 Workspace가 열리면 처음에는 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷](./Images/new-workspace.png)

## Lakehouse 만들기

이제 Workspace가 준비되었으니, 데이터를 수집할 데이터 레이크하우스를 만들 차례입니다.

1.  왼쪽 메뉴 모음에서 **Create**를 선택합니다. *New* 페이지의 *Data Engineering* 섹션에서 **Lakehouse**를 선택하고, 원하는 고유한 이름을 지정합니다.
2.  **Explorer** 창의 왼쪽에서 **Files** 노드의 **…** 메뉴를 선택하고 **New subfolder**를 선택하여 **new_data**라는 하위 폴더를 만듭니다.

## 파이프라인 만들기

**개념 설명**: **Pipeline**은 데이터 이동 및 변환 작업을 자동화하고 조율(orchestrate)하는 데 사용되는 논리적인 그룹입니다. 파이프라인은 하나 이상의 **Activity(활동)**로 구성되며, 각 활동은 특정 작업을 수행합니다. 예를 들어, **Copy Data** 활동은 한 위치에서 다른 위치로 데이터를 복사하고, **Notebook** 활동은 Spark 코드를 실행합니다.

데이터를 수집하는 간단한 방법은 파이프라인에서 **Copy Data** 활동을 사용하여 소스에서 데이터를 추출하고 레이크하우스의 파일에 복사하는 것입니다.

1.  Lakehouse의 **Home** 페이지에서 **Get data**를 선택한 다음 **New data pipeline**을 선택하고, `Ingest Sales Data`라는 이름의 새 데이터 파이프라인을 만듭니다.
2.  **Copy Data** 마법사가 자동으로 열리지 않으면 파이프라인 편집기 페이지에서 **Copy Data > Use copy assistant**를 선택합니다.
3.  **Copy Data** 마법사의 **Choose data source** 페이지에서 검색창에 `HTTP`를 입력한 다음 **New sources** 섹션에서 **HTTP**를 선택합니다.

    ![데이터 소스 선택 페이지 스크린샷](./Images/choose-data-source.png)

4.  **Connect to data source** 창에서 데이터 소스 연결에 대한 다음 설정을 입력합니다.
    - **URL**: `https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/sales.csv`
    - **Connection**: Create new connection
    - **Connection name**: *고유한 이름 지정*
    - **Data gateway**: (none)
    - **Authentication kind**: Anonymous
5.  **Next**를 선택한 후, 데이터가 샘플링될 때까지 기다렸다가 다음 설정이 선택되었는지 확인합니다.
    - **File format**: DelimitedText
    - **Column delimiter**: Comma (,)
    - **Row delimiter**: Line feed (\n)
    - **First row as header**: 선택됨
    - **Compression type**: None
6.  **Preview data**를 선택하여 수집될 데이터의 샘플을 확인합니다. 그런 다음 데이터 미리보기를 닫고 **Next**를 선택합니다.
7.  **Connect to data destination** 페이지에서 다음 데이터 대상 옵션을 설정한 다음 **Next**를 선택합니다.
    - **Root folder**: Files
    - **Folder path name**: new_data
    - **File name**: sales.csv
    - **Copy behavior**: None
8.  **Copy summary** 페이지에서 복사 작업의 세부 정보를 검토한 다음 **Save + Run**을 선택합니다.

    **Copy Data** 활동을 포함하는 새 파이프라인이 다음과 같이 생성됩니다.

    ![Copy Data 활동이 포함된 파이프라인 스크린샷](./Images/copy-data-pipeline.png)

9.  파이프라인 실행이 시작되면 파이프라인 디자이너 아래의 **Output** 창에서 상태를 모니터링할 수 있습니다. **&#8635;** (*Refresh*) 아이콘을 사용하여 상태를 새로 고치고 성공할 때까지 기다립니다.
10. 왼쪽 메뉴 모음에서 레이크하우스를 선택합니다.
11. **Home** 페이지의 **Explorer** 창에서 **Files**를 확장하고 **new_data** 폴더를 선택하여 **sales.csv** 파일이 복사되었는지 확인합니다.

## Notebook 만들기

**핸즈온의 의미**: 이 단계에서는 앞서 파이프라인을 통해 Lakehouse의 'Files' 영역으로 복사된 원시 데이터를 Spark 코드를 사용하여 가공하고, 분석에 적합한 구조화된 'Tables' 영역으로 적재하는 과정을 경험합니다. 이는 **ELT (Extract-Load-Transform)** 패턴의 'T'(Transform) 부분에 해당합니다.

1.  Lakehouse의 **Home** 페이지에서 **Open notebook** 메뉴의 **New notebook**을 선택합니다.
2.  기존 셀의 기본 코드를 다음 변수 선언으로 바꿉니다.

    ```python
    table_name = "sales"
    ```

3.  셀의 **…** 메뉴에서 **Toggle parameter cell**을 선택합니다. 이렇게 하면 파이프라인에서 노트북을 실행할 때 이 셀에 선언된 변수가 매개변수(parameter)로 처리되도록 구성됩니다.
4.  매개변수 셀 아래에 **+ Code** 버튼을 사용하여 새 코드 셀을 추가합니다. 그런 다음 다음 코드를 추가합니다.

    ```python
    from pyspark.sql.functions import *

    # Read the new sales data
    df = spark.read.format("csv").option("header","true").load("Files/new_data/*.csv")

    ## Add month and year columns
    df = df.withColumn("Year", year(col("OrderDate"))).withColumn("Month", month(col("OrderDate")))

    # Derive FirstName and LastName columns
    df = df.withColumn("FirstName", split(col("CustomerName"), " ").getItem(0)).withColumn("LastName", split(col("CustomerName"), " ").getItem(1))

    # Filter and reorder columns
    df = df["SalesOrderNumber", "SalesOrderLineNumber", "OrderDate", "Year", "Month", "FirstName", "LastName", "EmailAddress", "Item", "Quantity", "UnitPrice", "TaxAmount"]

    # Load the data into a table
    df.write.format("delta").mode("append").saveAsTable(table_name)
    ```
    **코드 설명**:
    - `spark.read...load("Files/new_data/*.csv")`: `new_data` 폴더에 있는 모든 CSV 파일을 읽어 DataFrame으로 로드합니다.
    - `df.withColumn(...)`: `OrderDate`에서 연도와 월을 추출하여 새 열을 추가하고, `CustomerName`을 분리하여 이름과 성 열을 만듭니다.
    - `df.write.format("delta").mode("append").saveAsTable(table_name)`: 변환된 DataFrame을 Delta 형식의 테이블로 저장합니다. `mode("append")`는 테이블이 이미 존재할 경우 데이터를 덮어쓰지 않고 추가하도록 지정합니다. `table_name`은 위에서 정의한 매개변수 셀의 변수를 사용합니다.

5.  노트북이 다음과 유사하게 보이는지 확인한 다음, 툴바의 **&#9655; Run all** 버튼을 사용하여 포함된 모든 셀을 실행합니다.

    ![매개변수 셀과 데이터 변환 코드가 포함된 노트북 스크린샷](./Images/notebook.png)

6.  노트북 실행이 완료되면 왼쪽의 **Explorer** 창에서 **Tables**의 **…** 메뉴를 선택하고 **Refresh**를 선택하여 **sales** 테이블이 생성되었는지 확인합니다.
7.  노트북 이름을 `Load Sales`로 변경합니다.
8.  레이크하우스로 돌아가 **Explorer** 창을 새로고침하고, **Tables**를 확장한 다음, **sales** 테이블을 선택하여 포함된 데이터의 미리보기를 확인합니다.

## 파이프라인 수정하기

이제 데이터를 변환하고 테이블에 로드하는 노트북을 구현했으므로, 재사용 가능한 ETL 프로세스를 만들기 위해 이 노트북을 파이프라인에 통합할 수 있습니다.

**핸즈온의 의미**: 이 단계에서는 개별적으로 실행했던 **Copy Data** 활동과 **Notebook** 활동을 하나의 파이프라인으로 연결하여 완전한 자동화 프로세스를 구축합니다. 또한, 반복 실행 시 데이터 중복을 방지하기 위해 기존 파일을 삭제하는 **Delete data** 활동을 추가함으로써, 보다 견고하고 실제적인 데이터 파이프라인을 만드는 방법을 학습합니다.

1.  이전에 만든 **Ingest Sales Data** 파이프라인을 엽니다.
2.  **Activities** 탭에서 **Delete data** 활동을 추가합니다. 새 **Delete data** 활동을 **Copy data** 활동의 왼쪽에 배치하고, **On completion** 출력을 **Copy data** 활동에 연결합니다.

    ![Delete data 및 Copy data 활동이 포함된 파이프라인 스크린샷](./Images/delete-data-activity.png)

3.  **Delete data** 활동을 선택하고 다음과 같이 속성을 설정합니다.
    - **General**:
        - **Name**: `Delete old files`
    - **Source**:
        - **File path type**: Wildcard file path
        - **Folder path**: Files / **new_data**
        - **Wildcard file name**: `*.csv`        
        - **Recursively**: 선택됨
    - **Logging settings**:
        - **Enable logging**: 선택 해제

4.  **Activities** 탭에서 **Notebook** 활동을 파이프라인에 추가합니다.
5.  **Copy data** 활동의 **On Completion** 출력을 **Notebook** 활동에 연결합니다.

    ![Copy Data 및 Notebook 활동이 포함된 파이p라인 스크린샷](./Images/pipeline.png)

6.  **Notebook** 활동을 선택하고 다음과 같이 속성을 설정합니다.
    - **General**:
        - **Name**: `Load Sales notebook`
    - **Settings**:
        - **Notebook**: Load Sales
        - **Base parameters**: 다음 속성을 가진 새 매개변수 추가:
            
            | Name | Type | Value |
            | -- | -- | -- |
            | table_name | String | new_sales |

    `table_name` 매개변수는 노트북에 전달되어 매개변수 셀의 `table_name` 변수에 할당된 기본값을 재정의합니다. 즉, 이번 파이프라인 실행에서는 'sales'가 아닌 'new_sales'라는 이름의 테이블이 생성됩니다.

7.  **Home** 탭에서 **&#128427;** (*Save*) 아이콘을 사용하여 파이프라인을 저장합니다. 그런 다음 **&#9655; Run** 버튼을 사용하여 파이프라인을 실행하고 모든 활동이 완료될 때까지 기다립니다.

    ![데이터 흐름 활동이 포함된 파이프라인 스크린샷](./Images/pipeline-run.png)

8.  레이크하우스로 이동하여 **Explorer** 창에서 **Tables**를 확장하고, 파이프라인에 의해 실행된 노트북이 생성한 **new_sales** 테이블을 선택하여 데이터 미리보기를 확인합니다.

이 실습에서는 파이프라인을 사용하여 외부 소스에서 레이크하우스로 데이터를 복사한 다음, Spark 노트북을 사용하여 데이터를 변환하고 테이블에 로드하는 데이터 수집 솔루션을 구현했습니다.

## 추가실습
6.  **Notebook** 활동을 선택하고 다음과 같이 속성을 설정합니다.
    - **General**:
        - **Name**: `Load Sales notebook`
    - **Settings**:
        - **Notebook**: Load Sales
        - **Base parameters**: 다음 속성을 가진 새 매개변수 추가:
            
            | Name | Type | Value |
            | -- | -- | -- |
            | table_name | String | ****** |

File path 입력란 아래의 **'Add dynamic content'**를 클릭합니다.
표현식 작성기에 다음과 같이 입력합니다.
```
    @concat('daily_sales', formatDateTime(utcNow(), 'yyyy_MM_dd'))
```
7.  **Home** 탭에서 **&#128427;** (*Save*) 아이콘을 사용하여 파이프라인을 저장합니다. 그런 다음 **&#9655; Run** 버튼을 사용하여 파이프라인을 실행하고 모든 활동이 완료될 때까지 기다립니다.

    ![데이터 흐름 활동이 포함된 파이프라인 스크린샷](./Images/pipeline-run.png)

8.  레이크하우스로 이동하여 **Explorer** 창에서 **Tables**를 확장하고, 파이프라인에 의해 실행된 노트북이 생성한 테이블 (테이블 명에 날짜가 포함되어 있는지?) 을 선택하여 데이터 미리보기를 확인합니다.


          
## 리소스 정리

이 실습에서는 Microsoft Fabric에서 파이프라인을 구현하는 방법을 배웠습니다.

레이크하우스 탐색을 마쳤다면 이 실습을 위해 만든 Workspace를 삭제할 수 있습니다.

1.  왼쪽 막대에서 Workspace 아이콘을 선택하여 포함된 모든 항목을 봅니다.
2.  **Workspace settings**를 선택하고 **General** 섹션에서 아래로 스크롤하여 **Remove this workspace**를 선택합니다.
3.  **Delete**를 선택하여 Workspace를 삭제합니다.
