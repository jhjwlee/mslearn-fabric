# Microsoft Fabric에서 데이터 액세스 보안 설정하기

Microsoft Fabric은 데이터 액세스 관리를 위한 다계층 보안 모델을 가지고 있습니다. 보안은 전체 Workspace, 개별 Item 또는 각 Fabric 엔진의 세분화된 권한을 통해 설정할 수 있습니다. 이 실습에서는 Workspace 및 Item 액세스 제어와 OneLake 데이터 액세스 역할을 사용하여 데이터를 보호합니다.

이 실습을 완료하는 데는 약 **45**분이 소요됩니다.

## Workspace 생성

Fabric에서 데이터를 다루기 전에, Fabric 평가판이 활성화된 Workspace(작업 영역)를 생성해야 합니다.

1.  브라우저에서 [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com/home?experience=fabric) (`https://app.fabric.microsoft.com/home?experience=fabric`)로 이동하여 Fabric 자격 증명으로 로그인합니다.
2.  왼쪽 메뉴 바에서 **Workspaces** (아이콘 모양 &#128455;)를 선택합니다.
3.  원하는 이름으로 새 Workspace를 만들고, Fabric 용량을 포함하는 라이선스 모드(*Trial*, *Premium*, 또는 *Fabric*)를 선택합니다.
4.  새 Workspace가 열리면 비어 있어야 합니다.

    ![Fabric의 비어 있는 Workspace 스크린샷](./Images/new-empty-workspace.png)

> **Note**: Workspace를 생성하면 자동으로 **Workspace Admin** 역할의 멤버가 됩니다.

## Data Warehouse 생성

다음으로, 생성한 Workspace에 Data Warehouse를 생성합니다.

1.  **+ New Item**을 클릭합니다. *New item* 페이지의 *Store Data* 섹션에서 **Sample warehouse**를 선택하고 원하는 이름으로 새 Data Warehouse를 생성합니다.

     약 1분 후, 새로운 Warehouse가 생성됩니다.

    ![새로운 warehouse 스크린샷](./Images/new-sample-warehouse.png)

## Lakehouse 생성
다음으로, 생성한 Workspace에 Lakehouse를 생성합니다.

1.  왼쪽 메뉴 바에서 **Workspaces** (아이콘 모양 🗇)를 선택합니다.
2.  생성한 Workspace를 선택합니다.
3.  Workspace에서 **+ New Item** 버튼을 선택한 다음 **Lakehouse**를 선택합니다. 원하는 이름으로 새 Lakehouse를 생성합니다.

   약 1분 후, 새로운 Lakehouse가 생성됩니다.

    ![Fabric의 새로운 lakehouse 스크린샷](./Images/new-sample-lakehouse.png)

4.  **Start with sample data** 타일을 선택한 다음 **Public holidays** 샘플을 선택합니다. 약 1분 후, Lakehouse에 데이터가 채워집니다.

## Workspace 액세스 제어 적용

---
### **개념 설명: Workspace Roles (작업 영역 역할)**

Workspace 역할은 Fabric 보안의 가장 넓은 범위의 제어 계층입니다. 이는 특정 `Workspace`와 그 안의 모든 콘텐츠(Lakehouse, Warehouse, Report 등)에 대한 접근을 제어합니다. 사용자를 Workspace 역할에 할당하는 것은 마치 건물 전체의 마스터 키를 주는 것과 같습니다. 주요 역할은 다음과 같습니다.

*   **Admin**: Workspace 관리, 사용자 추가/제거, 콘텐츠 삭제 등 모든 권한을 가집니다.
*   **Member**: Admin과 거의 동일한 권한을 가지지만, Workspace를 삭제할 수는 없습니다. 콘텐츠를 게시하고 공유할 수 있습니다.
*   **Contributor**: Workspace의 콘텐츠를 생성, 편집, 삭제할 수 있지만, 앱을 게시하거나 권한을 관리할 수는 없습니다.
*   **Viewer**: Workspace 내의 모든 항목을 볼 수만 있고(읽기 전용), 수정하거나 변경할 수는 없습니다. 이 역할은 보고서를 소비하는 사용자에게 적합합니다.

이 실습에서는 **Viewer** 역할을 사용하여 사용자가 Workspace 내의 모든 항목에 대한 읽기 권한을 어떻게 획득하는지 확인합니다.
---

이 실습에서는 사용자를 Workspace 역할에 추가하고, 권한을 적용하며, 각 권한 세트가 적용될 때 무엇을 볼 수 있는지 확인합니다. 두 개의 브라우저를 열고 다른 사용자로 로그인합니다. 한 브라우저에서는 **Workspace Admin**이 되고, 다른 브라우저에서는 권한이 적은 두 번째 사용자로 로그인합니다. 한 브라우저에서 Workspace Admin이 두 번째 사용자의 권한을 변경하면, 두 번째 브라우저에서 권한 변경의 효과를 확인할 수 있습니다.

1.  왼쪽 메뉴 바에서 **Workspaces** (아이콘 모양 &#128455;)를 선택합니다.
2.  생성한 Workspace를 선택합니다.
3.  화면 상단의 **Manage access**를 선택합니다.

> **Note**: 당신이 Workspace를 생성했기 때문에 현재 로그인한 사용자가 **Workspace Admin** 역할의 멤버로 표시됩니다. 아직 다른 사용자는 Workspace에 할당되지 않았습니다.

4.  다음으로, Workspace에 대한 권한이 없는 사용자가 무엇을 볼 수 있는지 확인합니다. 브라우저에서 InPrivate 창을 엽니다. Microsoft Edge 브라우저에서는 오른쪽 상단의 줄임표를 선택하고 **New InPrivate Window**를 선택합니다.
5.  https://app.fabric.microsoft.com을 입력하고 테스트에 사용할 두 번째 사용자로 로그인합니다.
6.  화면 왼쪽 하단에서 **Microsoft Fabric**을 선택한 다음 **Data Warehouse**를 선택합니다. 다음으로 **Workspaces** (아이콘 모양 &#128455;)를 선택합니다.

> **Note:** 두 번째 사용자는 Workspace에 대한 접근 권한이 없으므로 해당 Workspace가 보이지 않습니다.

7.  다음으로, 두 번째 사용자에게 **Workspace Viewer** 역할을 할당하고, 이 역할이 Workspace 내 Warehouse의 테이블에 대한 읽기 접근 권한을 부여하는 것을 확인합니다.
8.  Workspace Admin으로 로그인한 브라우저 창으로 돌아갑니다. 생성한 Workspace를 보여주는 페이지에 있는지 확인합니다. 페이지 하단에 새로운 Workspace 항목들과 샘플 Warehouse, Lakehouse가 나열되어 있어야 합니다.
9.  화면 오른쪽 상단의 **Manage access**를 선택합니다.
10. **Add people or groups**를 선택합니다. 테스트에 사용하는 두 번째 사용자의 이메일을 입력합니다. **Add**를 선택하여 사용자를 Workspace **Viewer** 역할에 할당합니다.
11. 두 번째 사용자로 로그인한 InPrivate 브라우저 창으로 돌아가 브라우저의 새로고침 버튼을 눌러 두 번째 사용자에게 할당된 세션 권한을 새로고침합니다.
12. 왼쪽 메뉴 바에서 **Workspaces** 아이콘(아이콘 모양 &#128455;)을 선택하고 Workspace Admin 사용자로 생성한 Workspace 이름을 선택합니다. 두 번째 사용자는 **Workspace Viewer** 역할을 할당받았기 때문에 이제 Workspace의 모든 항목을 볼 수 있습니다.

    ![Fabric의 Workspace 항목 스크린샷](./Images/workspace-viewer-view.png)

13. Warehouse를 선택하여 엽니다.
14. **Date** 테이블을 선택하고 행이 로드될 때까지 기다립니다. Workspace Viewer 역할의 멤버로서 Warehouse의 테이블에 대한 CONNECT 및 ReadData 권한이 있으므로 행을 볼 수 있습니다. Workspace Viewer 역할에 부여된 권한에 대한 자세한 내용은 [Workspace roles](https://learn.microsoft.com/en-us/fabric/data-warehouse/workspace-roles)를 참조하십시오.
15. 다음으로, 왼쪽 메뉴 바에서 **Workspaces** 아이콘을 선택한 다음, Lakehouse를 선택합니다.
16. Lakehouse가 열리면, 화면 오른쪽 상단에 **Lakehouse**라고 표시된 드롭다운 상자를 클릭하고 **SQL analytics endpoint**를 선택합니다.
17. **publicholidays** 테이블을 선택하고 데이터가 표시될 때까지 기다립니다. 사용자가 SQL analytics endpoint에 대한 읽기 권한을 부여하는 Workspace Viewer 역할의 멤버이기 때문에 Lakehouse 테이블의 데이터를 읽을 수 있습니다.

## Item 액세스 제어 적용

---
### **개념 설명: Item Permissions (항목 권한)**

Item 권한은 Workspace 내의 개별 Fabric 항목(Warehouse, Lakehouse, 보고서 등)에 대한 접근을 제어합니다. 이는 Workspace 역할보다 더 세분화된 제어 방식입니다. 예를 들어, 사용자에게 Workspace 전체에 대한 접근 권한을 주지 않고, 특정 Warehouse 하나에만 접근하도록 허용하고 싶을 때 사용합니다.

*   **공유(Share)**: 특정 항목을 다른 사용자와 공유하여 읽기(Read), 다시 공유하기(Reshare), 빌드(Build) 등의 권한을 부여할 수 있습니다.
*   **세부 권한 관리(Manage permissions)**: `ReadData`(SQL을 통해 데이터 읽기), `Read all`(OneLake에서 모든 파일 데이터 읽기) 등 더 구체적인 권한을 부여할 수 있습니다.

이 방법은 '건물 전체 키(Workspace 역할)' 대신 '특정 방 하나만의 키(Item 권한)'를 주는 것과 같아서, 최소 권한 원칙을 지키는 데 매우 유용합니다.
---

이 실습에서는 이전 실습에서 적용한 **Workspace Viewer** 권한을 제거한 다음, 권한이 적은 사용자가 Lakehouse 데이터가 아닌 Warehouse 데이터만 볼 수 있도록 Warehouse에 Item 수준 권한을 적용합니다.

1.  Workspace Admin으로 로그인한 브라우저 창으로 돌아갑니다. 왼쪽 탐색 창에서 **Workspaces**를 선택합니다.
2.  생성한 Workspace를 선택하여 엽니다.
3.  화면 상단에서 **Manage access**를 선택합니다.
4.  두 번째 사용자 이름 아래의 **Viewer** 단어를 선택합니다. 나타나는 메뉴에서 **Remove**를 선택합니다.

   ![Fabric의 Workspace 액세스 드롭다운 스크린샷](./Images/workspace-access.png)

5.  **Manage access** 섹션을 닫습니다.
6.  Workspace에서 Warehouse 이름 위로 마우스를 가져가면 줄임표(**...**)가 나타납니다. 줄임표를 선택하고 **Manage permissions**를 선택합니다.

7.  **Add user**를 선택하고 두 번째 사용자의 이름을 입력합니다.
8.  나타나는 상자의 **Additional permissions** 아래에서 **Read all data using SQL (ReadData)**를 선택하고 다른 모든 상자의 선택을 해제합니다.

    ![Fabric에서 Warehouse 권한 부여 스크린샷](./Images/grant-warehouse-access.png)

9.  **Grant**를 선택합니다.

10. 두 번째 사용자로 로그인한 브라우저 창으로 돌아갑니다. 브라우저 보기를 새로고침합니다.

11. 두 번째 사용자는 더 이상 Workspace에 접근할 수 없으며, 대신 Warehouse에만 접근할 수 있습니다. 왼쪽 탐색 창에서 더 이상 Workspace를 탐색하여 Warehouse를 찾을 수 없습니다. 왼쪽 탐색 메뉴에서 **OneLake**를 선택하여 Warehouse를 찾습니다.

12. Warehouse를 선택합니다. 나타나는 화면의 상단 메뉴 바에서 **Open**을 선택합니다.

13. Warehouse 뷰가 나타나면 **Date** 테이블을 선택하여 테이블 데이터를 봅니다. Warehouse에 대한 Item 권한을 사용하여 ReadData 권한이 적용되었기 때문에 사용자는 여전히 Warehouse에 대한 읽기 접근 권한이 있어 행을 볼 수 있습니다.

## Lakehouse에서 OneLake 데이터 액세스 역할 적용

---
### **개념 설명: OneLake Data Access Roles (OneLake 데이터 접근 역할)**

OneLake 데이터 액세스 역할은 보안 제어의 가장 세분화된 계층입니다. 이 기능은 `Lakehouse` 내에서 사용자 정의 역할을 만들고, 해당 역할에 특정 **폴더나 테이블**에 대한 읽기 권한만을 부여할 수 있게 해줍니다.

이는 Item 권한보다 한 단계 더 나아간 것입니다.
*   **Item 권한**: Lakehouse라는 '방'에 들어갈 수 있는 키를 줍니다. 하지만 기본 'Read' 권한만으로는 방 안의 내용물(데이터)을 볼 수 없습니다.
*   **OneLake 데이터 액세스 역할**: 방 안에 있는 특정 '파일 캐비닛(폴더)'이나 '서류(테이블)'를 볼 수 있는 구체적인 권한을 부여합니다.

이 기능을 사용하면 동일한 Lakehouse 내에서도 부서나 역할에 따라 접근할 수 있는 데이터를 분리하는 등 매우 정교한 보안 정책(폴더 수준 보안)을 구현할 수 있습니다. 이 기능은 현재 **Preview** 상태입니다.
---

이 실습에서는 Item 권한을 할당하고 OneLake 데이터 액세스 역할을 생성하여 Lakehouse의 데이터에 대한 접근을 제한하는 방법을 실험합니다.

1.  두 번째 사용자로 로그인한 브라우저에 그대로 있습니다.
2.  왼쪽 탐색 바에서 **OneLake**를 선택합니다. 두 번째 사용자는 Lakehouse를 볼 수 없습니다.
3.  Workspace Admin으로 로그인한 브라우저로 돌아갑니다.
4.  왼쪽 메뉴에서 **Workspaces**를 선택하고 당신의 Workspace를 선택합니다. Lakehouse 이름 위로 마우스를 가져갑니다.
5.  줄임표(**...**) 오른쪽의 줄임표를 선택하고 **Manage permissions**를 선택합니다.

      ![Fabric의 Lakehouse에서 권한 설정 스크린샷](./Images/lakehouse-manage-permissions.png)

6.  나타나는 화면에서 **Add user**를 선택합니다.
7.  두 번째 사용자를 Lakehouse에 할당하고 **Grant People Access** 창의 체크박스가 모두 선택 해제되었는지 확인합니다.

      ![Fabric의 grant access lakehouse 창 스크린샷](./Images/grant-people-access-window.png)

8.  **Grant**를 선택합니다. 이제 두 번째 사용자는 Lakehouse에 대한 읽기 권한을 가집니다. 읽기 권한은 사용자가 Lakehouse의 메타데이터만 볼 수 있게 하지만 기본 데이터는 볼 수 없습니다. 다음으로 이를 확인하겠습니다.
9.  두 번째 사용자로 로그인한 브라우저로 돌아갑니다. 브라우저를 새로고침합니다.
10. 왼쪽 탐색 창에서 **OneLake**를 선택합니다.
11. Lakehouse를 선택하고 엽니다.
12. 상단 메뉴 바에서 **Open**을 선택합니다. 읽기 권한이 부여되었음에도 불구하고 테이블이나 파일을 확장할 수 없습니다. 다음으로, OneLake 데이터 액세스 권한을 사용하여 두 번째 사용자에게 특정 폴더에 대한 접근 권한을 부여합니다.
13. Workspace 관리자로 로그인한 브라우저로 돌아갑니다.
14. 왼쪽 탐색 바에서 **Workspaces**를 선택합니다.
15. 당신의 Workspace 이름을 선택합니다.
16. Lakehouse를 선택합니다.
1. Lakehouse가 열리면, 상단 메뉴 바에서 **Manage OneLake data access**를 선택하고 **Continue** 버튼을 클릭하여 기능을 활성화합니다.

      ![Fabric 메뉴 바의 Manage OneLake data access (preview) 기능 스크린샷](./Images/manage-onelake-roles.png)

14. 나타나는 **Manage OneLake data access (preview)** 화면에서 **new role**을 선택합니다.
  
      ![manage OneLake data access 기능의 새로운 역할 기능 스크린샷](./Images/create-onelake-role.png)

15. 아래 스크린샷과 같이 publicholidays 폴더에만 접근할 수 있는 **publicholidays**라는 새 역할을 생성합니다.

      ![manage OneLake data access 기능의 폴더 할당 스크린샷](./Images/new-data-access-role.png)

16. 역할 생성이 완료되면 **Assign role**을 선택하고 역할을 두 번째 사용자에게 할당한 다음, **Add**를 선택하고 **Save**를 선택합니다.
 
       ![manage OneLake data access 기능의 폴더 할당 스크린샷](./Images/assign-role.png)

17. 두 번째 사용자로 로그인한 브라우저로 돌아갑니다. Lakehouse가 열려 있는 페이지에 있는지 확인합니다. 브라우저를 새로고침합니다.
18. **publicholidays** 테이블을 선택하고 데이터가 로드될 때까지 기다립니다. 사용자는 사용자 정의 OneLake 데이터 액세스 역할에 할당되었기 때문에 publicholidays 테이블의 데이터에만 접근할 수 있습니다. 이 역할은 다른 테이블, 파일 또는 폴더의 데이터가 아닌 publicholidays 테이블의 데이터만 볼 수 있도록 허용합니다.

## 리소스 정리

이 실습에서는 Workspace 액세스 제어, Item 액세스 제어 및 OneLake 데이터 액세스 역할을 사용하여 데이터를 보호했습니다.

1.  왼쪽 탐색 바에서 Workspace 아이콘을 선택하여 포함된 모든 항목을 봅니다.
2.  상단 도구 모음의 메뉴에서 **Workspace settings**를 선택합니다.
3.  **General** 섹션에서 **Remove this workspace**를 선택합니다.
