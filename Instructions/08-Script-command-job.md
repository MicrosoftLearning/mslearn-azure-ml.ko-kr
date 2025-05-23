---
lab:
  title: Azure Machine Learning에서 학습 스크립트를 명령 작업으로 실행
---

# Azure Machine Learning에서 학습 스크립트를 명령 작업으로 실행

Notebook은 실험과 개발에 이상적입니다. 기계 학습 모델을 개발하고 프로덕션 준비가 완료되면 스크립트를 사용하여 학습시키고 싶을 것입니다. 스크립트를 명령 작업으로 실행할 수 있습니다.

이 연습에서는 스크립트를 테스트한 후 명령 작업으로 실행합니다.

## 시작하기 전에

관리 수준 액세스 권한이 있는 [Azure 구독](https://azure.microsoft.com/free?azure-portal=true)이 필요합니다.

## Azure Machine Learning 작업 영역 프로비저닝

Azure Machine Learning 작업 영역은 모델을 학습하고 관리하는 데 필요한 모든 리소스 및 자산을 관리하기 위한 중심지를 제공합니다.** 스튜디오, Python SDK, Azure CLI를 통해 Azure Machine Learning 작업 영역과 상호 작용할 수 있습니다.

Azure CLI를 사용하여 작업 영역과 필요한 컴퓨팅을 프로비전하고, Python SDK를 사용하여 명령 작업을 실행합니다.

### 작업 영역 및 컴퓨팅 리소스 만들기

Azure Machine Learning 작업 영역, 컴퓨팅 인스턴스 및 컴퓨팅 클러스터를 만들려면 Azure CLI를 사용합니다. 필요한 모든 명령은 실행할 수 있도록 셸 스크립트로 그룹화됩니다.

1. 브라우저에서 `https://portal.azure.com/`에서 Azure Portal을 열고 Microsoft 계정으로 로그인합니다.
1. 검색 상자 오른쪽 페이지 맨 위에 있는 \[>_](*Cloud Shell*) 단추를 선택합니다. 그러면 포털 아래쪽에 Cloud Shell 창이 열립니다.
1. 메시지가 표시되면 **Bash**를 선택합니다. Cloud Shell을 처음 열면 사용할 셸 유형(Bash 또는 PowerShell)을 선택하라는 메시지가 표시됩니다.****
1. 올바른 구독이 지정되어 있고 **필요한 스토리지 계정이 선택되어 있지 않은지** 확인합니다. **적용**을 선택합니다.
1. 터미널에서 다음 명령을 입력하여 이 리포지토리를 복제합니다.

    ```azurecli
    rm -r azure-ml-labs -f
    git clone https://github.com/MicrosoftLearning/mslearn-azure-ml.git azure-ml-labs
    ```

    > `SHIFT + INSERT`를 사용하여 복사한 코드를 Cloud Shell에 붙여넣습니다.

1. 리포지토리가 복제된 후에는 다음 명령을 입력하여 이 랩의 폴더로 변경하고 포함된 **setup.sh** 스크립트를 실행합니다.
    
    ```azurecli
    cd azure-ml-labs/Labs/08
    ./setup.sh
    ```

    > 확장 기능이 설치되지 않았다는 오류 메시지는 무시합니다.

1. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 5~10분이 걸립니다.

    <details>
    <summary><b>문제 해결 팁</b>: 작업 영역 만들기 오류</summary><br>
    <p>CLI를 통해 설치 스크립트를 실행할 때 오류가 발생하는 경우 리소스를 수동으로 프로비전해야 합니다.</p>
    <ol>
        <li>Azure Portal 홈페이지에서 <b>+리소스 만들기</b>를 선택합니다.</li>
        <li><i>기계 학습</i>을 검색한 다음 <b>Azure Machine Learning</b>을 선택합니다. <b>만들기</b>를 실행합니다.</li>
        <li>다음 설정을 사용하여 새 Azure Machine Learning 리소스를 만듭니다. <ul>
                <li><b>구독</b>: ‘Azure 구독’</li>
                <li><b>리소스 그룹</b>: rg-dp100-labs</li>
                <li><b>작업 영역 이름</b>: mlw-dp100-labs</li>
                <li><b>지역</b>: ‘지리적으로 가장 가까운 지역 선택’<i></i></li>
                <li><b>스토리지 계정</b>: <i>‘작업 영역에 대해 만들 새로운 기본 스토리지 계정’</i></li>
                <li><b>키 자격 증명 모음</b>: ‘작업 영역에 대해 만들 새로운 기본 키 자격 증명 모음’</li>
                <li><b>Application insights</b>: ‘작업 영역에 대해 만들 새로운 기본 Application Insights 리소스’</li>
                <li><b>컨테이너 레지스트리</b>: 없음(‘처음으로 컨테이너에 모델을 배포할 때 자동으로 만들어짐’)</li>
            </ul>
        <li><b>검토 + 만들기</b>를 선택하고 작업 영역과 관련 리소스가 생성될 때까지 기다립니다(일반적으로 5분 정도 소요됨).</li>
        <li><b>리소스로 이동</b>을 선택하고 <b>개요</b> 페이지에서 <b>스튜디오 시작</b>을 선택합니다. Azure Machine Learning 스튜디오를 열 수 있는 또 다른 탭이 브라우저에 열립니다.</li>
        <li>스튜디오에 나타나는 팝업을 모두 닫으세요.</li>
        <li>Azure Machine Learning 스튜디오에서 <b>컴퓨팅</b> 페이지로 이동하고 <b>컴퓨팅 인스턴스</b> 탭에서<b>+ 새로 만들기</b>를 선택합니다.</li>
        <li>컴퓨팅 인스턴스에 고유한 이름을 지정한 다음 가상 머신 크기로 <b>Standard_DS11_v2</b>를 선택합니다.</li>
        <li><b>검토 + 만들기</b>를 선택한 다음, <b>만들기</b>를 선택합니다.</li>
        <li>다음으로 <b>컴퓨팅 클러스터</b> 탭을 선택하고<b>+ 새로 만들기</b>를 선택합니다.</li>
        <li>작업 영역을 생성한 지역과 동일한 지역을 선택한 다음 가상 머신 크기로 <b>Standard_DS11_v2</b>를 선택합니다. <b>다음</b>을 선택합니다.</li>
        <li>클러스터에 고유한 이름을 지정한 다음 <b>만들기</b>를 선택합니다.</li>
    </ol>
    </details>

## 랩 자료 복제

작업 영역과 필요한 컴퓨팅 리소스를 만들었으면 Azure Machine Learning 스튜디오를 열고 랩 자료를 작업 영역에 복제할 수 있습니다.

1. Azure Portal에서 **mlw-dp100-...** 이라는 Azure Machine Learning 작업 영역으로 이동합니다.
1. Azure Machine Learning 작업 영역을 선택하고 **개요** 페이지에서 **스튜디오 시작**을 선택합니다. Azure Machine Learning 스튜디오를 열 수 있는 또 다른 탭이 브라우저에 열립니다.
1. 스튜디오에 나타나는 팝업을 모두 닫으세요.
1. Azure Machine Learning 스튜디오 내에서 **컴퓨팅** 페이지로 이동하여 이전 섹션에서 만든 컴퓨팅 인스턴스와 클러스터가 있는지 확인합니다. 컴퓨팅 인스턴스가 실행 중이어야 하고, 클러스터는 유휴 상태여야 하며 실행 중인 노드가 0개여야 합니다.
1. **컴퓨팅 인스턴스** 탭에서 컴퓨팅 인스턴스를 찾고 **터미널** 애플리케이션을 선택합니다.
1. 터미널에서 다음 명령을 실행하여 컴퓨팅 인스턴스에 Python SDK를 설치합니다.

    ```
    pip uninstall azure-ai-ml
    pip install azure-ai-ml
    ```

    > 패키지를 찾아서 제거할 수 없다는 오류 메시지는 무시합니다.

1. 다음 명령을 실행하여 Notebooks, 데이터 및 기타 파일이 포함된 Git 리포지토리를 작업 영역에 복제합니다.

    ```
    git clone https://github.com/MicrosoftLearning/mslearn-azure-ml.git azure-ml-labs
    ```

1. 명령이 완료되면 **파일** 창에서 **&#8635;** 를 클릭하여 보기를 새로 고치고 **Users/*your-user-name */azure-ml-labs** 폴더가 새로 만들어졌습니다.

## Notebook을 스크립트로 변환

컴퓨팅 인스턴스에 연결된 Notebook을 사용하면 작성한 코드를 즉시 실행하고 해당 출력을 검토할 수 있으므로 실험 및 개발에 이상적입니다. 개발에서 프로덕션으로 이동하려면 스크립트를 사용하는 것이 좋습니다. 첫 번째 단계로 Azure Machine Learning 스튜디오를 사용하여 Notebook을 스크립트로 변환할 수 있습니다.

1. **Labs/08/src/Train classification model.ipynb** Notebook을 엽니다.

    > **인증**을 선택하고 인증을 요청하는 알림이 표시되면 필요한 단계를 따릅니다.

1. Notebook이 **Python 3.10 - AzureML** 커널을 사용하는지 확인합니다.
1. 모든 셀을 실행하여 코드를 탐색하고 모델을 학습합니다.
1. &#9776; **Notebook 메뉴**를 보려면 Notebook 상단에 있는 아이콘을 클릭합니다.
1. **다른 이름으로 내보내기**를 확장하고 **Python(.py)** 을 선택하여 Notebook을 Python 스크립트로 변환합니다.
1. 새 파일의 이름을 `train-classification-model`로 지정합니다.
1. 새 파일이 만들어지면 스크립트가 자동으로 열립니다. 파일을 탐색하여 Notebook과 동일한 코드가 포함되어 있는지 확인합니다.
1. **터미널에 스크립트를 저장하고 실행**하려면 Notebook 상단에 있는 &#9655;&#9655; 아이콘을 선택합니다.
1. 스크립트는 **python train-classification-model.py** 명령으로 시작되며 출력은 명령 아래에 표시되어야 합니다.

   > **참고:** 스크립트가 libstdc++6에 대한 ImportError를 반환하는 경우 스크립트를 다시 실행하기 전에 터미널에서 다음 명령을 실행합니다.
   > ```bash
   > sudo add-apt-repository ppa:ubuntu-toolchain-r/test
   > sudo apt-get update
   > sudo apt-get upgrade libstdc++6
   > ```

## 터미널로 스크립트 테스트

Notebook을 스크립트로 변환한 후 이를 더욱 구체화할 수 있습니다. 스크립트 작업 시 가장 좋은 방법 중 하나는 함수를 사용하는 것입니다. 스크립트가 함수로 구성되면 코드를 단위 테스트하기가 더 쉽습니다. 함수를 사용하면 스크립트는 코드 블록으로 구성되며, 각 블록은 특정 작업을 수행합니다.

1. **Labs/08/src/train-model-parameters.py** 스크립트를 열고 콘텐츠를 살펴봅니다.
    네 가지 다른 함수를 포함하는 주요 함수가 있습니다.

    - 데이터 읽기
    - 데이터 분할
    - 모델 학습
    - 모델 평가

    주요 함수 다음에 각 함수가 정의됩니다. 각 함수가 예상 입력과 출력을 어떻게 정의하는지 확인합니다.

1. **터미널에 스크립트를 저장하고 실행**하려면 Notebook 상단에 있는 &#9655;&#9655; 아이콘을 선택합니다. **데이터를 읽는 중...** 후에 파일 경로가 잘못되어 데이터를 가져올 수 없다는 오류가 발생합니다.

    스크립트를 사용하면 코드를 매개 변수화하여 입력 데이터나 매개 변수를 쉽게 변경할 수 있습니다. 이 경우 스크립트는 제공하지 않은 데이터 경로에 대한 입력 매개 변수를 예상합니다. **parse_args()** 함수의 스크립트 끝에서 정의된 매개 변수와 예상 매개 변수를 찾을 수 있습니다.

    두 가지 입력 매개 변수가 정의되어 있습니다.
    - **--training_data**는 문자열이 필요합니다.
    - **--reg_rate**는 숫자를 예상하지만 기본값은 0.01입니다.

    스크립트를 성공적으로 실행하려면 학습 데이터 매개 변수의 값을 지정해야 합니다. 학습 스크립트와 동일한 폴더에 저장된 **diabetes.csv** 파일을 참조하여 이를 수행해 보겠습니다.

1. 터미널에서 다음 명령을 실행합니다.

    ```
    cd azure-ml-labs/Labs/08/src/
    python train-model-parameters.py --training_data diabetes.csv
    ```

스크립트가 성공적으로 실행되어야 하며 결과적으로 출력에는 학습된 모델의 정확도와 AUC가 표시되어야 합니다.

터미널에서 스크립트를 테스트하는 것은 스크립트가 예상대로 작동하는지 확인하는 데 이상적입니다. 코드에 문제가 있으면 터미널에 오류가 표시됩니다.

**선택적으로** 코드를 편집하여 오류를 강제로 발생시키고 터미널에서 명령을 다시 실행하여 스크립트를 실행합니다. 예를 들어, **import pandas as pd** 행을 제거하고 입력 매개 변수가 포함된 스크립트를 저장하고 실행하여 오류 메시지를 검토합니다.

## 명령 작업으로 스크립트 실행

스크립트가 작동한다는 것을 알고 있으면 명령 작업으로 실행할 수 있습니다. 스크립트를 명령 작업으로 실행하면 스크립트의 모든 입력과 출력을 추적할 수 있습니다.

1. **Labs/08/Run script as command job.ipynb** Notebook을 엽니다.
1. Notebook의 모든 셀을 실행합니다.
1. Azure Machine Learning 스튜디오에서 **작업** 페이지로 이동합니다.
1. 실행한 명령 작업의 개요를 탐색하려면 **diabetes-train-script** 작업으로 이동합니다.
1. **코드** 탭으로 이동합니다. 명령 작업의 **code** 매개 변수 값인 **src** 폴더의 모든 콘텐츠가 여기에 복사됩니다. 명령 작업에 의해 실행된 학습 스크립트를 검토할 수 있습니다.
1. **출력 + 로그** 탭으로 이동합니다.
1. **std_log.txt** 파일을 열고 콘텐츠를 살펴봅니다. 이 파일의 콘텐츠는 명령의 출력입니다. 스크립트를 테스트할 때 터미널에 동일한 출력이 표시되었음을 기억합니다. 스크립트 문제로 인해 작업이 실패한 경우 여기에 오류 메시지가 표시됩니다.

**필요에 따라** 코드를 편집하여 오류를 강제로 발생시키고 Notebook을 사용하여 명령 작업을 다시 시작합니다. 예를 들어, 스크립트에서 **import pandas as pd** 줄을 제거하고 스크립트를 저장합니다. 또는 스크립트 대신 작업 구성 자체에 문제가 있는 경우 명령 작업 구성을 편집하여 오류 메시지를 탐색합니다.

## Azure 리소스 삭제

Azure Machine Learning 탐색을 마치면 지금까지 만든 리소스를 삭제하여 불필요한 Azure 비용을 방지해야 합니다.

1. Azure Machine Learning 스튜디오 탭을 닫고 Azure Portal로 돌아갑니다.
1. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
1. **rg-dp100-...** 리소스 그룹을 선택합니다.
1. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음 **삭제**를 선택합니다.
