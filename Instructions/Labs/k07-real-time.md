Microsoft Fabric 전문가가 번역하고 설명하는 핸즈온 가이드입니다.

# 핸즈온 랩: Microsoft Fabric에서 Real-Time Intelligence 시작하기

Microsoft Fabric은 실시간 데이터 스트림을 위한 분석 솔루션을 생성할 수 있는 **Real-Time Intelligence** 기능을 제공합니다. 이 실습에서는 Microsoft Fabric의 Real-Time Intelligence 기능을 사용하여 실시간 주식 시장 데이터 스트림을 수집, 분석 및 시각화합니다.

이 실습을 완료하는 데 약 **30**분이 소요됩니다.

> **참고**: 이 실습을 완료하려면 [Microsoft Fabric 테넌트](https://learn.microsoft.com/fabric/get-started/fabric-trial)가 필요합니다.

## Workspace 만들기

Fabric에서 데이터 작업을 시작하기 전에 Fabric 용량이 활성화된 Workspace를 만들어야 합니다.

1.  웹 브라우저에서 [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com/home?experience=fabric) `https://app.fabric.microsoft.com/home?experience=fabric`로 이동하여 Fabric 자격 증명으로 로그인합니다.
2.  왼쪽 메뉴 모음에서 **Workspaces** 아이콘( izgled: &#128455;)을 선택합니다.
3.  원하는 이름으로 새 Workspace를 만듭니다. **Advanced** 섹션에서 Fabric 용량(*Trial*, *Premium*, 또는 *Fabric*)을 포함하는 라이선스 모드를 선택해야 합니다.
4.  새 Workspace가 열리면 처음에는 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷](./Images/new-workspace.png)

## Eventstream 만들기

이제 스트리밍 소스에서 실시간 데이터를 찾아 수집할 준비가 되었습니다. 이를 위해 Fabric **Real-Time Hub**에서 시작합니다.

**개념 설명: Real-Time Intelligence**
- **Real-Time Hub**: 조직 내에서 사용 가능한 모든 실시간 데이터 스트림(예: IoT 센서 데이터, 로그 파일, 주식 시세 등)을 발견하고 관리할 수 있는 중앙 허브입니다.
- **Eventstream**: 지속적으로 발생하는 이벤트(데이터)의 흐름을 나타냅니다. Eventstream은 하나 이상의 데이터 소스(Source)와 하나 이상의 데이터 목적지(Destination)를 연결하는 파이프라인 역할을 합니다. 소스에서 들어온 데이터를 실시간으로 변환하여 목적지로 보낼 수 있습니다.

1.  왼쪽 메뉴 모음에서 **Real-Time** 허브를 선택합니다.
2.  Real-Time Hub의 **Connect to** 섹션에서 **Data sources**를 선택합니다.
3.  **Stock market** 샘플 데이터 소스를 찾아 **Connect**를 선택합니다. 그런 다음 **Connect** 마법사에서 소스 이름을 `stock`으로 지정하고 기본 이벤트스트림 이름을 `stock-data`로 변경합니다. 이 데이터와 관련된 기본 스트림은 자동으로 `stock-data-stream`으로 이름이 지정됩니다.

    ![새 이벤트스트림 스크린샷](./Images/name-eventstream.png)

4.  **Next**를 선택하고 소스와 이벤트스트림이 생성될 때까지 기다린 다음, **Open eventstream**을 선택합니다. 이벤트스트림은 디자인 캔버스에 **stock** 소스와 **stock-data-stream**을 표시합니다.

   ![이벤트스트림 캔버스 스크린샷](./Images/new-stock-stream.png)

## Eventhouse 만들기

이벤트스트림은 실시간 주식 데이터를 수집하지만, 현재는 그 데이터를 가지고 아무것도 하지 않습니다. 캡처된 데이터를 테이블에 저장할 수 있는 **Eventhouse**를 만들어 보겠습니다.

**개념 설명: Eventhouse**
- **Eventhouse**: 시계열 데이터(시간의 흐름에 따라 기록된 데이터)를 저장하고 분석하는 데 최적화된 데이터베이스입니다. 내부적으로는 Kusto Query Language(KQL)를 사용하는 고성능 분석 데이터베이스인 **KQL Database**를 기반으로 합니다. Eventhouse는 실시간 스트리밍 데이터를 효율적으로 수집하고, 저장된 데이터를 KQL을 사용하여 매우 빠르게 쿼리할 수 있도록 설계되었습니다.

1.  왼쪽 메뉴 모음에서 **Create**를 선택합니다. *New* 페이지의 *Real-Time Inteligence* 섹션에서 **Eventhouse**를 선택하고, 원하는 고유한 이름을 지정합니다.
2.  생성된 Eventhouse를 보면, Eventhouse와 동일한 이름의 **KQL database**가 포함되어 있는 것을 확인할 수 있습니다.
3.  데이터베이스를 선택하면, 관련 *queryset*이 있는 것을 확인할 수 있습니다. 이 파일에는 데이터베이스의 테이블을 쿼리하는 데 사용할 수 있는 몇 가지 샘플 KQL 쿼리가 포함되어 있습니다.

    하지만 현재는 쿼리할 테이블이 없습니다. 이 문제를 해결하기 위해 이벤트스트림에서 새 테이블로 데이터를 가져와 보겠습니다.

4.  KQL 데이터베이스의 메인 페이지에서 **Get data**를 선택합니다.
5.  데이터 소스로 **Eventstream** > **Existing eventstream**을 선택합니다.
6.  **Select or create a destination table** 창에서 `stock`이라는 새 테이블을 만듭니다. 그런 다음 **Configure the data source** 창에서 Workspace와 **stock-data** 이벤트스트림을 선택하고 연결 이름을 `stock-table`로 지정합니다.

   ![이벤트스트림에서 테이블을 로드하기 위한 구성 스크린샷](./Images/configure-destination.png)

7.  **Next** 버튼을 사용하여 데이터 검사 단계를 완료하고 구성을 마칩니다. 그런 다음 구성 창을 닫아 `stock` 테이블이 있는 Eventhouse를 확인합니다.

   ![테이블이 있는 이벤트하우스 스크린샷](./Images/eventhouse-with-table.png)

    스트림과 테이블 간의 연결이 생성되었습니다. 이벤트스트림에서 이를 확인해 보겠습니다.

8.  왼쪽 메뉴 모음에서 **Real-Time** 허브를 선택한 다음 **My data streams** 페이지를 봅니다. **stock-data-stream** 스트림의 **…** 메뉴에서 **Open eventstream**을 선택합니다.

    이제 이벤트스트림에 스트림의 목적지(Destination)가 표시됩니다.

   ![목적지가 있는 이벤트스트림 스크린샷](./Images/eventstream-destination.png)

    이 실습에서는 실시간 데이터를 캡처하여 테이블에 로드하는 매우 간단한 이벤트스트림을 만들었습니다. 실제 솔루션에서는 일반적으로 시간적 창(temporal window)에 대해 데이터를 집계하는 변환(예: 5분 단위로 각 주식의 평균 가격을 캡처)을 추가합니다.

    이제 캡처된 데이터를 쿼리하고 분석하는 방법을 살펴보겠습니다.

## 캡처된 데이터 쿼리하기

**개념 설명: Kusto Query Language (KQL)**
KQL은 시계열 데이터를 탐색하고 패턴을 분석하는 데 최적화된 강력하고 직관적인 쿼리 언어입니다. SQL과 유사한 점도 있지만, 파이프(`|`) 문자를 사용하여 데이터를 한 단계에서 다음 단계로 넘겨가며 순차적으로 처리하는 파이프라인 방식의 구문이 특징입니다.

**핸즈온의 의미**: 이 단계에서는 KQL을 사용하여 실시간으로 Eventhouse에 쌓이는 데이터를 직접 쿼리해 봅니다. `take`를 사용하여 간단히 데이터를 확인하고, `ago()`와 `summarize` 같은 KQL의 강력한 시계열 함수를 사용하여 최근 5분간의 평균 가격을 계산하는 분석 쿼리를 작성하는 과정을 체험합니다.

1.  왼쪽 메뉴 모음에서 Eventhouse 데이터베이스를 선택합니다.
2.  데이터베이스의 *queryset*을 선택합니다.
3.  쿼리 창에서 첫 번째 예제 쿼리를 다음과 같이 수정합니다.

    ```kql
    stock
    | take 100
    ```
    **코드 설명**: `stock` 테이블에서 100개의 행을 임의로 가져와서 보여줍니다.

4.  쿼리 코드를 선택하고 실행하여 테이블에서 100개의 데이터 행을 봅니다.

    ![KQL 쿼리 스크린샷](./Images/kql-stock-query.png)

5.  결과를 검토한 다음, 지난 5분 동안의 각 주식 기호에 대한 평균 가격을 검색하도록 쿼리를 수정합니다.

    ```kql
    stock
    | where ["time"] > ago(5m)
    | summarize avgPrice = avg(todecimal(bidPrice)) by symbol
    | project symbol, avgPrice
    ```
    **코드 설명**:
    - `| where ["time"] > ago(5m)`: `time` 열의 값이 지금으로부터 5분 전(`ago(5m)`)보다 최신인 데이터만 필터링합니다.
    - `| summarize avgPrice = avg(todecimal(bidPrice)) by symbol`: `bidPrice`의 평균을 계산하여 `avgPrice`라는 새 열을 만들고, 이 계산을 `symbol`(주식 기호)별로 그룹화하여 수행합니다.
    - `| project symbol, avgPrice`: 최종 결과에 `symbol`과 `avgPrice` 열만 표시하도록 선택합니다.

6.  수정된 쿼리를 강조 표시하고 실행하여 결과를 봅니다.
7.  몇 초 기다렸다가 다시 실행하면 실시간 스트림에서 새 데이터가 테이블에 추가됨에 따라 평균 가격이 변경되는 것을 확인할 수 있습니다.

## 실시간 대시보드 만들기

이제 데이터 스트림에 의해 채워지는 테이블이 있으므로, 실시간 대시보드를 사용하여 데이터를 시각화할 수 있습니다.

**핸즈온의 의미**: 이 단계에서는 분석 쿼리 결과를 정적인 테이블이 아닌, 실시간으로 자동 업데이트되는 동적인 시각화 차트로 만드는 과정을 경험합니다. 이를 통해 실시간 데이터의 변화를 직관적으로 모니터링하는 대시보드를 쉽게 구축할 수 있음을 확인합니다.

1.  쿼리 편집기에서 지난 5분 동안의 평균 주가를 검색하는 데 사용한 KQL 쿼리를 선택합니다.
2.  툴바에서 **Pin to dashboard**를 선택합니다. 그런 다음 쿼리를 다음 설정으로 **in a new dashboard**에 고정합니다.
    - **Dashboard name**: `Stock Dashboard`
    - **Tile name**: `Average Prices`
3.  대시보드를 만들고 엽니다. 다음과 같이 보여야 합니다.

    ![새 대시보드 스크린샷](./Images/stock-dashboard-table.png)

4.  대시보드 상단에서 **Viewing** 모드에서 **Editing** 모드로 전환합니다.
5.  **Average Prices** 타일의 **Edit** (*연필*) 아이콘을 선택합니다.
6.  **Visual formatting** 창에서 **Visual**을 *Table*에서 *Column chart*로 변경합니다.
7.  대시보드 상단에서 **Apply changes**를 선택하고 수정된 대시보드를 봅니다.

    ![차트 타일이 있는 대시보드 스크린샷](./Images/stock-dashboard-chart.png)

    이제 실시간 주식 데이터의 라이브 시각화가 준비되었습니다.

## 알림(Alert) 만들기

**개념 설명: Activator**
Microsoft Fabric의 Real-Time Intelligence에는 **Activator**라는 기술이 포함되어 있으며, 이는 실시간 이벤트를 기반으로 특정 작업(Action)을 트리거할 수 있습니다. 예를 들어, 특정 센서 값이 임계치를 초과하거나, 특정 단어가 포함된 로그가 감지될 때 이메일을 보내거나 다른 시스템을 호출하는 등의 자동화된 대응을 설정할 수 있습니다.

1.  주가 시각화가 포함된 대시보드 창의 툴바에서 **Set alert**를 선택합니다.
2.  **Set alert** 창에서 다음 설정으로 알림을 만듭니다.
    - **Run query every**: 5 minutes
    - **Check**: On each event grouped by
    - **Grouping field**: symbol
    - **When**: avgPrice
    - **Condition**: Increases by
    - **Value**: 100
    - **Action**: Send me an email
    - **Save location**: *새 항목으로 고유한 이름 지정*

    ![알림 설정 스크린샷](./Images/configure-activator.png)
    **설정 설명**: 이 설정은 5분마다 쿼리를 실행하여, 각 주식 기호(`symbol`)별로 `avgPrice`가 이전 값보다 `100` 이상 증가하면 나에게 이메일을 보내도록 구성합니다.

3.  알림을 만들고 저장될 때까지 기다립니다. 그런 다음 생성되었음을 확인하는 창을 닫습니다.
4.  왼쪽 메뉴 모음에서 Workspace 페이지를 선택합니다.
5.  Workspace 페이지에서 이 연습에서 만든 항목들, 특히 알림에 대한 **Activator**가 포함되어 있는지 확인합니다.
6.  Activator를 열고, **avgPrice** 노드 아래에서 알림의 고유 식별자를 선택합니다. 그런 다음 **History** 탭을 봅니다.

    알림이 트리거되지 않았을 수 있으며, 이 경우 기록에는 데이터가 없습니다. 평균 주가가 100 이상 변경되면 Activator가 이메일을 보내고 알림이 기록에 기록됩니다.

## 리소스 정리

이 실습에서는 Eventhouse를 생성하고, Eventstream을 사용하여 실시간 데이터를 수집하고, KQL 데이터베이스 테이블에서 수집된 데이터를 쿼리하고, 실시간 데이터를 시각화하는 실시간 대시보드를 만들고, Activator를 사용하여 알림을 구성했습니다.

Fabric에서 Real-Time Intelligence 탐색을 마쳤다면 이 실습을 위해 만든 Workspace를 삭제할 수 있습니다.

1.  왼쪽 막대에서 Workspace 아이콘을 선택합니다.
2.  툴바에서 **Workspace settings**를 선택합니다.
3.  **General** 섹션에서 **Remove this workspace**를 선택합니다.
