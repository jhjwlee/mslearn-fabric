

# 핸즈온 랩: Spark와 Microsoft Fabric Notebook을 사용하여 데이터 수집하기

이 랩에서는 Microsoft Fabric Notebook을 생성하고 PySpark를 사용하여 Azure Blob Storage 경로에 연결한 다음, 쓰기 최적화를 사용하여 데이터를 Lakehouse로 로드합니다.

이 실습을 완료하는 데 약 **30**분이 소요됩니다.

이 경험을 위해, 여러 Notebook 코드 셀에 걸쳐 코드를 작성하게 되는데, 이는 실제 환경에서 작업하는 방식과는 다를 수 있지만 디버깅에 유용할 수 있습니다.

또한 샘플 데이터 세트로 작업하기 때문에, 최적화가 대규모 프로덕션 환경에서 볼 수 있는 것과 같지는 않지만, 여전히 개선 효과를 볼 수 있으며 매 밀리초가 중요할 때 최적화는 핵심입니다.

> **참고**: 이 실습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## Workspace 만들기

Fabric에서 데이터 작업을 시작하기 전에 Fabric 평가판이 활성화된 Workspace를 만들어야 합니다.

1.  웹 브라우저에서 [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com/home?experience=fabric) `https://app.fabric.microsoft.com/home?experience=fabric`로 이동하여 Fabric 자격 증명으로 로그인합니다.
2.  왼쪽 메뉴 모음에서 **Workspaces** 아이콘( izgled: &#128455;)을 선택합니다.
3.  원하는 이름으로 새 Workspace를 만듭니다. **Advanced** 섹션에서 Fabric 용량(*Trial*, *Premium*, 또는 *Fabric*)을 포함하는 라이선스 모드를 선택해야 합니다.
4.  새 Workspace가 열리면 처음에는 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷](./Images/new-workspace.png)

## Workspace 및 Lakehouse 목적지 만들기

먼저 새 Lakehouse를 만들고, Lakehouse에 목적지 폴더를 만듭니다.

1.  Workspace에서 **+ New item > Lakehouse**를 선택하고, 이름을 지정한 후 **Create**를 선택합니다.
2.  **Files**에서 **[...]**를 선택하여 `RawData`라는 이름의 **New subfolder**를 만듭니다.
3.  Lakehouse 내의 Lakehouse Explorer에서 **RawData > ... > Properties**를 선택합니다.
4.  **RawData** 폴더의 **ABFS path**를 나중에 사용할 수 있도록 빈 메모장에 복사합니다. 경로는 다음과 유사한 형태여야 합니다.
    `abfss://{workspace_name}@onelake.dfs.fabric.microsoft.com/{lakehouse_name}.Lakehouse/Files/{folder_name}`

이제 Lakehouse와 RawData 목적지 폴더가 있는 Workspace가 준비되었습니다.

## Fabric Notebook 생성 및 외부 데이터 로드하기

**핸즈온의 의미**: 이 단계는 Fabric Notebook의 핵심 기능 중 하나를 보여줍니다. 즉, 외부 클라우드 스토리지(여기서는 공개 Azure Blob Storage)에 저장된 대용량 데이터(NYC 옐로우 택시 데이터)에 직접 연결하여 데이터를 Spark DataFrame으로 읽어오는 과정을 경험합니다. 이를 통해 데이터를 먼저 Fabric으로 이동시키지 않고도 원격 데이터를 직접 분석하고 처리할 수 있는 강력함을 체험할 수 있습니다.

1.  Lakehouse의 상단 메뉴에서 **Open notebook > New notebook**을 선택하여 새 Notebook을 엽니다.
2.  기본 셀에서 코드가 **PySpark (Python)**으로 설정되어 있는지 확인합니다.
3.  코드 셀에 다음 코드를 삽입합니다. 이 코드는 다음 작업을 수행합니다.
    - 연결 문자열을 위한 매개변수 선언
    - 연결 문자열 구축
    - 데이터를 DataFrame으로 읽기

    ```Python
    # Azure Blob Storage access info
    blob_account_name = "azureopendatastorage"
    blob_container_name = "nyctlc"
    blob_relative_path = "yellow"
    
    # Construct connection path
    wasbs_path = f'wasbs://{blob_container_name}@{blob_account_name}.blob.core.windows.net/{blob_relative_path}'
    print(wasbs_path)
    
    # Read parquet data from Azure Blob Storage path
    blob_df = spark.read.parquet(wasbs_path)
    ```
    **코드 설명**:
    - `wasbs://...`: Azure Blob Storage에 연결하기 위한 프로토콜 및 경로 형식입니다. `wasbs`는 보안 연결을 의미합니다.
    - `spark.read.parquet(wasbs_path)`: Spark 세션을 사용하여 지정된 `wasbs_path`의 모든 Parquet 파일을 읽어 하나의 Spark DataFrame(`blob_df`)으로 로드합니다.

4.  코드 셀 옆의 **&#9655; Run Cell**을 선택하여 DataFrame에 데이터를 연결하고 읽습니다.
    **예상 결과:** 명령이 성공적으로 실행되고 `wasbs://nyctlc@azureopendatastorage.blob.core.windows.net/yellow`가 출력됩니다.

5.  데이터를 파일로 쓰려면 이제 **RawData** 폴더의 **ABFS 경로**가 필요합니다.
6.  **새 코드 셀**에 다음 코드를 삽입합니다.

    ```python
    # Declare file name    
    file_name = "yellow_taxi"
    
    # Construct destination path
    output_parquet_path = f"**ここにABFSパスを挿入**/{file_name}"
    print(output_parquet_path)
        
    # Load the first 1000 rows as a Parquet file
    blob_df.limit(1000).write.mode("overwrite").parquet(output_parquet_path)
    ```
    **코드 설명**:
    - `f"**ここにABFSパスを挿入**/{file_name}"`: 이전에 복사해 둔 `RawData` 폴더의 ABFS 경로를 `**ここにABFSパスを挿入**` 부분에 붙여넣습니다.
    - `blob_df.limit(1000)`: 전체 데이터가 매우 크므로, 테스트를 위해 처음 1000개의 행만 선택합니다.
    - `.write.mode("overwrite").parquet(output_parquet_path)`: 선택된 데이터를 Parquet 형식으로 지정된 `output_parquet_path`에 저장합니다. `mode("overwrite")`는 동일한 이름의 파일이 이미 존재할 경우 덮어쓰도록 합니다.

7.  **RawData**의 ABFS 경로를 추가하고 **&#9655; Run Cell**을 선택하여 1000개의 행을 `yellow_taxi.parquet` 파일로 씁니다.
8.  Lakehouse Explorer에서 데이터 로드를 확인하려면 **Files > ... > Refresh**를 선택합니다.

이제 **RawData**라는 새 폴더와 그 안에 `yellow_taxi.parquet` "파일"이 보일 것입니다. (실제로는 파티션 파일들이 들어있는 폴더로 표시됩니다.)

## 데이터 변환 및 Delta 테이블로 로드하기

데이터 수집 작업은 단순히 파일을 로드하는 것만으로 끝나지 않을 가능성이 높습니다. Lakehouse의 Delta 테이블은 확장 가능하고 유연한 쿼리 및 저장을 허용하므로, 이 테이블도 만들어 보겠습니다.

**핸즈온의 의미**: 이 단계는 일반적인 **ETL(Extract-Transform-Load)** 프로세스 중 **T(변환)**와 **L(로드)** 단계를 수행합니다. 원시 파일(Parquet)을 읽어와서, 비즈니스 요구사항에 따라 데이터를 변환(새 열 추가, 불필요한 데이터 필터링)한 다음, 최종적으로 분석에 최적화된 구조화된 **Delta 테이블**로 저장하는 과정을 경험합니다.

1.  새 코드 셀을 만들고 다음 코드를 삽입합니다.

    ```python
    from pyspark.sql.functions import col, to_timestamp, current_timestamp, year, month
    
    # Read the parquet data from the specified path
    raw_df = spark.read.parquet(output_parquet_path)   
    
    # Add dataload_datetime column with current timestamp
    filtered_df = raw_df.withColumn("dataload_datetime", current_timestamp())
    
    # Filter columns to exclude any NULL values in storeAndFwdFlag
    filtered_df = filtered_df.filter(col("storeAndFwdFlag").isNotNull())
    
    # Load the filtered data into a Delta table
    table_name = "yellow_taxi"
    filtered_df.write.format("delta").mode("append").saveAsTable(table_name)
    
    # Display results
    display(filtered_df.limit(1))
    ```
    **코드 설명**:
    - `raw_df.withColumn("dataload_datetime", current_timestamp())`: 데이터가 로드된 시점을 기록하기 위해 `dataload_datetime`이라는 새 열을 현재 타임스탬프 값으로 추가합니다.
    - `filtered_df.filter(col("storeAndFwdFlag").isNotNull())`: `storeAndFwdFlag` 열에 Null 값이 있는 행을 제거하여 데이터를 정제합니다.
    - `.write.format("delta").mode("append").saveAsTable(table_name)`: 필터링된 데이터를 `yellow_taxi`라는 이름의 Delta 테이블로 저장합니다. `mode("append")`는 테이블이 이미 존재할 경우 데이터를 추가하도록 합니다.

2.  코드 셀 옆의 **&#9655; Run Cell**을 선택합니다.
3.  표시된 결과를 검토하고 확인합니다.

    ![성공적인 출력을 보여주는 스크린샷](Images/notebook-transform-result.png)

이제 외부 데이터에 성공적으로 연결하고, Parquet 파일로 쓰고, 데이터를 DataFrame으로 로드하고, 데이터를 변환하고, Delta 테이블로 로드했습니다.

## SQL 쿼리로 Delta 테이블 데이터 분석하기

이 랩은 데이터 수집에 중점을 두고 *추출, 변환, 로드* 프로세스를 설명하지만, 데이터를 미리 보는 것도 중요합니다.

**개념 설명: Spark SQL**
Spark SQL은 Spark의 SQL 언어 API로, SQL 문을 실행하거나 데이터를 관계형 테이블에 유지하는 데 사용할 수 있습니다. 많은 데이터 분석가들이 SQL 구문에 익숙하기 때문에, PySpark 코드 내에서 직접 SQL을 사용하여 데이터를 쿼리하고 조작할 수 있는 강력하고 유연한 방법을 제공합니다. `createOrReplaceTempView`는 DataFrame을 SQL로 쿼리할 수 있는 임시 뷰(View)로 등록하는 함수입니다.

1.  새 코드 셀을 만들고 아래 코드를 삽입합니다.

    ```python
    # Load table into df
    delta_table_name = "yellow_taxi"
    table_df = spark.read.format("delta").table(delta_table_name)
    
    # Create temp SQL table
    table_df.createOrReplaceTempView("yellow_taxi_temp")
    
    # SQL Query
    table_df = spark.sql('SELECT * FROM yellow_taxi_temp')
    
    # Display 10 results
    display(table_df.limit(10))
    ```
    **코드 설명**:
    - `spark.read.format("delta").table(delta_table_name)`: `yellow_taxi` Delta 테이블을 읽어 DataFrame으로 로드합니다.
    - `table_df.createOrReplaceTempView("yellow_taxi_temp")`: 이 DataFrame을 `yellow_taxi_temp`라는 이름의 임시 뷰로 등록합니다. 이 뷰는 현재 Spark 세션에서만 유효합니다.
    - `spark.sql('SELECT * FROM yellow_taxi_temp')`: PySpark 코드 내에서 SQL 쿼리를 직접 실행하여 `yellow_taxi_temp` 뷰에서 모든 데이터를 선택합니다.

2.  코드 셀 옆의 **&#9655; Run Cell**을 선택합니다.

## 리소스 정리

이 실습에서는 Fabric에서 PySpark와 함께 Notebook을 사용하여 데이터를 로드하고 Parquet으로 저장했습니다. 그런 다음 해당 Parquet 파일을 사용하여 데이터를 추가로 변환했습니다. 마지막으로 SQL을 사용하여 Delta 테이블을 쿼리했습니다.

탐색을 마쳤으면 이 실습을 위해 만든 Workspace를 삭제할 수 있습니다.

1.  왼쪽 막대에서 Workspace 아이콘을 선택하여 포함된 모든 항목을 봅니다.
2.  **Workspace settings**를 선택하고 **General** 섹션에서 아래로 스크롤하여 **Remove this workspace**를 선택합니다.
3.  **Delete**를 선택하여 Workspace를 삭제합니다.
