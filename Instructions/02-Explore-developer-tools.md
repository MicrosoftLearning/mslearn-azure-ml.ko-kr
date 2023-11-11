---
lab:
  title: 작업 영역 상호 작용을 위한 개발자 도구 살펴보기
---

# 작업 영역 상호 작용을 위한 개발자 도구 살펴보기

다양한 도구를 사용하여 Azure Machine Learning 작업 영역과 상호 작용할 수 있습니다. 수행해야 하는 작업과 개발자 도구에 대한 기본 설정에 따라 언제 사용할 도구를 선택할 수 있습니다. 이 랩은 작업 영역 상호 작용에 일반적으로 사용되는 개발자 도구를 소개하기 위해 설계되었습니다. 특정 도구를 더 자세히 사용하는 방법을 알아보려면 탐색할 다른 랩이 있습니다.

## 시작하기 전에

관리 수준 액세스 권한이 있는 [Azure 구독](https://azure.microsoft.com/free?azure-portal=true)이 필요합니다.

Azure Machine Learning 작업 영역과 상호 작용하기 위해 일반적으로 사용되는 개발자 도구는 다음과 같습니다.

- **Azure Machine Learning 확장을 사용하는 Azure CLI** : 이 명령줄 접근 방식은 인프라 자동화에 이상적입니다.
- **** Azure Machine Learning 스튜디오: 사용자에게 친숙한 UI를 사용하여 작업 영역 및 모든 기능을 탐색합니다.
- **Azure Machine Learning용 Python SDK** : 데이터 과학자에게 이상적인 Jupyter Notebook에서 작업을 제출하고 모델을 관리하는 데 사용합니다.

해당 도구로 일반적으로 수행되는 작업에 대해 이러한 각 도구를 탐색합니다.

## Azure CLI를 사용하여 인프라 프로비전

데이터 과학자가 Azure Machine Learning을 사용하여 기계 학습 모델을 학습하려면 필요한 인프라를 설정해야 합니다. Azure Machine Learning 확장과 함께 Azure CLI를 사용하여 컴퓨팅 인스턴스와 같은 Azure Machine Learning 작업 영역 및 리소스를 만들 수 있습니다.

시작하려면 Azure Cloud Shell을 열고 Azure Machine Learning 확장을 설치하고 Git 리포지토리를 복제합니다.

1. 브라우저에서 Azure Portal을 `https://portal.azure.com/`열고 Microsoft 계정으로 로그인합니다.
1. 검색 상자 오른쪽 페이지 맨 위에 있는 \[>_](*Cloud Shell*) 단추를 선택합니다. 그러면 포털 아래쪽에 Cloud Shell 창이 열립니다.
1. 메시지가 표시되면 Bash**를 선택합니다**. 클라우드 셸을 처음 열면 사용하려는 셸 유형(*Bash* 또는 *PowerShell*)을 선택하라는 메시지가 표시됩니다.
1. 올바른 구독이 지정되었는지 확인하고 클라우드 셸에 대한 스토리지를 만들라는 메시지가 표시되면 스토리지** 만들기를 선택합니다**. 스토리지가 생성될 때까지 기다립니다.
1. 다음 명령을 사용하여 이전 버전과 충돌하지 않도록 ML CLI 확장(버전 1 및 2 모두)을 제거합니다.
    
    ```azurecli
    az extension remove -n azure-cli-ml
    az extension remove -n ml
    ```

    > 복사한 코드를 Cloud Shell에 붙여넣는 데 사용합니다 `SHIFT + INSERT` .

    > 확장이 설치되지 않았다는 (오류) 메시지를 무시합니다.

1. 다음 명령을 사용하여 Azure Machine Learning(v2) 확장을 설치합니다.
    
    ```azurecli
    az extension add -n ml -y
    ```

1. 리소스 그룹을 만듭니다. 가까운 위치를 선택합니다.
    
    ```azurecli
    az group create --name "rg-dp100-labs" --location "eastus"
    ```

1. 작업 영역을 만듭니다.
    
    ```azurecli
    az ml workspace create --name "mlw-dp100-labs" -g "rg-dp100-labs"
    ```

1. 작업 영역 및 관련 리소스가 생성될 때까지 기다립니다. 일반적으로 약 5분이 걸립니다.

## Azure CLI를 사용하여 컴퓨팅 인스턴스 만들기

기계 학습 모델을 학습하는 데 필요한 인프라의 또 다른 중요한 부분은 컴퓨팅**입니다**. 모델을 로컬로 학습시킬 수 있지만 클라우드 컴퓨팅을 사용하는 것이 더 확장 가능하고 비용 효율적입니다.

데이터 과학자가 Azure Machine Learning 작업 영역에서 기계 학습 모델을 개발하는 경우 Jupyter Notebook을 실행할 수 있는 가상 머신을 사용하려고 합니다. 개발의 **경우 컴퓨팅 인스턴스** 가 적합합니다.

Azure Machine Learning 작업 영역을 만든 후 Azure CLI를 사용하여 컴퓨팅 인스턴스를 만들 수도 있습니다.

이 연습에서는 다음 설정을 사용하여 컴퓨팅 인스턴스를 만듭니다.

- **컴퓨팅 이름**: *컴퓨팅 인스턴스의 이름입니다. 고유하고 24자 미만이어야 합니다.*
- **가상 머신 크기**: STANDARD_DS11_V2
- **컴퓨팅 유형** (인스턴스 또는 클러스터): ComputeInstance
- **Azure Machine Learning 작업 영역 이름**: mlw-dp100-labs
- **리소스 그룹**: rg-dp100-labs

1. 다음 명령을 사용하여 작업 영역에서 컴퓨팅 인스턴스를 만듭니다. 컴퓨팅 인스턴스 이름에 "XXXX"가 포함된 경우 난수로 바꿔 고유한 이름을 만듭니다.

    ```azurecli
    az ml compute create --name "ciXXXX" --size STANDARD_DS11_V2 --type ComputeInstance -w mlw-dp100-labs -g rg-dp100-labs
    ```

    이름이 있는 컴퓨팅 인스턴스가 이미 있다는 오류 메시지가 표시되면 이름을 변경하고 명령을 다시 시도합니다.

## Azure CLI를 사용하여 컴퓨팅 클러스터 만들기

컴퓨팅 인스턴스는 개발에 이상적이지만 기계 학습 모델을 학습하려는 경우 컴퓨팅 클러스터가 더 적합합니다. 컴퓨팅 클러스터를 사용하기 위해 작업이 제출된 경우에만 0개 이상의 노드로 크기를 조정하고 작업을 실행합니다. 컴퓨팅 클러스터가 더 이상 필요하지 않으면 비용을 최소화하기 위해 자동으로 0개 노드로 크기를 조정합니다. 

컴퓨팅 클러스터를 만들려면 컴퓨팅 인스턴스를 만드는 것과 유사하게 Azure CLI를 사용할 수 있습니다.

다음 설정을 사용하여 컴퓨팅 클러스터를 만듭니다.

- **컴퓨팅 이름**: aml-cluster
- **가상 머신 크기**: STANDARD_DS11_V2
- **컴퓨팅 유형**: AmlCompute *(컴퓨팅 클러스터 만들기)*
- **최대 인스턴스**: *최대 노드 수*
- **Azure Machine Learning 작업 영역 이름**: mlw-dp100-labs
- **리소스 그룹**: rg-dp100-labs

1. 다음 명령을 사용하여 작업 영역에서 컴퓨팅 클러스터를 만듭니다.
    
    ```azurecli
    az ml compute create --name "aml-cluster" --size STANDARD_DS11_V2 --max-instances 2 --type AmlCompute -w mlw-dp100-labs -g rg-dp100-labs
    ```

## Azure Machine Learning 스튜디오 사용하여 워크스테이션 구성

Azure CLI는 자동화에 적합하지만 실행한 명령의 출력을 검토할 수 있습니다. Azure Machine Learning 스튜디오 사용하여 리소스 및 자산이 만들어졌는지 여부를 검사 작업이 성공적으로 실행되었는지 여부를 검사 작업이 실패한 이유를 검토할 수 있습니다. 

1. Azure Portal에서 mlw-dp100-labs**라는 **Azure Machine Learning 작업 영역으로 이동합니다.
1. Azure Machine Learning 작업 영역을 선택하고 개요 **** 페이지에서 시작 스튜디오**를 선택합니다**. 브라우저에서 다른 탭이 열리고 Azure Machine Learning 스튜디오 열립니다.
1. 스튜디오에 표시되는 팝업을 닫습니다.
1. Azure Machine Learning 스튜디오 내에서 컴퓨팅** 페이지로 이동하여 **이전 섹션에서 만든 컴퓨팅 인스턴스 및 클러스터가 있는지 확인합니다. 컴퓨팅 인스턴스가 실행되고 클러스터가 유휴 상태이고 0개의 노드가 실행되고 있어야 합니다.

## Python SDK를 사용하여 모델 학습

이제 필요한 컴퓨팅이 만들어졌는지 확인했으므로 Python SDK를 사용하여 학습 스크립트를 실행할 수 있습니다. 컴퓨팅 인스턴스에 Python SDK를 설치 및 사용하고 컴퓨팅 클러스터에서 기계 학습 모델을 학습시킵니다.

1. 터미널을 **시작할 컴퓨팅 인스턴스**에 대한 **터미널** 애플리케이션을 선택합니다.
1. 터미널에서 터미널에서 다음 명령을 실행하여 컴퓨팅 인스턴스에 Python SDK를 설치합니다.

    ```
    pip uninstall azure-ai-ml
    pip install azure-ai-ml
    ```

    > 패키지가 설치되지 않았다는 (오류) 메시지를 무시합니다.

1. 다음 명령을 실행하여 Notebook, 데이터 및 기타 파일이 포함된 Git 리포지토리를 작업 영역에 복제합니다.

    ```
    git clone https://github.com/MicrosoftLearning/mslearn-azure-ml.git azure-ml-labs
    ```

1. 명령이 완료되면 파일 창에서 **&#8635**를 선택하여 **보기를 새로 고치고 새 **Users/your-user-name */* azure-ml-labs 폴더가 만들어졌는지 확인합니다**.**
1. **Labs/02/Run training script.ipynb** Notebook을 엽니다.

    > 인증**을 선택하고 **인증을 요청하는 알림이 표시되면 필요한 단계를 따릅니다.

1. Notebook에서 Python 3.8 - AzureML** 커널을 사용하는**지 확인합니다. 각 커널에는 자체 패키지 집합이 미리 설치된 고유한 이미지가 있습니다.
1. Notebook의 모든 셀을 실행합니다.

Azure Machine Learning 작업 영역에 새 작업이 만들어집니다. 작업은 작업 구성에 정의된 입력, 사용된 코드 및 모델을 평가하는 메트릭과 같은 출력을 추적합니다.

## Azure Machine Learning 스튜디오 작업 기록 검토

Azure Machine Learning 작업 영역에 작업을 제출할 때 Azure Machine Learning 스튜디오 해당 상태 검토할 수 있습니다.

1. Notebook에서 출력으로 제공된 작업 URL을 선택하거나 Azure Machine Learning 스튜디오 작업** 페이지로 **이동합니다.
1. 새로운 실험은 당뇨병 훈련**이라는 이름으로 **나열됩니다. 최신 작업 **당뇨병-pythonv2-train**을 선택합니다.
1. 작업의 **속성을 검토합니다**. 작업 **상태를** 확인합니다.
    - **대기 중**: 작업이 컴퓨팅을 사용할 수 있길 기다리고 있습니다.
    - **** 준비 중: 컴퓨팅 클러스터의 크기가 조정되거나 컴퓨팅 대상에 환경이 설치되고 있습니다.
    - **실행** 중: 학습 스크립트가 실행 중입니다.
    - **완료**: 학습 스크립트가 실행되고 작업이 모든 최종 정보로 업데이트되고 있습니다.
    - **완료됨**: 작업이 성공적으로 완료되고 종료됩니다.
    - **** 실패: 작업이 실패하고 종료되었습니다.
1. 출력 + 로그** 아래에서 **스크립트의 출력은 user_logs/std_log.txt**에서 **찾을 수 있습니다. 스크립트의 **인쇄** 문 출력이 여기에 표시됩니다. 스크립트에 문제가 있어 오류가 발생하면 여기에서도 오류 메시지를 찾을 수 있습니다.
1. 코드** 아래에서 **작업 구성에서 지정한 폴더를 찾을 수 있습니다. 이 폴더에는 학습 스크립트 및 데이터 세트가 포함됩니다.

## Azure 리소스 삭제

Azure Machine Learning 탐색을 마치면 지금까지 만든 리소스를 삭제하여 불필요한 Azure 비용을 방지해야 합니다.

1. Azure Machine Learning 스튜디오 탭을 닫고 Azure Portal로 돌아갑니다.
1. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
1. **rg-dp100-labs** 리소스 그룹을 선택합니다.
1. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다. 
1. 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음 **삭제**를 선택합니다.
