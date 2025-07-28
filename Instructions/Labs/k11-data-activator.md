# Fabric에서 Data Activator 사용하기

> **IMPORTAN**T: 이 실습은 더 이상 사용되지 않으며, 곧 제거되거나 업데이트될 예정입니다. 현재 설명서는 정확하지 않으며 지원되지 않는 실습입니다.

Microsoft Fabric의 Data Activator는 데이터에서 발생하는 상황을 기반으로 조치를 취하는 기능입니다. Data Activator를 사용하면 데이터를 모니터링하고 데이터 변경에 대응하기 위한 트리거(Trigger)를 생성할 수 있습니다.

이 실습을 완료하는 데는 약 **30분**이 소요됩니다.

> **Note**: 이 실습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## Workspace 생성

Fabric에서 데이터를 다루기 전에, Fabric 평가판이 활성화된 Workspace(작업 영역)를 생성해야 합니다.

1.  [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com/home?experience=fabric) (`https://app.fabric.microsoft.com/home?experience=fabric`)에서 **Data Activator**를 선택합니다.
2.  왼쪽 메뉴 바에서 **Workspaces** (아이콘 모양 &#128455;)를 선택합니다.
3.  원하는 이름으로 새 Workspace를 만들고, Fabric 용량을 포함하는 라이선스 모드(*Trial*, *Premium*, 또는 *Fabric*)를 선택합니다.
4.  새 Workspace가 열리면 비어 있어야 합니다.

    ![Fabric의 비어 있는 Workspace 스크린샷](./Images/new-workspace.png)

이 실습에서는 Fabric의 Data Activator를 사용하여 `reflex`를 생성합니다. Data Activator는 기능 탐색에 사용할 수 있는 샘플 데이터셋을 편리하게 제공합니다. 이 샘플 데이터를 사용하여 실시간 데이터를 분석하고, 특정 조건이 충족될 때 이메일을 보내는 `trigger`를 만드는 `reflex`를 생성해 보겠습니다.

> **Note**: Data Activator 샘플 프로세스는 백그라운드에서 일부 임의의 데이터를 생성합니다. 조건과 필터가 복잡할수록 트리거되는 데 더 많은 시간이 걸립니다. 그래프에 데이터가 표시되지 않으면 몇 분간 기다린 후 페이지를 새로고침 하세요. 그렇지만, 실습을 계속하기 위해 그래프에 데이터가 표시될 때까지 기다릴 필요는 없습니다.

## 시나리오

이 시나리오에서 당신은 다양한 제품을 판매하고 배송하는 회사의 데이터 분석가입니다. Redmond 시로의 모든 배송 및 판매 데이터에 대한 책임이 있습니다. 배송 중인 소포를 모니터링하는 `reflex`를 만들고 싶습니다. 배송하는 제품 카테고리 중 하나는 운송 중 특정 온도로 냉장 보관해야 하는 의료 처방약입니다. 처방약이 포함된 소포의 온도가 특정 임계값보다 높거나 낮을 경우 배송 부서에 이메일을 보내는 `reflex`를 만들고자 합니다. 이상적인 온도는 33도에서 41도 사이여야 합니다. 기존 `reflex` 이벤트에 이미 유사한 `trigger`가 포함되어 있으므로, Redmond 시로 배송되는 소포에 대해 특별히 하나를 생성합니다. 시작해 봅시다!

## Reflex 생성

---

### **개념 설명: Reflex란 무엇일까요?**

`Reflex`는 Data Activator의 핵심 구성 요소로, 특정 비즈니스 시나리오에 대한 데이터 모니터링 및 대응 체계를 담는 컨테이너입니다. `Reflex` 안에는 다음과 같은 요소들이 포함됩니다.

*   **Data Streams (데이터 소스)**: 모니터링할 데이터가 어디서 오는지 정의합니다. (예: Power BI 보고서, Eventstream)
*   **Objects (객체)**: 추적하려는 실제 세상의 대상을 모델링합니다. (예: '소포', '장치', '고객')
*   **Triggers (트리거)**: 객체의 데이터가 특정 조건을 만족할 때 어떤 행동(Action)을 취할지 정의하는 규칙입니다.

하나의 `Reflex`는 특정 목표(예: '배송 온도 모니터링')를 위한 모든 논리와 데이터를 한 곳에서 관리하게 해주는 프로젝트와 같습니다.

---

1.  화면 우측 하단의 아이콘이 Data Activator를 나타내는지 확인하여 Data Activator 홈 화면에 있는지 확인합니다. **reflex (Preview)** 버튼을 선택하여 새 `reflex`를 생성해 봅시다.

    ![Data Activator 홈 화면 스크린샷](./Images/data-activator-home-screen.png)

2.  실제 운영 환경에서는 자체 데이터를 사용하겠지만, 이 실습에서는 Data Activator가 제공하는 샘플 데이터를 사용합니다. **Use Sample Data** 버튼을 선택하여 `reflex` 생성을 마칩니다.

    ![Data Activator Get Data 화면 스크린샷](./Images/data-activator-get-started.png)

3.  기본적으로 Data Activator는 *Reflex YYYY-MM-DD hh:mm:ss* 라는 이름으로 `reflex`를 생성합니다. Workspace에 여러 `reflex`가 있을 수 있으므로, 기본 이름을 더 설명적인 이름으로 변경해야 합니다. 좌측 상단의 현재 `reflex` 이름 옆에 있는 드롭다운 메뉴를 선택하고 이름을 예시에 맞게 ***Contoso Shipping Reflex***로 변경합니다.

    ![Data Activator reflex 홈 화면 스크린샷](./Images/data-activator-reflex-home-screen.png)

이제 `reflex`가 생성되었으므로 `trigger`와 `action`을 추가할 수 있습니다.

## Reflex 홈 화면 익히기

`Reflex`의 홈 화면은 *Design* 모드와 *Data* 모드의 두 섹션으로 나뉩니다. 화면 왼쪽 하단의 각 탭을 선택하여 모드를 전환할 수 있습니다. *Design* 모드 탭에서는 `trigger`, `property`, `event`를 사용하여 객체를 정의합니다. *Data* 모드 탭에서는 데이터 소스를 추가하고 `reflex`가 처리하는 데이터를 볼 수 있습니다. `Reflex`를 생성하면 기본적으로 열려 있는 *Design* 모드 탭부터 살펴보겠습니다.

### Design 모드

현재 *Design* 모드가 아니라면, 화면 왼쪽 하단의 **Design** 탭을 선택하세요.

![Data Activator reflex Design 모드 스크린샷](./Images/data-activator-design-tab.png)

*Design* 모드에 익숙해지기 위해 화면의 다른 섹션들, 즉 `trigger`, `property`, `event`를 선택해 보세요. 각 섹션은 다음 부분에서 더 자세히 다루겠습니다.

### Data 모드

현재 *Data* 모드가 아니라면, 화면 왼쪽 하단의 **Data** 탭을 선택하세요. 실제 환경에서는 여기에 EventStreams나 Power BI 시각적 개체로부터 데이터를 추가하게 됩니다. 이 실습에서는 Data Activator가 제공하는 샘플 데이터를 사용합니다. 이 샘플은 소포 배송 상태를 모니터링하는 세 개의 EventStreams로 이미 설정되어 있습니다.

![Data Activator reflex Data 모드 스크린샷](./Images/data-activator-data-tab.png)

각각의 다른 이벤트를 선택하고 스트림에서 사용되는 데이터를 관찰해 보세요.

![Data Activator reflex Data 모드 이벤트 스크린샷](./Images/data-activator-get-data-tab-event-2.png)

이제 `reflex`에 `trigger`를 추가할 차례입니다. 하지만 먼저, 새로운 `object`를 생성해 보겠습니다.

## Object 생성

---

### **개념 설명: Object란 무엇일까요?**

`Object`는 Data Activator에서 추적하고 모니터링하려는 실제 세상의 개별적인 대상을 의미합니다. 예를 들어, 이 시나리오에서는 '각각의 배송 소포'가 `object`가 됩니다. `Object`는 `PackageId`와 같은 고유한 키(Key)를 통해 식별되며, `Temperature`, `City` 등 여러 `property`(속성)를 가집니다.

Data Activator는 들어오는 데이터 스트림(`event`)을 보고 `PackageId`를 기준으로 "아, 이 데이터는 A 소포에 대한 것이구나", "이 데이터는 B 소포에 대한 것이구나" 와 같이 각 `object`의 상태를 개별적으로 추적합니다. 이렇게 함으로써 수천 개의 소포가 있더라도 각 소포의 온도에 대해 개별적인 `trigger`를 적용할 수 있습니다.

---

실제 시나리오에서는 Data Activator 샘플에 이미 *Package*라는 `object`가 포함되어 있으므로 새 `object`를 만들 필요가 없을 수도 있습니다. 하지만 이 실습에서는 `object` 생성 방법을 보여주기 위해 새로운 `object`를 만듭니다. *Redmond Packages*라는 새 `object`를 생성해 봅시다.

1.  현재 *Data* 모드가 아니라면, 화면 왼쪽 하단의 **Data** 탭을 선택하세요.

2.  ***Package In Transit*** 이벤트를 선택합니다. *PackageId*, *Temperature*, *ColdChainType*, *City*, *SpecialCare* 열의 값에 주목하세요. 이 열들을 사용하여 `trigger`를 생성할 것입니다.

3.  오른쪽에 *Assign your Data* 대화 상자가 아직 열려있지 않다면, 화면 오른쪽의 **Assign your data** 버튼을 선택합니다.

    ![Data Activator reflex Data 모드 assign your data 버튼 스크린샷](./Images/data-activator-data-tab-assign-data-button.png)

4.  *Assign your data* 대화 상자에서 ***Assign to new object*** 탭을 선택하고 다음 값을 입력합니다.

    *   **Object Name**: `Redmond Packages`
        *   **설명**: 우리가 추적할 대상의 이름입니다. 'Redmond로 가는 소포들'을 의미합니다.
    *   **Assign key column**: `PackageId`
        *   **설명**: 이것이 가장 중요한 설정입니다. `PackageId`는 각 소포를 고유하게 식별하는 키입니다. Data Activator는 이 키를 사용하여 수많은 소포 중 특정 소포의 상태(예: 온도)를 개별적으로 추적합니다.
    *   **Assign properties**: `City`, `ColdChainType`, `SpecialCare`, `Temperature`
        *   **설명**: 데이터 스트림에서 가져와 각 소포(`object`)에 연결할 속성들입니다. 이 속성값들은 `trigger`의 조건 설정에 사용됩니다.

    ![Data Activator reflex Data 모드 assign your data 대화 상자 스크린샷](./Images/data-activator-data-tab-assign-data.png)

5.  **Save**를 선택한 다음 **Save and go to design mode**를 선택합니다.

6.  이제 *Design* 모드로 돌아왔을 것입니다. ***Redmond Packages***라는 새 `object`가 추가되었습니다. 이 새 `object`를 선택하고, *Events*를 확장한 다음 **Package in Transit** 이벤트를 선택합니다.

    ![새 object가 추가된 Data Activator reflex Design 모드 스크린샷](./Images/data-activator-design-tab-new-object.png)

이제 `trigger`를 생성할 시간입니다.

## Trigger 생성

---

### **개념 설명: Trigger란 무엇일까요?**

`Trigger`는 Data Activator의 "만약 ~하면, ~하라" (If-Then) 규칙입니다. `Object`의 데이터(속성)를 지속적으로 감시하다가, 우리가 정의한 특정 조건이 충족되면 미리 설정된 `action`(행동)을 자동으로 실행합니다.

`Trigger`는 다음 요소로 구성됩니다.
*   **어떤 속성을 감시할 것인가?** (예: `Temperature`)
*   **어떤 조건이 충족되어야 하는가?** (예: 온도가 33도 미만 또는 41도 초과)
*   **어떤 추가 필터가 필요한가?** (예: `City`가 'Redmond'이고 `SpecialCare`가 'Medicine'인 경우에만)
*   **조건이 충족되면 무엇을 할 것인가?** (예: 이메일 보내기)

이러한 `trigger` 덕분에 수동으로 데이터를 확인할 필요 없이 중요한 비즈니스 이벤트에 자동으로 대응할 수 있습니다.

---

`trigger`가 무엇을 해야 하는지 다시 검토해 봅시다: *처방약이 포함된 소포의 온도가 특정 임계값보다 높거나 낮을 경우 배송 부서에 이메일을 보내는 `reflex`를 만들고 싶습니다. 이상적인 온도는 33도에서 41도 사이여야 합니다. 기존 `reflex` 이벤트에 이미 유사한 `trigger`가 포함되어 있으므로, Redmond 시로 배송되는 소포에 대해 특별히 하나를 생성합니다.*

1.  **Redmond Packages** `object`의 *Package In Transit* 이벤트 내에서 상단 메뉴의 **New Trigger** 버튼을 선택합니다. *Untitled*라는 기본 이름으로 새 `trigger`가 생성됩니다. `trigger`를 더 잘 정의하기 위해 이름을 ***Medicine temp out of range***로 변경합니다.

    ![Data Activator reflex Design에서 새 trigger 생성 스크린샷](./Images/data-activator-trigger-new.png)

2.  이제 `reflex`를 작동시킬 속성 또는 이벤트 열을 선택할 차례입니다. `Object`를 만들 때 여러 속성을 생성했으므로, **Existing property** 버튼을 선택하고 ***Temperature*** 속성을 선택합니다.

    ![Data Activator reflex Design에서 속성 선택 스크린샷](./Images/data-activator-trigger-select-property.png)

    이 속성을 선택하면 샘플 과거 온도 값이 있는 그래프가 표시됩니다.

    ![과거 값의 Data Activator 속성 그래프 스크린샷](./Images/data-activator-trigger-property-sample-graph.png)

3.  이제 이 속성에서 어떤 종류의 조건을 발생시킬지 결정해야 합니다. 이 경우, 온도가 41도 이상이거나 33도 미만일 때 `reflex`를 작동시키고 싶습니다. 숫자 범위를 찾고 있으므로 **Numeric** 버튼을 선택하고 **Exits range** 조건을 선택합니다.

    ![Data Activator reflex Design에서 조건 유형 선택 스크린샷](./Images/data-activator-trigger-select-condition-type.png)

4.  이제 조건에 대한 값을 입력해야 합니다. 범위 값으로 ***33***과 ***41***을 입력합니다. *exits numeric range* 조건을 선택했으므로, 온도가 *33*도 미만이거나 *41*도 이상일 때 `trigger`가 실행됩니다.

    ![Data Activator reflex Design에서 조건 값 입력 스크린샷](./Images/data-activator-trigger-select-condition-define.png)

5.  지금까지 `trigger`가 실행될 속성과 조건을 정의했지만, 아직 필요한 모든 매개변수가 포함되지는 않았습니다. `trigger`가 **Redmond** *시*와 **Medicine** *특별 관리* 유형에 대해서만 실행되도록 해야 합니다. 해당 조건에 대한 필터 두 개를 추가해 봅시다. **Add filter** 버튼을 선택하고, 속성을 ***City***로, 관계를 ***Equal***로 설정하고, 값으로 ***Redmond***를 입력합니다. 그런 다음 ***SpecialCare*** 속성으로 새 필터를 추가하고, ***Equal***로 설정한 후 값으로 ***Medicine***을 입력합니다.

    ![Data Activator reflex Design에서 필터 추가 스크린샷](./Images/data-activator-trigger-select-condition-add-filter.png)

6.  의약품이 냉장 보관되는지 확인하기 위해 필터를 하나 더 추가해 봅시다. **Add filter** 버튼을 선택하고, ***ColdChainType*** 속성을 설정하고, ***Equal***로 설정한 후 값으로 ***Refrigerated***를 입력합니다.

    ![Data Activator reflex Design에서 추가 필터 추가 스크린샷](./Images/data-activator-trigger-select-condition-add-filter-additional.png)

7.  거의 다 왔습니다! 이제 `trigger`가 실행될 때 어떤 조치를 취할지 정의하기만 하면 됩니다. 이 경우, 배송 부서에 이메일을 보내고 싶습니다. **Email** 버튼을 선택합니다.

    ![Data Activator에서 작업 추가 스크린샷](./Images/data-activator-trigger-select-action.png)

8.  이메일 작업에 다음 값을 입력합니다.

    *   **Send to**: 기본적으로 현재 사용자 계정이 선택되어 있어야 하며, 이 실습에서는 이대로 두어도 괜찮습니다.
    *   **Subject**: `Redmond Medical Package outside acceptable temperature range`
    *   **Headline**: `Temperature too high or too low`
    *   **Additional information**: 체크박스 목록에서 *Temperature* 속성을 선택합니다. 이렇게 하면 이메일 본문에 실시간 온도 값이 포함되어 수신자가 문제의 심각성을 바로 알 수 있습니다.

    ![Data Activator에서 작업 정의 스크린샷](./Images/data-activator-trigger-define-action.png)

9.  상단 메뉴에서 **Save**를 선택한 다음 **Start**를 선택합니다.

이제 Data Activator에서 `trigger`를 생성하고 시작했습니다.

## Trigger 업데이트 및 중지

이 `trigger`의 유일한 문제는 온도를 포함한 이메일을 보냈지만, 소포의 *PackageId*를 보내지 않았다는 점입니다. *PackageId*를 포함하도록 `trigger`를 업데이트해 봅시다.

1.  **Redmond Packages** `object`에서 **Packages in Transit** 이벤트를 선택하고 상단 메뉴에서 **New Property**를 선택합니다.

    ![Data Activator에서 object로부터 이벤트 선택 스크린샷](./Images/data-activator-trigger-select-event.png)

2.  *Packages in Transit* 이벤트에서 해당 열을 선택하여 **PackageId** 속성을 추가합시다. 속성 이름을 *Untitled*에서 *PackageId*로 변경하는 것을 잊지 마세요.

    ![Data Activator에서 속성 생성 스크린샷](./Images/data-activator-trigger-create-new-property.png)

3.  `trigger` 작업을 업데이트해 봅시다. **Medicine temp out of range** `trigger`를 선택하고, 하단의 **Act** 섹션으로 스크롤하여 **Additional information**을 선택하고 **PackageId** 속성을 추가합니다. 아직 **Save** 버튼을 누르지 마세요.

    ![Data Activator에서 trigger에 속성 추가 스크린샷](./Images/data-activator-trigger-add-property-existing-trigger.png)

4.  `trigger`를 업데이트했으므로, 올바른 조치는 `trigger`를 저장하는 것이 아니라 업데이트하는 것이어야 합니다. 하지만 이 실습에서는 그 반대로 **Update** 버튼 대신 **Save** 버튼을 선택하여 어떤 일이 일어나는지 보겠습니다.

    ---
    ### **코드 설명: Save vs. Update**
    Data Activator에서 `trigger`는 실시간으로 실행되는 '활성 인스턴스'입니다.
    *   **Save**: `trigger`의 *정의*만 저장합니다. 이미 실행 중인 활성 `trigger` 인스턴스에는 변경 사항이 즉시 적용되지 않습니다.
    *   **Update**: `trigger`의 정의를 저장하고, 동시에 실행 중인 활성 인스턴스에도 새로운 변경 사항을 즉시 적용합니다.
    실시간 모니터링 시스템에서는 `Update`를 사용하여 중단 없이 변경 사항을 반영하는 것이 일반적입니다.
    ---

    *Update* 버튼을 선택해야 하는 이유는 `trigger`를 업데이트할 때 `trigger`를 저장하고 현재 실행 중인 `trigger`를 새 조건으로 업데이트하기 때문입니다. *Save* 버튼만 선택하면, `trigger`를 업데이트하도록 선택할 때까지 현재 실행 중인 `trigger`는 새 조건을 적용하지 않습니다. 자, **Save** 버튼을 선택해 봅시다.

5.  *Update* 대신 *Save*를 선택했기 때문에, 화면 상단에 *There's a property update available. Update now to ensure the trigger has the most recent changes*라는 메시지가 나타나는 것을 확인했습니다. 이 메시지에는 *Update* 버튼도 함께 있습니다. 자, **Update** 버튼을 선택합시다.

    ![Data Activator에서 trigger 업데이트 스크린샷](./Images/data-activator-trigger-updated.png)

6.  상단 메뉴에서 **Stop** 버튼을 선택하여 `trigger`를 중지합니다.

## 리소스 정리

이 실습에서는 Data Activator에서 `trigger`가 있는 `reflex`를 생성했습니다. 이제 Data Activator 인터페이스와 `reflex` 및 그 `object`, `trigger`, `property`를 생성하는 방법에 익숙해졌을 것입니다.

Data Activator `reflex` 탐색을 마쳤다면, 이 실습을 위해 생성한 Workspace를 삭제할 수 있습니다.

1.  왼쪽 바에서 Workspace 아이콘을 선택하여 포함된 모든 항목을 봅니다.
2.  도구 모음의 **...** 메뉴에서 **Workspace settings**를 선택합니다.
3.  **General** 섹션에서 **Remove this workspace**를 선택합니다.
