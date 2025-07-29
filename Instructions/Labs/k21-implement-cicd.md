# Microsoft Fabric에서 배포 파이프라인 구현하기

Microsoft Fabric의 배포 파이프라인을 사용하면 개발, 테스트, 프로덕션과 같은 환경 간에 Fabric 항목의 콘텐츠 변경 사항을 복사하는 프로세스를 자동화할 수 있습니다. 배포 파이프라인을 사용하여 최종 사용자에게 도달하기 전에 콘텐츠를 개발하고 테스트할 수 있습니다. 이 실습에서는 배포 파이프라인을 생성하고, 파이프라인에 단계를 할당합니다. 그런 다음 개발 Workspace에 일부 콘텐츠를 생성하고 배포 파이프라인을 사용하여 개발, 테스트, 프로덕션 파이프라인 단계 간에 배포합니다.

> **Note**: 이 실습을 완료하려면 Fabric Workspace의 관리자(Admin) 역할의 멤버여야 합니다. 역할을 할당하려면 [Microsoft Fabric의 Workspace 역할](https://learn.microsoft.com/en-us/fabric/get-started/roles-workspaces)을 참조하세요.

이 실습을 완료하는 데는 약 **20**분이 소요됩니다.

---
### **개념 설명: Deployment Pipelines (배포 파이프라인)**

`Deployment pipeline`(배포 파이프라인)은 소프트웨어 개발 및 데이터 분석에서 CI/CD(지속적 통합/지속적 배포) 관행을 구현하기 위한 핵심 도구입니다. 그 목적은 콘텐츠(예: 보고서, 데이터세트, Lakehouse)를 한 환경에서 다른 환경으로 체계적이고 자동화된 방식으로 이동시켜, 수동 작업으로 인한 실수를 줄이고 배포 프로세스의 안정성을 높이는 것입니다.

일반적으로 다음과 같은 3단계 환경으로 구성됩니다.
1.  **Development (개발)**: 분석가와 개발자가 새로운 콘텐츠를 만들거나 기존 콘텐츠를 수정하는 작업 영역입니다. 이곳에서 모든 변경이 시작됩니다.
2.  **Test (테스트)**: 개발 환경에서 만들어진 콘텐츠가 이리로 배포됩니다. 여기서는 품질 보증(QA) 팀이나 소수의 사용자가 콘텐츠가 예상대로 작동하는지, 데이터가 정확한지, 성능에 문제가 없는지 등을 테스트합니다.
3.  **Production (프로덕션)**: 테스트를 통과한 콘텐츠가 최종적으로 배포되는 곳입니다. 이곳의 콘텐츠는 조직의 모든 최종 사용자가 실제로 사용하게 됩니다.

Fabric의 `Deployment pipeline`을 사용하면 버튼 클릭 몇 번으로 `Development`에서 `Test`로, `Test`에서 `Production`으로 콘텐츠를 원활하게 복사하고, 각 단계 간의 차이점을 시각적으로 비교할 수 있어 배포 과정을 안전하고 효율적으로 관리할 수 있습니다.

---

## Workspace 생성

Fabric 평가판이 활성화된 세 개의 Workspace를 생성합니다.

1.  브라우저에서 [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com/home?experience=fabric) (`https://app.fabric.microsoft.com/home?experience=fabric`)로 이동하여 Fabric 자격 증명으로 로그인합니다.
2.  왼쪽 메뉴 바에서 **Workspaces** (아이콘 모양 &#128455;)를 선택합니다.
3.  `Development`라는 이름의 새 Workspace를 만들고, Fabric 용량을 포함하는 라이선스 모드(*Trial*, *Premium*, 또는 *Fabric*)를 선택합니다.
4.  1 & 2단계를 반복하여 `Test`와 `Production`이라는 이름의 두 개의 Workspace를 더 만듭니다. 이제 당신의 Workspace는 `Development`, `Test`, `Production` 세 개입니다.
5.  왼쪽 메뉴 바에서 **Workspaces** 아이콘을 선택하고 `Development`, `Test`, `Production`이라는 이름의 세 개의 Workspace가 있는지 확인합니다.

> **Note**: Workspace에 고유한 이름을 입력하라는 메시지가 표시되면, `Development`, `Test` 또는 `Production` 단어에 하나 이상의 임의의 숫자를 추가하세요.

## Deployment pipeline 생성

다음으로, 배포 파이프라인을 생성합니다.

1.  왼쪽 메뉴 바에서 **Workspaces**를 선택합니다.
2.  **Deployment Pipelines**를 선택한 다음, **New pipeline**을 선택합니다.
3.  **Add a new deployment pipeline** 창에서 파이프라인에 고유한 이름을 지정하고 **Next**를 선택합니다.
4.  새 파이프라인 창에서 **Create and continue**를 선택합니다.

## 배포 파이프라인의 단계에 Workspace 할당하기

---
### **개념 설명: 단계(Stage)와 Workspace의 관계**

Fabric의 `Deployment pipeline`에서 각 단계(`Development`, `Test`, `Production`)는 반드시 하나의 전용 `Workspace`에 연결되어야 합니다. 즉, 파이프라인의 단계는 단순히 개념적인 구분이 아니라, 실제 콘텐츠가 저장되고 관리되는 물리적인 `Workspace`와 일대일로 매핑됩니다.

*   `Development` 단계 <- `Development` Workspace 할당
*   `Test` 단계 <- `Test` Workspace 할당
*   `Production` 단계 <- `Production` Workspace 할당

이렇게 각 단계를 별도의 Workspace에 할당함으로써 환경 간의 격리를 보장합니다. 개발 환경에서의 작업이 테스트나 프로덕션 환경에 직접적인 영향을 미치지 않으며, 배포 작업을 통해서만 콘텐츠가 단계적으로 이동할 수 있게 됩니다. 이것이 바로 안정적인 CI/CD 프로세스의 핵심입니다.

---

배포 파이프라인의 단계에 Workspace를 할당합니다.

1.  왼쪽 메뉴 바에서 생성한 파이프라인을 선택합니다.
2.  나타나는 창에서 각 배포 단계 아래의 **Assign a workspace** 옵션을 확장하고, 단계 이름과 일치하는 Workspace의 이름을 선택합니다.
3.  각 배포 단계에 대해 체크 표시 **Assign**을 선택합니다.

  ![배포 파이프라인 스크린샷](./Images/deployment-pipeline.png)

## 콘텐츠 생성

아직 Workspace에는 Fabric 항목이 생성되지 않았습니다. 다음으로, 개발 Workspace에 `lakehouse`를 생성합니다.

1.  왼쪽 메뉴 바에서 **Workspaces**를 선택합니다.
2.  **Development** Workspace를 선택합니다.
3.  **New Item**을 선택합니다.
4.  나타나는 창에서 **Lakehouse**를 선택하고, **New lakehouse window**에서 `lakehouse`의 이름을 **LabLakehouse**로 지정합니다.
5.  **Create**를 선택합니다.
6.  Lakehouse Explorer 창에서 **Start with sample data**를 선택하여 새로운 `lakehouse`에 데이터를 채웁니다.

  ![Lakehouse Explorer 스크린샷](./Images/lakehouse-explorer.png)

7.  **NYCTaxi** 샘플을 선택합니다.
8.  왼쪽 메뉴 바에서 생성한 파이프라인을 선택합니다.
9.  **Development** 단계를 선택하면, 배포 파이프라인 캔버스 아래에서 생성한 `lakehouse`를 단계 항목으로 볼 수 있습니다. **Test** 단계의 왼쪽 가장자리에는 원 안에 **X** 표시가 있습니다. **X** 표시는 `Development`와 `Test` 단계가 동기화되지 않았음을 나타냅니다.
10. **Test** 단계를 선택하면, 배포 파이프라인 캔버스 아래에서 생성한 `lakehouse`가 소스(이 경우 **Development** 단계를 의미함)에만 단계 항목으로 존재함을 볼 수 있습니다.

  ![단계 간 콘텐츠 불일치를 보여주는 배포 파이프라인 스크린샷](./Images/lab-pipeline-compare.png)

## 단계 간 콘텐츠 배포하기

---
### **개념 설명: 배포(Deploy)와 비교(Compare)**

`Deployment pipeline`의 가장 강력한 기능은 단계 간 콘텐츠를 **비교(Compare)** 하고 **배포(Deploy)** 하는 것입니다.

*   **비교(Compare)**: `Development` 단계와 `Test` 단계(또는 `Test`와 `Production`)를 비교하면, Fabric은 두 Workspace에 있는 항목들을 분석하여 차이점을 시각적으로 보여줍니다.
    *   **New**: 소스 단계(예: `Development`)에만 존재하는 새 항목.
    *   **Different**: 양쪽 단계에 모두 존재하지만 내용이 다른 항목.
    *   **Missing from**: 대상 단계(예: `Test`)에만 존재하는 항목.
    *   **X** 아이콘은 두 단계의 콘텐츠가 일치하지 않음을 나타냅니다.
*   **배포(Deploy)**: 이 버튼을 누르면 소스 단계의 선택된 항목들이 대상 단계로 **복사**됩니다. 이 작업을 통해 소스 환경의 변경 사항이 다음 환경으로 이전됩니다. 배포가 성공적으로 완료되면 두 단계의 콘텐츠가 동일해지며, 아이콘이 녹색 체크 표시(✓)로 변경되어 동기화되었음을 알려줍니다.

이 실습에서는 새로 만든 `Lakehouse`를 `Development`에서 `Test`로, 그리고 `Test`에서 `Production`으로 순차적으로 배포하는 과정을 경험합니다.
---

`Lakehouse`를 **Development** 단계에서 **Test** 및 **Production** 단계로 배포합니다.
1.  배포 파이프라인 캔버스에서 **Test** 단계를 선택합니다.
2.  배포 파이프라인 캔버스 아래에서 Lakehouse 항목 옆의 체크박스를 선택합니다. 그런 다음 **Deploy** 버튼을 선택하여 `lakehouse`의 현재 상태를 **Test** 단계로 복사합니다.
3.  나타나는 **Deploy to next stage** 창에서 **Deploy**를 선택합니다.
    이제 배포 파이프라인 캔버스의 `Production` 단계에 원 안의 X 표시가 있습니다. `lakehouse`는 `Development`와 `Test` 단계에는 존재하지만 아직 `Production` 단계에는 없습니다.
4.  배포 캔버스에서 **Production** 단계를 선택합니다.
5.  배포 파이프라인 캔버스 아래에서 Lakehouse 항목 옆의 체크박스를 선택합니다. 그런 다음 **Deploy** 버튼을 선택하여 `lakehouse`의 현재 상태를 **Production** 단계로 복사합니다.
6.  나타나는 **Deploy to next stage** 창에서 **Deploy**를 선택합니다. 이제 단계들 사이의 녹색 체크 표시는 모든 단계가 동기화되어 동일한 콘텐츠를 포함하고 있음을 나타냅니다.
7.  배포 파이프라인을 사용하여 단계 간에 배포하면 배포 단계에 해당하는 Workspace의 콘텐츠도 업데이트됩니다. 이를 확인해 봅시다.
8.  왼쪽 메뉴 바에서 **Workspaces**를 선택합니다.
9.  **Test** Workspace를 선택합니다. `lakehouse`가 그곳에 복사되었습니다.
10. 왼쪽 메뉴의 **Workspaces** 아이콘에서 **Production** Workspace를 엽니다. `lakehouse`가 `Production` Workspace에도 복사되었습니다.

## 정리

이 실습에서는 배포 파이프라인을 생성하고, 파이프라인에 단계를 할당했습니다. 그런 다음 개발 Workspace에 콘텐츠를 생성하고 배포 파이프라인을 사용하여 파이프라인 단계 간에 배포했습니다.

-   왼쪽 탐색 바에서 각 Workspace의 아이콘을 선택하여 포함된 모든 항목을 봅니다.
-   상단 도구 모음의 메뉴에서 **Workspace settings**를 선택합니다.
-   **General** 섹션에서 **Remove this workspace**를 선택합니다.
