---
lab:
  title: MLflow를 사용하여 Notebooks에서 모델 학습 추적
---

# MLflow를 사용하여 Notebooks에서 모델 학습 추적

종종 여러 모델을 실험하고 학습하여 새로운 데이터 과학 프로젝트를 시작하게 됩니다. 작업을 추적하고 학습한 모델과 모델의 성능에 대한 개요를 유지하기 위해 MLflow 추적을 사용할 수 있습니다.

이 연습에서는 컴퓨팅 인스턴스에서 실행되는 Notebook 내에서 MLflow를 사용하여 모델 학습을 기록합니다.

## 시작하기 전에

관리 수준 액세스 권한이 있는 [Azure 구독](https://azure.microsoft.com/free)이 필요합니다.

## Azure Machine Learning 작업 영역 프로비저닝

Azure Machine Learning 작업 영역은 모델을 학습하고 관리하는 데 필요한 모든 리소스 및 자산을 관리하기 위한 중심지를 제공합니다.** 스튜디오, Python SDK, Azure CLI를 통해 Azure Machine Learning 작업 영역과 상호 작용할 수 있습니다.

Azure CLI를 사용하여 작업 영역과 필요한 컴퓨팅을 프로비전하고 Python SDK를 사용하여 자동화된 Machine Learning으로 분류 모델을 학습합니다.

### 작업 영역 및 컴퓨팅 리소스 만들기

Azure Machine Learning 작업 영역 및 컴퓨팅 인스턴스를 만들려면 Azure CLI를 사용합니다. 필요한 모든 명령은 실행할 수 있도록 셸 스크립트로 그룹화됩니다.
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
    cd azure-ml-labs/Labs/07
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
1. Azure Machine Learning 스튜디오 내에서 **컴퓨팅** 페이지로 이동하여 이전 섹션에서 만든 컴퓨팅 인스턴스가 있는지 확인합니다. 컴퓨팅 인스턴스가 실행 중이어야 합니다.
1. **컴퓨팅 인스턴스** 탭에서 컴퓨팅 인스턴스를 찾고 **터미널** 애플리케이션을 선택합니다.
1. 터미널에서 다음 명령을 실행하여 컴퓨팅 인스턴스에 Python SDK 및 MLflow 라이브러리를 설치합니다.

    ```
    pip uninstall azure-ai-ml
    pip install azure-ai-ml
    pip install mlflow
    ```

    > 패키지를 찾아서 제거할 수 없다는 오류 메시지는 무시합니다.

1. 다음 명령을 실행하여 Notebook, 데이터 및 기타 파일이 포함된 Git 리포지토리를 작업 영역에 복제합니다.

    ```
    git clone https://github.com/MicrosoftLearning/mslearn-azure-ml.git azure-ml-labs
    ```

1. 명령이 완료되면 **파일** 창에서 **&#8635;** 를 클릭하여 보기를 새로 고치고 **Users/*your-user-name */azure-ml-labs** 폴더가 새로 만들어졌습니다.

## MLflow를 사용하여 모델 학습 추적

이제 필요한 모든 리소스가 준비되었으므로 Notebook에서 모델을 학습할 때 Notebook을 실행하여 MLflow를 구성하고 사용할 수 있습니다.

1. **Labs/07/Track model training with MLflow.ipynb** Notebook을 엽니다.

    > **인증**을 선택하고 인증을 요청하는 알림이 표시되면 필요한 단계를 따릅니다.

1. Notebook이 **Python 3.10 - AzureML** 커널을 사용하는지 확인합니다.
1. Notebook의 모든 셀을 실행합니다.
1. 모델을 학습할 때마다 만들어지는 새 작업을 검토합니다.

    > **참고:** 모델을 학습시키면 셀의 출력에 작업 실행으로 가는 링크가 표시됩니다. 링크에서 오류를 반환하는 경우에도 왼쪽 패널에서 **작업**을 선택하여 작업 실행을 검토할 수 있습니다.
    
## Azure 리소스 삭제

Azure Machine Learning 탐색을 마치면 지금까지 만든 리소스를 삭제하여 불필요한 Azure 비용을 방지해야 합니다.

1. Azure Machine Learning 스튜디오 탭을 닫고 Azure Portal로 돌아갑니다.
1. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
1. **rg-dp100-...** 리소스 그룹을 선택합니다.
1. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음 **삭제**를 선택합니다.
