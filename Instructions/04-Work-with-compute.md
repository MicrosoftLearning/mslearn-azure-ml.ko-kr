---
lab:
  title: Azure Machine Learning에서 컴퓨팅 리소스 작업
---

# Azure Machine Learning에서 컴퓨팅 리소스 작업

클라우드의 주요 이점 중 하나는 대규모 데이터의 비용 효율적인 처리를 위해 스케일링 가능한 주문형 컴퓨팅 리소스를 사용하는 기능입니다.

이 연습에서는 Azure Machine Learning에서 클라우드 컴퓨팅을 사용하여 대규모 실험 및 프로덕션 코드를 실행하는 방법을 알아봅니다.

## 시작하기 전에

관리 수준 액세스 권한이 있는 [Azure 구독](https://azure.microsoft.com/free?azure-portal=true)이 필요합니다.

## Azure Machine Learning 작업 영역 프로비저닝

Azure Machine Learning 작업 영역은 모델을 학습하고 관리하는 데 필요한 모든 리소스 및 자산을 관리하기 위한 중심지를 제공합니다.** 스튜디오, Python SDK, Azure CLI를 통해 Azure Machine Learning 작업 영역과 상호 작용할 수 있습니다.

Azure Machine Learning 작업 영역을 만들려면 Azure CLI를 사용합니다. 필요한 모든 명령은 실행할 수 있도록 셸 스크립트로 그룹화됩니다.

1. 브라우저에서 `https://portal.azure.com/`에서 Azure Portal을 열고 Microsoft 계정으로 로그인합니다.
1. 검색 상자 오른쪽 페이지 맨 위에 있는 \[>_](*Cloud Shell*) 단추를 선택합니다. 그러면 포털 아래쪽에 Cloud Shell 창이 열립니다.
1. 메시지가 표시되면 **Bash**를 선택합니다. Cloud Shell을 처음 열면 사용할 셸 유형(Bash 또는 PowerShell)을 선택하라는 메시지가 표시됩니다.****
1. 올바른 구독이 지정되어 있고 **필요한 스토리지 계정이 선택되어 있지 않은지** 확인합니다. **적용**을 선택합니다.
1. 이전 버전과의 충돌을 방지하려면 터미널에서 다음 명령을 실행하여 ML CLI 확장(버전 1 및 2 모두)을 제거합니다.

    ```azurecli
    az extension remove -n azure-cli-ml
    az extension remove -n ml
    ```

    > `SHIFT + INSERT`를 사용하여 복사한 코드를 Cloud Shell에 붙여넣습니다.

    > 확장 기능이 설치되지 않았다는 오류 메시지는 무시합니다.

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

1. 명령이 완료될 때까지 기다립니다. 일반적으로 5~10분 정도 걸립니다.

    <details>  
    <summary><b>문제 해결 팁</b>: 작업 영역 만들기 오류</summary><br>
    <p>CLI를 통해 작업 영역을 만들 때 오류가 발생하는 경우 리소스를 수동으로 프로비전해야 합니다.</p>
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
    </ol>
    </details>

## 컴퓨팅 설정 스크립트 만들기

Azure Machine Learning 작업 영역 내에서 Notebooks를 실행하려면 컴퓨팅 인스턴스가 필요합니다. 설정 스크립트를 사용하여 만들 때 컴퓨팅 인스턴스를 구성할 수 있습니다.

1. Azure Portal에서 **mlw-dp100-labs**라는 Azure Machine Learning 작업 영역으로 이동합니다.
1. Azure Machine Learning 작업 영역을 선택하고 **개요** 페이지에서 **스튜디오 시작**을 선택합니다. Azure Machine Learning 스튜디오를 열 수 있는 또 다른 탭이 브라우저에 열립니다.
1. 스튜디오에 나타나는 팝업을 모두 닫으세요.
1. Azure Machine Learning 스튜디오 내에서 **Notebooks** 페이지로 이동합니다.
1. **파일** 창에서 **파일 추가** &#10753; 아이콘을 클릭합니다.
1. **새 파일 만들기**를 선택합니다.
1. 파일 위치가 **Users/* your-user-name***인지 확인합니다.
1. 파일 형식을 **Bash(*.sh)** 로 변경합니다.
1. 파일 이름을 `compute-setup.sh`로 변경합니다.
1. 새로 만들어진 **compute-setup.sh** 파일을 열고 다음 콘텐츠를 해당 콘텐츠에 붙여넣습니다.

    ```azurecli
    #!/bin/bash

    # clone repository
    git clone https://github.com/MicrosoftLearning/mslearn-azure-ml.git azure-ml-labs
    ```

1. **compute-setup.sh** 파일을 저장합니다.

## 컴퓨팅 인스턴스 만들기

컴퓨팅 인스턴스를 만들려면 Studio, Python SDK 또는 Azure CLI를 사용할 수 있습니다. 스튜디오를 사용하여 방금 만든 설정 스크립트로 컴퓨팅 인스턴스를 만듭니다.

1. 왼쪽 메뉴를 사용하여 **컴퓨팅** 페이지로 이동합니다.
1. **컴퓨팅 인스턴스** 탭에서 **새로 만들기**를 선택합니다.
1. 다음 설정으로 컴퓨팅 인스턴스를 구성합니다(아직 만들지 않음). 
    - **컴퓨팅 이름**: 고유한 이름 입력
    - **가상 머신 유형**: *CPU*
    - **가상 머신 크기**: *Standard_DS11_v2*
1. **다음**을 선택합니다.
1. **일정 추가**를 선택하고 매일 **18:00** 또는 **오후 6시**에 컴퓨팅 인스턴스를 **중지**하도록 일정을 구성합니다.
1. **다음**을 선택합니다.
1. 보안 설정을 검토하되 선택하지 **마세요**.
    - **SSH 액세스 사용**: *이를 사용하면 SSH 클라이언트를 통해 가상 머신에 직접 액세스할 수 있습니다.*
    - **가상 네트워크 사용**: *일반적으로 엔터프라이즈 환경에서 네트워크 보안을 강화하기 위해 이 기능을 사용합니다.*
    - **다른 사용자에게 할당**: *이를 사용하여 컴퓨팅 인스턴스를 다른 데이터 과학자에게 할당할 수 있습니다.*
1. **다음**을 선택합니다.
1. **생성 스크립트로 프로비전** 토글을 선택합니다.
1. 이전에 만든 **compute-setup.sh** 스크립트를 선택합니다.
1. **검토 + 만들기**를 선택하여 컴퓨팅 인스턴스를 만들고 인스턴스가 시작되어 상태가 **실행 중**으로 바뀔 때까지 기다립니다.
1. 컴퓨팅 인스턴스가 실행 중이면 **Notebooks** 페이지로 이동합니다. **파일** 창에서 **&#8635;** 을 클릭하여 보기를 새로 고치고 새 **Users/*your-user-name*/dp100-azure-ml-labs** 폴더가 만들어졌는지 확인합니다.

## 컴퓨팅 인스턴스 구성

컴퓨팅 인스턴스를 만들면 해당 인스턴스에서 Notebooks를 실행할 수 있습니다. 원하는 코드를 실행하려면 특정 패키지를 설치해야 할 수도 있습니다. 설치 스크립트에 패키지를 포함하거나 터미널을 사용하여 설치할 수 있습니다.

1. **컴퓨팅 인스턴스** 탭에서 컴퓨팅 인스턴스를 찾고 **터미널** 애플리케이션을 선택합니다.
1. 터미널에서 다음 명령을 실행하여 컴퓨팅 인스턴스에 Python SDK를 설치합니다.

    ```
    pip uninstall azure-ai-ml
    pip install azure-ai-ml
    ```

    > 패키지가 설치되지 않았다는 내용의(오류) 메시지는 무시합니다.

1. 패키지가 설치되면 탭을 닫아 터미널을 종료할 수 있습니다.

## 컴퓨팅 클러스터 만들기

Notebooks는 실험 중 개발 또는 반복 작업에 이상적입니다. 실험할 때 컴퓨팅 인스턴스에서 Notebooks를 실행하여 코드를 빠르게 테스트하고 검토할 수 있습니다. 프로덕션으로 전환할 때 컴퓨팅 클러스터에서 스크립트를 실행하려고 합니다. Python SDK를 사용하여 컴퓨팅 클러스터를 만든 다음 이를 사용하여 스크립트를 작업으로 실행합니다.

1. **Labs/04/Work with compute.ipynb** Notebook을 엽니다.

    > **인증**을 선택하고 인증을 요청하는 알림이 표시되면 필요한 단계를 따릅니다.

1. Notebook이 **Python 3.8 - AzureML** 커널을 사용하는지 확인합니다.
1. Notebook의 모든 셀을 실행합니다.

## Azure 리소스 삭제

Azure Machine Learning 탐색을 마치면 지금까지 만든 리소스를 삭제하여 불필요한 Azure 비용을 방지해야 합니다.

1. Azure Machine Learning 스튜디오 탭을 닫고 Azure Portal로 돌아갑니다.
1. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
1. **rg-dp100-labs** 리소스 그룹을 선택합니다.
1. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음 **삭제**를 선택합니다.
