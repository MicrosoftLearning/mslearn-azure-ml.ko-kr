---
lab:
  title: 책임 있는 AI dashboard 만들기 및 탐색
---

# 책임 있는 AI dashboard 만들기 및 탐색

모델을 학습한 후에는 모델을 평가하여 모델이 예상대로 작동하는지 여부를 탐색할 수 있습니다. 성능 메트릭 옆에는 고려할 수 있는 다른 요인이 있습니다. Azure Machine Learning의 책임 있는 AI dashboard 통해 데이터와 모델의 예측을 분석하여 편견이나 불공정성을 식별할 수 있습니다.

이 연습에서는 데이터를 준비하고 Azure Machine Learning에서 책임 있는 AI dashboard 만듭니다.

## 시작하기 전에

관리 수준 액세스 권한이 있는 [Azure 구독](https://azure.microsoft.com/free?azure-portal=true)이 필요합니다.

## Azure Machine Learning 작업 영역 프로비저닝

Azure Machine Learning 작업 영역은 모델을 학습하고 관리하는 데 필요한 모든 리소스 및 자산을 관리하기 위한 중심지를 제공합니다. 스튜디오, Python SDK 및 Azure CLI를 통해 Azure Machine Learning 작업 영역과 상호 작용할 수 있습니다.

Azure CLI를 사용하여 작업 영역 및 필요한 컴퓨팅을 프로비전하고 Python SDK를 사용하여 명령 작업을 실행합니다.

### 작업 영역 및 컴퓨팅 리소스 만들기

Azure Machine Learning 작업 영역, 컴퓨팅 instance 및 컴퓨팅 클러스터를 만들려면 Azure CLI를 사용합니다. 필요한 모든 명령은 실행할 수 있도록 셸 스크립트로 그룹화됩니다.

1. 브라우저에서 에서 `https://portal.azure.com/`Azure Portal 열고 Microsoft 계정으로 로그인합니다.
1. 검색 상자 오른쪽 페이지 맨 위에 있는 \[>_](*Cloud Shell*) 단추를 선택합니다. 그러면 포털 아래쪽에 Cloud Shell 창이 열립니다.
1. 메시지가 표시되면 **Bash** 를 선택합니다. Cloud Shell을 처음 열면 사용할 셸 유형(Bash 또는 PowerShell)을 선택하라는 메시지가 표시됩니다. 
1. 올바른 구독이 지정되었는지 확인하고 클라우드 셸에 대한 **스토리지** 를 만들라는 메시지가 표시되면 스토리지 만들기를 선택합니다. 스토리지가 생성될 때까지 기다립니다.
1. 터미널에서 다음 명령을 입력하여 이 리포지토리를 복제합니다.

    ```azurecli
    rm -r azure-ml-labs -f
    git clone https://github.com/MicrosoftLearning/mslearn-azure-ml.git azure-ml-labs
    ```

    > 을 사용하여 `SHIFT + INSERT` 복사한 코드를 Cloud Shell 붙여넣습니다.

1. 리포지토리가 복제된 후 다음 명령을 입력하여 이 랩의 폴더로 변경하고 포함된 **setup.sh** 스크립트를 실행합니다.

    ```azurecli
    cd azure-ml-labs/Labs/10
    ./setup.sh
    ```

    > 확장이 설치되지 않았다는 (오류) 메시지를 무시합니다.

1. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 5~10분이 걸립니다.

## 랩 자료 복제

작업 영역 및 필요한 컴퓨팅 리소스를 만든 경우 Azure Machine Learning 스튜디오 열고 랩 자료를 작업 영역에 복제할 수 있습니다.

1. Azure Portal **mlw-dp100-...** 라는 Azure Machine Learning 작업 영역으로 이동합니다.
1. Azure Machine Learning 작업 영역을 선택하고 **개요** 페이지에서 **스튜디오 시작을** 선택합니다. 브라우저에서 다른 탭이 열리고 Azure Machine Learning 스튜디오 열립니다.
1. 스튜디오에 표시되는 팝업을 닫습니다.
1. Azure Machine Learning 스튜디오 내에서 **컴퓨팅** 페이지로 이동하여 이전 섹션에서 만든 컴퓨팅 instance 및 클러스터가 있는지 확인합니다. 컴퓨팅 instance 실행되고 클러스터가 유휴 상태이고 0개의 노드가 실행되고 있어야 합니다.
1. **컴퓨팅 인스턴스** 탭에서 컴퓨팅 instance 찾고 **터미널** 애플리케이션을 선택합니다.
1. 터미널에서 터미널에서 다음 명령을 실행하여 컴퓨팅 instance Python SDK를 설치합니다.

    ```
    pip uninstall azure-ai-ml
    pip install azure-ai-ml
    ```

    > 패키지를 찾아 제거할 수 없다는 (오류) 메시지를 무시합니다.

1. 다음 명령을 실행하여 Notebook, 데이터 및 기타 파일이 포함된 Git 리포지토리를 작업 영역에 복제합니다.

    ```
    git clone https://github.com/MicrosoftLearning/mslearn-azure-ml.git azure-ml-labs
    ```

1. 명령이 완료되면 **파일** 창에서 **&#8635;** 클릭하여 보기를 새로 고치고 새 **Users/*your-user-name*/azure-ml-labs** 폴더가 만들어졌는지 확인합니다.

## 모델을 평가하고 Notebook에서 제출하는 파이프라인 만들기

이제 필요한 모든 리소스가 있으므로 Notebook을 실행하여 기본 제공 책임 있는 AI 구성 요소를 검색하고, 파이프라인을 만들고, 파이프라인을 제출하여 책임 있는 AI dashboard 생성할 수 있습니다.

1. **Labs/10/Create Responsible AI dashboard.ipynb** Notebook을 엽니다.

    > **인증을** 선택하고 인증을 요청하는 알림이 표시되면 필요한 단계를 따릅니다.

1. Notebook에서 **Python 3.8 - AzureML** 커널을 사용하는지 확인합니다.
1. Notebook의 모든 셀을 실행합니다.

## Azure 리소스 삭제

Azure Machine Learning 탐색을 마치면 지금까지 만든 리소스를 삭제하여 불필요한 Azure 비용을 방지해야 합니다.

1. Azure Machine Learning 스튜디오 탭을 닫고 Azure Portal 돌아갑니다.
1. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
1. **rg-dp100-...** 리소스 그룹을 선택합니다.
1. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음 **삭제**를 선택합니다.
