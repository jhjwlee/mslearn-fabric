
# 핸즈온 랩: Microsoft Fabric에서 Dataflows (Gen2) 생성 및 사용하기

Microsoft Fabric에서 **Dataflow (Gen2)**는 다양한 데이터 소스에 연결하고 Power Query Online에서 데이터 변환을 수행하는 데 사용됩니다.

**개념 설명**: **Dataflow (Gen2)**는 Power BI 사용자에게 친숙한 Power Query 환경을 사용하여 클라우드에서 데이터를 추출, 변환, 적재(ETL)하는 재사용 가능한 구성 요소입니다. 코드를 작성하지 않고도 시각적인 인터페이스를 통해 데이터 정제 및 준비 작업을 정의할 수 있습니다. 이렇게 생성된 데이터 흐름은 Data Pipeline에서 데이터를 Lakehouse나 다른 분석 저장소로 수집하는 데 사용되거나, Power BI 보고서를 위한 데이터 세트를 정의하는 데 직접 사용될 수 있습니다.

이 랩은 Dataflow (Gen2)의 다양한 요소를 소개하기 위해 설계되었으며, 기업에서 존재할 수 있는 복잡한 솔루션을 만드는 것이 목적이 아닙니다. 이 실습을 완료하는 데 약 **30분**이 소요됩니다.

> **참고**: 이 실습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## Workspace 만들기

Fabric에서 데이터 작업을 시작하기 전에 Fabric 평가판이 활성화된 Workspace(작업 영역)를 만들어야 합니다.

1.  웹 브라우저에서 [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com/home?experience=fabric) `https://app.fabric.microsoft.com/home?experience=fabric`로 이동하여 Fabric 자격 증명으로 로그인합니다.
2.  왼쪽 메뉴 모음에서 **Workspaces** 아이콘( izgled: &#128455;)을 선택합니다.
3.  원하는 이름으로 새 Workspace를 만듭니다. 고급 섹션에서 Fabric 용량(*Trial*, *Premium*, 또는 *Fabric*)을 포함하는 라이선스 모드를 선택해야 합니다.
4.  새 Workspace가 열리면 처음에는 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷](./Images/new-workspace.png)

## Lakehouse 만들기

이제 Workspace가 준비되었으니, 데이터를 수집할 데이터 레이크하우스를 만들 차례입니다.

1.  왼쪽 메뉴 모음에서 **Create**를 선택합니다. *New* 페이지의 *Data Engineering* 섹션에서 **Lakehouse**를 선택하고, 원하는 고유한 이름을 지정합니다.
    > **참고**: **Create** 옵션이 사이드바에 고정되어 있지 않다면, 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    약 1분 후, 새로운 빈 Lakehouse가 생성됩니다.

 ![새로운 레이크하우스](./Images/new-lakehouse.png)

## 데이터 수집을 위한 Dataflow (Gen2) 만들기

이제 Lakehouse가 있으니 데이터를 수집해야 합니다. 이를 위한 한 가지 방법은 *추출, 변환, 적재* (ETL) 프로세스를 캡슐화하는 데이터 흐름(Dataflow)을 정의하는 것입니다.

**핸즈온의 의미**: 이 단계는 Fabric의 핵심 기능 중 하나인 Dataflow를 직접 생성하는 과정입니다. 사용자는 코딩 없이 시각적 인터페이스(Power Query)를 통해 원격에 있는 CSV 파일을 가져오고, 데이터를 변환하고, Lakehouse에 저장하는 전체 ETL 파이프라인을 구축하는 경험을 하게 됩니다.

1.  Lakehouse 홈페이지에서 **Get data** > **New Dataflow Gen2**를 선택합니다. 몇 초 후, 새 데이터 흐름을 위한 Power Query 편집기가 아래와 같이 열립니다.

 ![새로운 데이터 흐름](./Images/new-dataflow.png)

2.  **Import from a Text/CSV file**을 선택하고 다음 설정으로 새 데이터 소스를 만듭니다.
    - **Link to file**: *선택됨*
    - **File path or URL**: `https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/orders.csv`
    - **Connection**: Create new connection
    - **data gateway**: (none)
    - **Authentication kind**: Anonymous

3.  **Next**를 선택하여 파일 데이터를 미리 보고, **Create**를 눌러 데이터 소스를 생성합니다. Power Query 편집기는 아래와 같이 데이터 소스와 데이터를 형식화하기 위한 초기 쿼리 단계 세트를 보여줍니다.

 ![Power Query 편집기의 쿼리](./Images/power-query.png)

4.  도구 모음 리본에서 **Add column** 탭을 선택합니다. 그런 다음 **Custom column**을 선택하고 새 열을 만듭니다.

5.  *New column name*을 `MonthNo`로 설정하고, *Data type*을 **Whole Number**로 설정한 후, 다음 수식을 추가합니다: `Date.Month([OrderDate])` - 아래 그림과 같습니다:

    **코드 설명 (Power Query M 언어)**:
    - `Date.Month([OrderDate])`: 이것은 Power Query의 M 언어 함수입니다. `[OrderDate]` 열(Date 타입)에서 월(Month)에 해당하는 숫자(1~12)를 추출하여 반환합니다.

 ![Power Query 편집기의 사용자 지정 열](./Images/custom-column.png)

6.  **OK**를 선택하여 열을 생성하고 사용자 지정 열을 추가하는 단계가 쿼리에 추가되는 것을 확인합니다. 결과 열이 데이터 창에 표시됩니다.

 ![사용자 지정 열 단계가 추가된 쿼리](./Images/custom-column-added.png)

> **팁:** 오른쪽의 Query Settings 창에서 **Applied Steps**에 각 변환 단계가 포함된 것을 확인하세요. 하단에서 **Diagram view** 버튼을 토글하여 단계의 시각적 다이어그램을 켤 수도 있습니다.
>
> 단계는 위아래로 이동하거나, 기어 아이콘을 선택하여 편집할 수 있으며, 각 단계를 선택하여 미리보기 창에서 변환이 적용되는 것을 볼 수 있습니다.

7.  **OrderDate** 열의 데이터 유형이 **Date**로, 새로 생성된 **MonthNo** 열의 데이터 유형이 **Whole Number**로 설정되었는지 확인하고 확정합니다.

## Dataflow에 데이터 목적지 추가하기

**핸즈온의 의미**: 이 단계는 ETL 프로세스의 'L'(Load)에 해당합니다. Power Query에서 변환한 데이터를 최종적으로 어디에 저장할지 지정하는 과정입니다. 여기서는 앞서 만든 Lakehouse를 목적지로 설정하여, 데이터 파이프라인의 완성된 흐름을 구성합니다.

1.  도구 모음 리본에서 **Home** 탭을 선택합니다. 그런 다음 **Add data destination** 드롭다운 메뉴에서 **Lakehouse**를 선택합니다.
    > **참고:** 이 옵션이 회색으로 표시되면 이미 데이터 목적지가 설정되어 있을 수 있습니다. Power Query 편집기 오른쪽의 Query settings 창 하단에서 데이터 목적지를 확인하세요. 기본 목적지가 이미 설정된 경우, 이를 제거하고 새 목적지를 추가할 수 있습니다.

2.  **Connect to data destination** 대화 상자에서 연결을 편집하고 Power BI 조직 계정을 사용하여 로그인하여 데이터 흐름이 레이크하우스에 액세스하는 데 사용하는 ID를 설정합니다.

 ![데이터 목적지 구성 페이지](./Images/dataflow-connection.png)

3.  **Next**를 선택하고 사용 가능한 작업 영역 목록에서 자신의 작업 영역을 찾아 이 실습 시작 시 만든 레이크하우스를 선택합니다. 그런 다음 **orders**라는 새 테이블을 지정합니다.

   ![데이터 목적지 구성 페이지](./Images/data-destination-target.png)

4.  **Next**를 선택하고 **Choose destination settings** 페이지에서 **Use automatic settings** 옵션을 비활성화하고 **Append**를 선택한 다음 **Save settings**를 선택합니다.
    > **참고:** 데이터 유형 업데이트는 *Power Query* 편집기를 사용하는 것을 권장하지만, 원한다면 이 페이지에서도 할 수 있습니다.

    ![데이터 목적지 설정 페이지](./Images/destination-settings.png)

5.  메뉴 바에서 **View**를 열고 **Diagram view**를 선택합니다. Power Query 편집기의 쿼리에서 **Lakehouse** 목적지가 아이콘으로 표시되는 것을 확인하세요.

   ![레이크하우스 목적지가 있는 쿼리](./Images/lakehouse-destination.png)

6.  도구 모음 리본에서 **Home** 탭을 선택합니다. 그런 다음 **Save & run**을 선택하고 **Dataflow 1** 데이터 흐름이 작업 영역에 생성될 때까지 기다립니다.

## 파이프라인에 데이터 흐름 추가하기

데이터 흐름을 파이프라인의 활동(activity)으로 포함할 수 있습니다.

**개념 설명**: **Data Pipeline**은 데이터 수집 및 처리 활동을 조율(orchestrate)하는 데 사용됩니다. Dataflow가 특정 ETL 작업을 수행하는 '일꾼'이라면, Data Pipeline은 이 일꾼(Dataflow)과 다른 종류의 작업(예: Notebook 실행, Stored Procedure 호출 등)들을 정해진 순서와 스케줄에 따라 실행시키는 '감독관' 역할을 합니다.

1.  Fabric 지원 작업 영역에서 **+ New item** > **Data pipeline**을 선택한 다음, 메시지가 표시되면 **Load data**라는 새 파이프라인을 만듭니다.
    파이프라인 편집기가 열립니다.
    > **팁**: Copy Data 마법사가 자동으로 열리면 닫으세요!

2.  **Pipeline activity**를 선택하고, **Dataflow** 활동을 파이프라인에 추가합니다.
3.  새로운 **Dataflow1** 활동이 선택된 상태에서, **Settings** 탭의 **Dataflow** 드롭다운 목록에서 **Dataflow 1**(이전에 만든 데이터 흐름)을 선택합니다.

   ![데이터 흐름 활동이 있는 파이프라인](./Images/dataflow-activity.png)

4.  **Home** 탭에서 **&#128427;** (*Save*) 아이콘을 사용하여 파이프라인을 저장합니다.
5.  **&#9655; Run** 버튼을 사용하여 파이프라인을 실행하고 완료될 때까지 기다립니다. 몇 분 정도 걸릴 수 있습니다.

   ![성공적으로 완료된 데이터 흐름이 있는 파이프라인](./Images/dataflow-pipeline-succeeded.png)

6.  왼쪽 가장자리 메뉴 모음에서 자신의 Lakehouse를 선택합니다.
7.  **Tables**의 **...** 메뉴에서 **refresh**를 선택합니다. 그런 다음 **Tables**를 확장하고 데이터 흐름에 의해 생성된 **orders** 테이블을 선택합니다.

   ![데이터 흐름에 의해 로드된 테이블](./Images/loaded-table.png)



## 리소스 정리

Microsoft Fabric에서 데이터 흐름 탐색을 마쳤다면 이 실습을 위해 만든 Workspace를 삭제할 수 있습니다.

1.  브라우저에서 Microsoft Fabric으로 이동합니다.
2.  왼쪽 막대에서 Workspace 아이콘을 선택하여 포함된 모든 항목을 봅니다.
3.  **Workspace settings**를 선택하고 **General** 섹션에서 아래로 스크롤하여 **Remove this workspace**를 선택합니다.
4.  **Delete**를 선택하여 Workspace를 삭제합니다.
