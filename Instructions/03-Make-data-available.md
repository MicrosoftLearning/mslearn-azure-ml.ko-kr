---
lab:
  title: Azure Machine Learning에서 데이터를 사용할 수 있도록 만들기
---

# Azure Machine Learning에서 데이터를 사용할 수 있도록 만들기

일반적으로는 로컬 파일 시스템의 데이터를 사용하는 경우가 많습니다. 하지만 엔터프라이즈 환경에서는 여러 데이터 과학자 및 기계 학습 엔지니어가 액세스할 수 있는 중앙 위치에 데이터를 저장하는 방식이 더 효과적일 수 있습니다.

이 연습에서는 Azure Machine Learning에서 데이터 액세스를 추상화하는 데 사용되는 기본 개체인 데이터 *저장소* 및 데이터 *자산을* 살펴봅니다.

## 시작하기 전에

관리 수준 액세스 권한이 있는 [Azure 구독](https://azure.microsoft.com/free?azure-portal=true)이 필요합니다.

## Azure Machine Learning 작업 영역 프로비저닝

Azure Machine Learning 작업 영역은 모델을 학습하고 관리하는 데 필요한 모든 리소스 및 자산을 관리하기 위한 중심지를 제공합니다. 스튜디오, Python SDK 및 Azure CLI를 통해 Azure Machine Learning 작업 영역과 상호 작용할 수 있습니다. 

Azure CLI를 사용하여 작업 영역 및 필요한 리소스를 프로비전하는 셸 스크립트를 사용합니다. 다음으로, Azure Machine Learning 스튜디오 디자이너를 사용하여 모델을 학습시키고 비교합니다.

### 작업 영역 및 컴퓨팅 리소스 만들기

Azure Machine Learning 작업 영역 및 컴퓨팅 리소스를 만들려면 Azure CLI를 사용합니다. 필요한 모든 명령은 실행할 수 있도록 셸 스크립트로 그룹화됩니다.
1. 브라우저에서 에서 `https://portal.azure.com/`Azure Portal 열고 Microsoft 계정으로 로그인합니다.
1. 검색 상자 오른쪽 페이지 맨 위에 있는 \[>_](*Cloud Shell*) 단추를 선택합니다. 그러면 포털 아래쪽에 Cloud Shell 창이 열립니다.
1. 메시지가 표시되면 **Bash** 를 선택합니다. Cloud Shell을 처음 열면 사용할 셸 유형(Bash 또는 PowerShell)을 선택하라는 메시지가 표시됩니다.  
1. 올바른 구독이 지정되었는지 확인하고 클라우드 셸에 대한 **스토리지** 를 만들라는 메시지가 표시되면 스토리지 만들기를 선택합니다. 스토리지가 생성될 때까지 기다립니다.
1. 터미널에서 다음 명령을 입력하여 이 리포지토리를 복제합니다.

    ```azurecli
    rm -r azure-ml-labs -f
    git clone https://github.com/MicrosoftLearning/mslearn-azure-ml.git azure-ml-labs
    ```

    > 복사한 코드를 Cloud Shell 붙여넣는 데 사용합니다`SHIFT + INSERT`. 

1. 리포지토리가 복제된 후 다음 명령을 입력하여 이 랩의 폴더로 변경하고 포함된 **setup.sh** 스크립트를 실행합니다.

    ```azurecli
    cd azure-ml-labs/Labs/03
    ./setup.sh
    ```

    > 확장이 설치되지 않았다는 (오류) 메시지를 무시합니다. 

1. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 5~10분이 걸립니다. 

## 기본 데이터 저장소 살펴보기

Azure Machine Learning 작업 영역을 만들면 스토리지 계정이 자동으로 만들어지고 작업 영역에 연결됩니다. 스토리지 계정이 연결된 방법을 살펴봅니다.

1. Azure Portal **rg-dp100-labs**라는 새 리소스 그룹으로 이동합니다.
1. 리소스 그룹에서 스토리지 계정을 선택합니다. 이름은 종종 작업 영역에 대해 제공한 이름으로 시작됩니다(하이픈 없음).
1. 스토리지 계정의 **개요** 페이지를 검토합니다. 스토리지 계정에는 개요 창 및 왼쪽 메뉴와 같이 **데이터 스토리지** 에 대한 몇 가지 옵션이 있습니다.
1. **컨테이너를** 선택하여 스토리지 계정의 Blob Storage 부분을 탐색합니다. 
1. **azureml-blobstore-... 컨테이너에** 유의하세요. 데이터 자산의 기본 데이터 저장소는 이 컨테이너를 사용하여 데이터를 저장합니다. 
1. 화면 맨 위에 있는 &#43; **컨테이너** 단추를 사용하여 새 컨테이너를 만들고 이름을 로 지정합니다 `training-data`. 
1. 왼쪽 메뉴에서 **파일 공유** 를 선택하여 스토리지 계정의 파일 공유 부분을 탐색합니다.
1. **코드-...** 파일 공유를 확인합니다. 작업 영역의 모든 Notebook은 여기에 저장됩니다. 랩 자료를 복제한 후 **코드-.../Users/*your-user-name*/azure-ml-labs** 폴더에서 이 파일 공유의 파일을 찾을 수 있습니다.

## 액세스 키 복사

Azure Machine Learning 작업 영역에서 데이터 저장소를 만들려면 몇 가지 자격 증명을 제공해야 합니다. Blob Storage에 대한 액세스 권한을 작업 영역에 제공하는 쉬운 방법은 계정 키를 사용하는 것입니다.

1. 스토리지 계정의 왼쪽 메뉴에서 **액세스 키** 탭을 선택합니다.
1. key1 및 key2라는 두 개의 키가 제공됩니다. 각 키에는 동일한 기능이 있습니다. 
1. **key1** 아래의 **키** 필드에 대해 **표시**를 선택합니다.
1. **키** 필드의 값을 메모장에 복사합니다. 이 값은 나중에 Notebook에 붙여넣어야 합니다. 
1. 페이지 위쪽에서 스토리지 계정의 이름을 복사합니다. 이름은 **mlwdp100storage...** 로 시작해야 합니다. 나중에 전자 필기장에도 이 값을 붙여넣어야 합니다. 

> **참고**: 자동 대문자(Word에서 발생)를 방지하기 위해 계정 키와 이름을 메모장에 복사합니다. 키는 대/소문자를 구분합니다.

## 랩 자료 복제

Python SDK를 사용하여 데이터 저장소 및 데이터 자산을 만들려면 랩 자료를 작업 영역에 복제해야 합니다.

1. Azure Portal **mlw-dp100-labs**라는 Azure Machine Learning 작업 영역으로 이동합니다.
1. Azure Machine Learning 작업 영역을 선택하고 **개요** 페이지에서 **스튜디오 시작을** 선택합니다. 브라우저에서 다른 탭이 열리고 Azure Machine Learning 스튜디오 열립니다.
1. 스튜디오에 표시되는 팝업을 닫습니다.
1. Azure Machine Learning 스튜디오 내에서 **컴퓨팅** 페이지로 이동하여 이전 섹션에서 만든 컴퓨팅 인스턴스 및 클러스터가 있는지 확인합니다. 컴퓨팅 인스턴스가 실행되고 클러스터가 유휴 상태이고 0개의 노드가 실행 중이어야 합니다.
1. **컴퓨팅 인스턴스** 탭에서 컴퓨팅 인스턴스를 찾고 **터미널** 애플리케이션을 선택합니다.
1. 터미널에서 터미널에서 다음 명령을 실행하여 컴퓨팅 인스턴스에 Python SDK를 설치합니다.
    
    ```
    pip uninstall azure-ai-ml
    pip install azure-ai-ml
    pip install mltable
    ```

    > 패키지가 설치되지 않았다는 (오류) 메시지를 무시합니다.

1. 다음 명령을 실행하여 Notebook, 데이터 및 기타 파일이 포함된 Git 리포지토리를 작업 영역에 복제합니다.
    
    ```
    git clone https://github.com/MicrosoftLearning/mslearn-azure-ml.git azure-ml-labs
    ```
 
1. 명령이 완료되면 **파일** 창에서 **&#8635;** 클릭하여 보기를 새로 고치고 새 **Users/*your-user-name*/azure-ml-labs** 폴더가 만들어졌는지 확인합니다. 

**필요에 따라** 다른 브라우저 탭에서 [Azure Portal](https://portal.azure.com?azure-portal=true) 다시 이동합니다. 스토리지 계정에서 파일 공유 **코드-...** 다시 탐색하여 새로 만든 **azure-ml-labs** 폴더에서 복제된 랩 자료를 찾습니다.

## 데이터 저장소 및 데이터 자산 만들기

Python SDK를 사용하여 데이터 저장소 및 데이터 자산을 만드는 코드는 Notebook에 제공됩니다.

1. **Labs/03/work with data.ipynb** Notebook을 엽니다.

    > **인증을** 선택하고 인증을 요청하는 알림이 표시되면 필요한 단계를 따릅니다. 

1. Notebook에서 **Python 3.8 - AzureML** 커널을 사용하는지 확인합니다. 
1. Notebook의 모든 셀을 실행합니다.

## 선택 사항: 데이터 자산 탐색

**필요에 따라** 데이터 자산이 연결된 스토리지 계정에 저장되는 방법을 살펴볼 수 있습니다.

1. Azure Machine Learning 스튜디오 **데이터** 탭으로 이동하여 데이터 자산을 탐색합니다. 
1. **당뇨병-로컬** 데이터 자산 이름을 선택하여 세부 정보를 탐색합니다. 

    **diabetes-local** 데이터 자산의 데이터 **원본**에서 파일이 업로드된 위치를 찾을 수 있습니다. **LocalUpload/...** 로 시작하는 경로에는 스토리지 계정 컨테이너 **azureml-blobstore-...** 내의 경로가 표시됩니다. Azure Portal 해당 경로로 이동하여 파일이 있는지 확인할 수 있습니다.

## Azure 리소스 삭제

Azure Machine Learning 탐색을 마치면 지금까지 만든 리소스를 삭제하여 불필요한 Azure 비용을 방지해야 합니다.

1. Azure Machine Learning 스튜디오 탭을 닫고 Azure Portal 돌아갑니다.
1. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
1. **rg-dp100-labs** 리소스 그룹을 선택합니다.
1. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다. 
1. 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음 **삭제**를 선택합니다.
