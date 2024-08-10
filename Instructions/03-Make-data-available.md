---
lab:
  title: Azure Machine Learning에서 데이터를 사용할 수 있도록 만들기
---

# Azure Machine Learning에서 데이터를 사용할 수 있도록 만들기

일반적으로는 로컬 파일 시스템의 데이터를 사용하는 경우가 많습니다. 하지만 엔터프라이즈 환경에서는 여러 데이터 과학자 및 기계 학습 엔지니어가 액세스할 수 있는 중앙 위치에 데이터를 저장하는 방식이 더 효과적일 수 있습니다.

이 연습에서는 Azure Machine Learning에서 데이터 액세스를 추상화하는 데 사용되는 기본 개체인 *데이터 저장소*와 *데이터 자산*을 살펴보겠습니다.

## 시작하기 전에

관리 수준 액세스 권한이 있는 [Azure 구독](https://azure.microsoft.com/free?azure-portal=true)이 필요합니다.

## Azure Machine Learning 작업 영역 프로비저닝

Azure Machine Learning 작업 영역은 모델을 학습하고 관리하는 데 필요한 모든 리소스 및 자산을 관리하기 위한 중심지를 제공합니다.** 스튜디오, Python SDK, Azure CLI를 통해 Azure Machine Learning 작업 영역과 상호 작용할 수 있습니다.

Azure CLI를 사용하여 작업 영역과 필요한 리소스를 프로비전하는 셸 스크립트를 사용합니다. 다음으로 Azure Machine Learning 스튜디오의 디자이너를 사용하여 모델을 학습하고 비교합니다.

### 작업 영역 및 컴퓨팅 리소스 만들기

Azure Machine Learning 작업 영역과 컴퓨팅 리소스를 만들려면 Azure CLI를 사용합니다. 필요한 모든 명령은 실행할 수 있도록 셸 스크립트로 그룹화됩니다.

1. 브라우저에서 `https://portal.azure.com/`에서 Azure Portal을 열고 Microsoft 계정으로 로그인합니다.
1. 검색 상자 오른쪽 페이지 맨 위에 있는 \[>_](*Cloud Shell*) 단추를 선택합니다. 그러면 포털 아래쪽에 Cloud Shell 창이 열립니다.
1. 메시지가 표시되면 **Bash**를 선택합니다. Cloud Shell을 처음 열면 사용할 셸 유형(Bash 또는 PowerShell)을 선택하라는 메시지가 표시됩니다.****
1. 올바른 구독이 지정되어 있고 **필요한 스토리지 계정이 선택되어 있지 않은지** 확인합니다. **적용**을 선택합니다.
1. 이 리포지토리를 복제하려면 터미널에 다음 명령을 입력합니다.

    ```azurecli
    rm -r azure-ml-labs -f
    git clone https://github.com/MicrosoftLearning/mslearn-azure-ml.git azure-ml-labs
    ```

    > `SHIFT + INSERT`를 사용하여 복사한 코드를 Cloud Shell에 붙여넣습니다.

1. 리포지토리가 복제된 후 다음 명령을 입력하여 이 랩용 폴더로 변경하고 포함된 **setup.sh** 스크립트를 실행합니다.

    ```azurecli
    cd azure-ml-labs/Labs/03
    ./setup.sh
    ```

    > 확장 기능이 설치되지 않았다는 오류 메시지는 무시합니다.

1. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 5~10분이 걸립니다.

## 기본 데이터 저장소 탐색

Azure Machine Learning 작업 영역을 만들면 스토리지 계정이 자동으로 만들어져 작업 영역에 연결됩니다. 스토리지 계정이 연결되는 방법을 살펴보겠습니다.

1. Azure Portal에서 **rg-dp100-...** 이라는 새 리소스 그룹으로 이동합니다.
1. 리소스 그룹에서 스토리지 계정을 선택합니다. 이름은 작업 영역에 제공한 이름(하이픈 없이)으로 시작되는 경우가 많습니다.
1. 스토리지 계정의 **개요** 페이지를 검토합니다. 개요 창과 왼쪽 메뉴에 표시된 것처럼 스토리지 계정에는 **데이터 스토리지**에 대한 여러 옵션이 있습니다.
1. 스토리지 계정의 Blob Storage 부분을 탐색하려면 **컨테이너**를 선택합니다.
1. **azureml-blobstore-...** 컨테이너를 확인합니다. 데이터 자산의 기본 데이터 저장소는 이 컨테이너를 사용하여 데이터를 저장합니다.
1. 화면 상단의 &#43; **컨테이너** 단추를 클릭하고 새 컨테이너를 만들고 이름을 `training-data`로 지정합니다.
1. 스토리지 계정의 파일 공유 부분을 탐색하려면 왼쪽 메뉴에서 **파일 공유**를 선택합니다.
1. **code-...** 파일 공유를 확인합니다. 작업 영역의 모든 Notebooks는 여기에 저장됩니다. 랩 재질을 복제한 후 **code-.../Users/*your-user-name*/azure-ml-labs** 폴더에서 이 파일 공유의 파일을 찾을 수 있습니다.

## 액세스 키 복사

Azure Machine Learning 작업 영역에서 데이터 저장소를 만들려면 몇 가지 자격 증명을 제공해야 합니다. 작업 영역에 Blob Storage에 대한 액세스 권한을 제공하는 쉬운 방법은 계정 키를 사용하는 것입니다.

1. 스토리지 계정의 왼쪽 메뉴에서 **액세스 키** 탭을 선택합니다.
1. key1과 key2라는 두 개의 키가 제공됩니다. 각 키에는 동일한 기능이 있습니다. 
1. **key1** 아래의 **키** 필드에 대해 **표시**를 선택합니다.
1. **키** 필드의 값을 메모장에 복사합니다. 나중에 이 값을 Notebook에 붙여넣어야 합니다.
1. 페이지 상단에서 스토리지 계정의 이름을 복사합니다. 이름은 **mlwdp100storage...** 로 시작해야 합니다. 나중에 이 값을 Notebook에도 붙여넣어야 합니다.

> **참고**: 자동 대문자화(Word에서 발생)를 방지하려면 계정 키와 이름을 메모장에 복사합니다. 키는 대/소문자를 구분합니다.

## 랩 자료 복제

Python SDK를 사용하여 데이터 저장소와 데이터 자산을 만들려면 랩 재질을 작업 영역에 복제해야 합니다.

1. Azure Portal에서 **mlw-dp100-labs**라는 Azure Machine Learning 작업 영역으로 이동합니다.
1. Azure Machine Learning 작업 영역을 선택하고 **개요** 페이지에서 **스튜디오 시작**을 선택합니다. Azure Machine Learning 스튜디오를 열 수 있는 또 다른 탭이 브라우저에 열립니다.
1. 스튜디오에 나타나는 팝업을 모두 닫으세요.
1. Azure Machine Learning 스튜디오 내에서 **컴퓨팅** 페이지로 이동하여 이전 섹션에서 만든 컴퓨팅 인스턴스와 클러스터가 있는지 확인합니다. 컴퓨팅 인스턴스가 실행 중이어야 하고, 클러스터는 유휴 상태여야 하며 실행 중인 노드가 0개여야 합니다.
1. **컴퓨팅 인스턴스** 탭에서 컴퓨팅 인스턴스를 찾고 **터미널** 애플리케이션을 선택합니다.
1. 터미널에서 다음 명령을 실행하여 컴퓨팅 인스턴스에 Python SDK를 설치합니다.

    ```azurecli
    pip uninstall azure-ai-ml
    pip install azure-ai-ml
    pip install mltable
    ```

    > 패키지가 설치되지 않았다는 내용의(오류) 메시지는 무시합니다.

1. 다음 명령을 실행하여 Notebooks, 데이터 및 기타 파일이 포함된 Git 리포지토리를 작업 영역에 복제합니다.

    ```azurecli
    git clone https://github.com/MicrosoftLearning/mslearn-azure-ml.git azure-ml-labs
    ```

1. 명령이 완료되면 **파일** 창에서 **&#8635;** 를 클릭하여 보기를 새로 고치고 **Users/*your-user-name */azure-ml-labs** 폴더가 새로 만들어졌습니다.

**선택적으로** 다른 브라우저 탭에서 [Azure Portal](https://portal.azure.com?azure-portal=true)로 다시 이동합니다. 스토리지 계정의 파일 공유 **code-...** 를 다시 탐색하여 새로 만들어진 **azure-ml-labs** 폴더에서 복제된 랩 재질을 찾습니다.

## 데이터 저장소 및 데이터 자산 만들기

Python SDK를 사용하여 데이터 저장소와 데이터 자산을 만드는 코드는 Notebook에 제공됩니다.

1. **Labs/03/Work with data.ipynb** Notebook을 엽니다.

    > **인증**을 선택하고 인증을 요청하는 알림이 표시되면 필요한 단계를 따릅니다.

1. Notebook이 **Python 3.8 - AzureML** 커널을 사용하는지 확인합니다.
1. Notebook의 모든 셀을 실행합니다.

## 선택 사항: 데이터 자산 살펴보기

**선택적으로** 데이터 자산이 연결된 스토리지 계정에 저장되는 방식을 탐색할 수 있습니다.

1. 데이터 자산을 탐색하려면 Azure Machine Learning 스튜디오의 **데이터** 탭으로 이동합니다.
1. 세부 정보를 살펴보려면 **diabetes-local** 데이터 자산 이름을 선택합니다. 

    **diabetes-local** 데이터 자산의 **데이터 원본**에서 파일이 업로드된 위치를 확인할 수 있습니다. **LocalUpload/...** 로 시작하는 경로는 스토리지 계정 컨테이너 **azureml-blobstore-...** 내의 경로를 표시합니다. Azure Portal에서 해당 경로로 이동하여 파일이 존재하는지 확인할 수 있습니다.

## Azure 리소스 삭제

Azure Machine Learning 탐색을 마치면 지금까지 만든 리소스를 삭제하여 불필요한 Azure 비용을 방지해야 합니다.

1. Azure Machine Learning 스튜디오 탭을 닫고 Azure Portal로 돌아갑니다.
1. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
1. **rg-dp100-...** 리소스 그룹을 선택합니다.
1. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음 **삭제**를 선택합니다.
