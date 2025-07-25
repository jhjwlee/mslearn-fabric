
# 핸즈온 랩: Microsoft Fabric에서 Notebook을 사용하여 데이터 과학을 위한 데이터 탐색하기

이 랩에서는 데이터 탐색을 위해 Notebook을 사용합니다.

**개념 설명**: **Notebook**은 데이터 과학자와 분석가에게 필수적인 도구입니다. 코드를 셀 단위로 작성하고, 실행하며, 그 결과를 즉시 확인하고, 설명 텍스트(Markdown)와 시각화를 코드와 함께 저장할 수 있는 대화형 환경을 제공합니다. 이러한 특징 덕분에 데이터를 단계별로 탐색하고, 가설을 검증하며, 분석 과정을 문서화하고 공유하는 **탐색적 데이터 분석(Exploratory Data Analysis, EDA)** 작업에 매우 효과적입니다.

이 실습에서는 데이터 세트를 탐색하고, 요약 통계를 생성하며, 데이터를 더 잘 이해하기 위한 시각화를 만드는 방법을 배웁니다.

이 실습을 완료하는 데 약 **30**분이 소요됩니다.

> **참고**: 이 실습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## Workspace 만들기

Fabric에서 데이터 작업을 시작하기 전에 Fabric 평가판이 활성화된 Workspace를 만들어야 합니다.

1.  웹 브라우저에서 [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com/home?experience=fabric) `https://app.fabric.microsoft.com/home?experience=fabric`로 이동하여 Fabric 자격 증명으로 로그인합니다.
2.  왼쪽 메뉴 모음에서 **Workspaces** 아이콘( izgled: &#128455;)을 선택합니다.
3.  원하는 이름으로 새 Workspace를 만듭니다. **Advanced** 섹션에서 Fabric 용량(*Trial*, *Premium*, 또는 *Fabric*)을 포함하는 라이선스 모드를 선택해야 합니다.
4.  새 Workspace가 열리면 처음에는 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷](./Images/new-workspace.png)

## Notebook 만들기

1.  왼쪽 메뉴 모음에서 **Create**를 선택합니다. *New* 페이지의 *Data Science* 섹션에서 **Notebook**을 선택하고, 원하는 고유한 이름을 지정합니다.
2.  첫 번째 셀을 Markdown 셀로 변환한 다음, 내용을 삭제하고 다음 텍스트를 입력합니다.

    ```text
    # Perform data exploration for data science

    Use the code in this notebook to perform data exploration for data science.
    ```

## 데이터프레임으로 데이터 로드하기

이제 코드를 실행하여 데이터를 가져올 준비가 되었습니다. Azure Open Datasets의 [**당뇨병 데이터 세트**](https://learn.microsoft.com/azure/open-datasets/dataset-diabetes?tabs=azureml-opendatasets?azure-portal=true)를 사용합니다. 데이터를 로드한 후에는 행과 열로 작업하기 위한 일반적인 구조인 Pandas 데이터프레임으로 변환합니다.

1.  Notebook에서 최신 셀 아래의 **+ Code** 아이콘을 사용하여 새 코드 셀을 추가합니다.
2.  다음 코드를 입력하여 데이터 세트를 데이터프레임으로 로드합니다.

    ```python
    # Azure storage access info for open dataset diabetes
    blob_account_name = "azureopendatastorage"
    blob_container_name = "mlsamples"
    blob_relative_path = "diabetes"
    blob_sas_token = r"" # Blank since container is Anonymous access
    
    # Set Spark config to access  blob storage
    wasbs_path = f"wasbs://%s@%s.blob.core.windows.net/%s" % (blob_container_name, blob_account_name, blob_relative_path)
    spark.conf.set("fs.azure.sas.%s.%s.blob.core.windows.net" % (blob_container_name, blob_account_name), blob_sas_token)
    print("Remote blob path: " + wasbs_path)
    
    # Spark read parquet, note that it won't load any data yet by now
    df = spark.read.parquet(wasbs_path)
    ```

3.  셀을 실행합니다.
4.  새 코드 셀을 추가하고 다음 코드를 입력하여 로드된 데이터를 확인합니다.

    ```python
    display(df)
    ```

5.  셀 명령이 완료되면 셀 아래의 출력을 검토합니다. 데이터는 당뇨병 환자에 대한 10개의 기본 변수(나이, 성별, 체질량 지수, 평균 혈압 및 6개의 혈청 측정값)와 관심 반응(기준 1년 후 질병 진행의 정량적 측정값, **Y**로 표시됨)으로 구성됩니다.
6.  데이터는 Spark 데이터프레임으로 로드되었습니다. scikit-learn과 같은 많은 Python 데이터 과학 라이브러리는 Pandas 데이터프레임을 예상합니다. 아래 코드를 실행하여 데이터 세트를 Pandas 데이터프레임으로 변환합니다.

    ```python
    df = df.toPandas()
    df.head()
    ```

## 데이터 형태 확인하기

이제 데이터를 로드했으므로, 행과 열의 수, 데이터 유형, 결측값과 같은 데이터 세트의 구조를 확인할 수 있습니다.

1.  새 코드 셀에 다음 코드를 입력하고 실행합니다.

    ```python
    # Display the number of rows and columns in the dataset
    print("Number of rows:", df.shape[0])
    print("Number of columns:", df.shape[1])

    # Display the data types of each column
    print("\nData types of columns:")
    print(df.dtypes)
    ```
    데이터 세트에는 **442개의 행**과 **11개의 열**이 포함되어 있습니다. 이는 442개의 샘플과 11개의 특징 또는 변수가 있음을 의미합니다.

## 결측 데이터 확인하기

1.  새 코드 셀에 다음 코드를 입력하고 실행하여 결측값을 확인합니다.

    ```python
    missing_values = df.isnull().sum()
    print("\nMissing values per column:")
    print(missing_values)
    ```
    결과를 통해 이 데이터 세트에는 결측 데이터가 없음을 확인할 수 있습니다.

## 수치형 변수에 대한 기술 통계 생성하기

이제 기술 통계(descriptive statistics)를 생성하여 수치형 변수의 분포를 이해해 보겠습니다.

1.  새 코드 셀에 다음 코드를 입력하고 실행합니다.

    ```python
    df.describe()
    ```
    **결과 분석**: `describe()` 함수는 각 수치형 열에 대한 개수(count), 평균(mean), 표준편차(std), 최소값(min), 4분위수(25%, 50%, 75%), 최대값(max)을 보여줍니다.
    - 평균 연령은 약 48.5세이며, 표준 편차는 13.1년입니다. 가장 어린 개인은 19세이고 가장 나이가 많은 사람은 79세입니다.
    - 평균 BMI는 약 26.4로, [WHO 기준](https://www.who.int/health-topics/obesity#tab=tab_1)에 따르면 **과체중** 범주에 속합니다.

## 데이터 분포 시각화하기

`BMI` 특징을 검증하고, 그 특성을 더 잘 이해하기 위해 분포를 시각화해 보겠습니다.

1.  새 코드 셀에 다음 코드를 입력하고 실행합니다.

    ```python
    import matplotlib.pyplot as plt
    import seaborn as sns
    import numpy as np
    
    # Calculate the mean, median of the BMI variable
    mean = df['BMI'].mean()
    median = df['BMI'].median()
   
    # Histogram of the BMI variable
    plt.figure(figsize=(8, 6))
    plt.hist(df['BMI'], bins=20, color='skyblue', edgecolor='black')
    plt.title('BMI Distribution')
    plt.xlabel('BMI')
    plt.ylabel('Frequency')
    
    # Add lines for the mean and median
    plt.axvline(mean, color='red', linestyle='dashed', linewidth=2, label='Mean')
    plt.axvline(median, color='green', linestyle='dashed', linewidth=2, label='Median')
    
    # Add a legend
    plt.legend()
    plt.show()
    ```
    **결과 분석**: 이 히스토그램을 통해 데이터 세트에서 BMI의 범위와 분포를 관찰할 수 있습니다. 예를 들어, 대부분의 BMI는 23.2와 29.2 사이에 있으며, 데이터는 오른쪽으로 약간 치우쳐져 있습니다(right-skewed).

## 다변량 분석 수행하기

**핸즈온의 의미**: 이 섹션에서는 데이터 내의 패턴과 관계를 파악하기 위해 산점도(scatter plot) 및 상자 그림(box plot)과 같은 시각화를 생성합니다. 이는 단일 변수만 보는 것을 넘어, 여러 변수 간의 상호작용을 탐색하는 **다변량 분석**의 기초를 다지는 과정입니다. Python의 시각화 라이브러리인 `matplotlib`와 `seaborn`을 사용하여 직관적인 그래프를 만드는 방법을 직접 경험합니다.

1.  새 코드 셀에 다음 코드를 입력하여 BMI와 목표 변수(Y) 간의 관계를 산점도로 확인합니다.

    ```python
    import matplotlib.pyplot as plt
    import seaborn as sns

    # Scatter plot of BMI vs. Target variable
    plt.figure(figsize=(8, 6))
    sns.scatterplot(x='BMI', y='Y', data=df)
    plt.title('BMI vs. Target variable')
    plt.xlabel('BMI')
    plt.ylabel('Target')
    plt.show()
    ```
    **결과 분석**: BMI가 증가함에 따라 목표 변수도 증가하는 것을 볼 수 있으며, 이는 이 두 변수 사이에 양의 선형 관계가 있음을 나타냅니다.

2.  새 코드 셀에 다음 코드를 입력하여 성별에 따른 혈압 분포를 상자 그림으로 확인합니다.

    ```python
    import seaborn as sns
    import matplotlib.pyplot as plt
    
    fig, ax = plt.subplots(figsize=(7, 5))
    
    # Replace numeric values with labels
    df['SEX'] = df['SEX'].replace({1: 'Male', 2: 'Female'})
    
    sns.boxplot(x='SEX', y='BP', data=df, ax=ax)
    ax.set_title('Blood pressure across Gender')
    plt.tight_layout()
    plt.show()
    ```
    **결과 분석**: 이러한 관찰은 남성 및 여성 환자의 혈압 프로필에 차이가 있음을 시사합니다. 평균적으로 여성 환자는 남성 환자보다 혈압이 더 높습니다.

3.  데이터를 집계하면 시각화 및 분석이 더 용이해질 수 있습니다. 새 코드 셀에 다음 코드를 입력하여 성별에 따른 평균 혈압 및 BMI를 막대 차트로 비교합니다.

    ```python
    # ... (코드 생략) ...
    ```
    **결과 분석**: 이 그래프는 여성 환자의 평균 혈압이 남성 환자에 비해 더 높다는 것을 보여줍니다. 또한 평균 체질량 지수(BMI)도 여성이 남성보다 약간 더 높다는 것을 보여줍니다.

4.  새 코드 셀에 다음 코드를 입력하여 나이에 따른 BMI의 변화를 선 그래프로 확인합니다.

    ```python
    # ... (코드 생략) ...
    ```
    **결과 분석**: 19세에서 30세 연령 그룹이 가장 낮은 평균 BMI 값을 가지며, 가장 높은 평균 BMI는 65세에서 79세 연령 그룹에서 발견됩니다.

## 상관 관계 분석하기

**개념 설명: 상관 관계 분석 (Correlation Analysis)**
상관 관계 분석은 두 변수 간의 선형 관계의 강도와 방향을 측정하는 통계적 방법입니다. 상관 계수는 -1에서 +1 사이의 값을 가지며, +1에 가까울수록 강한 양의 상관 관계(한 변수가 증가할 때 다른 변수도 증가), -1에 가까울수록 강한 음의 상관 관계(한 변수가 증가할 때 다른 변수는 감소)를 의미합니다. 0에 가까우면 선형 관계가 없음을 나타냅니다. **히트맵(Heatmap)**은 이러한 상관 관계 행렬을 색상으로 시각화하여 변수 간의 관계를 한눈에 파악할 수 있도록 도와주는 매우 유용한 도구입니다.

1.  새 코드 셀에 다음 코드를 입력하여 모든 수치형 변수 간의 상관 관계를 계산합니다.

    ```python
    df.corr(numeric_only=True)
    ```

2.  상관 관계를 시각화하기 위해 히트맵을 생성합니다. 새 코드 셀에 다음 코드를 입력합니다.

    ```python
    plt.figure(figsize=(15, 7))
    sns.heatmap(df.corr(numeric_only=True), annot=True, vmin=-1, vmax=1, cmap="Blues")
    ```
    **결과 분석**: `S1`과 `S2` 변수는 **0.89**의 높은 양의 상관 관계를 가지며, 이는 두 변수가 같은 방향으로 움직이는 경향이 있음을 나타냅니다. 또한 `S3`와 `S4`는 **-0.73**의 강한 음의 상관 관계를 가집니다. 이는 `S3`이 증가하면 `S4`는 감소하는 경향이 있음을 의미합니다.

## Notebook 저장 및 Spark 세션 종료하기

이제 데이터 탐색을 마쳤으므로, 의미 있는 이름으로 Notebook을 저장하고 Spark 세션을 종료할 수 있습니다.

1.  Notebook 메뉴 바에서 ⚙️ **Settings** 아이콘을 사용하여 Notebook 설정을 봅니다.
2.  Notebook의 **Name**을 **Explore data for data science**로 설정한 다음 설정 창을 닫습니다.
3.  Notebook 메뉴에서 **Stop session**을 선택하여 Spark 세션을 종료합니다.

## 리소스 정리

이 실습에서는 데이터 탐색을 위해 Notebook을 만들고 사용했습니다. 또한 요약 통계를 계산하고 데이터의 패턴과 관계를 더 잘 이해하기 위한 시각화를 생성하는 코드를 실행했습니다.

모델 및 실험 탐색을 마쳤다면 이 실습을 위해 만든 Workspace를 삭제할 수 있습니다.

1.  왼쪽 막대에서 Workspace 아이콘을 선택하여 포함된 모든 항목을 봅니다.
2.  툴바의 **...** 메뉴에서 **Workspace settings**를 선택합니다.
3.  **General** 섹션에서 **Remove this workspace**를 선택합니다.
