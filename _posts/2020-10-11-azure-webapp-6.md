---
layout: post
title: "Public Cloud 전환 프로젝트 : 6부 강제 멘탈 강화 훈련 : Azure Data Disk Box 950만개 컨텐츠 파일의 비밀"
date: 2020-10-11 01:30:13
img: azure-webapp-404.png # Add image post (optional)
categories: cloud azure
tags: cloud azure MS
---
[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2Fgraudis%2Fgraudis.github.io%2Fblob%2Fmaster%2F_posts%2F2020-10-04-azure-webapp-6.md&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)

​	**필자연역 :** Linux 을 15년 이상 해왔으며, Linux 기반의 Enterprise 인프라 구축 엔지니어 생활을 8년 동안 하였으며, OpenSource 구축 및 기술지원 분야에서 다양한 경험을 하였다.

​	지금까지의 필자의 경력은 주로 On-Premis Enterprise 기업환경의 인프라 구축 및 기술 지원등의 TA 역활을 다년간 진행하였기에 Public Cloud 전환 프로젝트는 또다른 경험을 하는 기회였다.

필자의 주분야는 OpenSource WEB/WAS 분야 이며, 설계/구축/장애 기술지원에 다양한 경험을 가지고 있다.

* **프리랜서 TA 로 프로젝트를 진행 하다보니 이제서야 연재를 작성할 시간적 여유가 생겨 늦게 나마 이야기를 이어 나가고자 한다.**



#### 6. Azure Data Disk Box 950만개 컨텐츠 파일의 비밀( 프로젝트 팀원에게 받은 빅선물 : 고맙다! 이관된 파일 재검 전수조사! )"

  **" Custom Docker Image 적용을 위하여 필자가 한참 Docker Image 를 Build "** 하고 있을 때의 시점의 이야기 이다.



  On-Premise 시스템에 연동되어 있는 AS-IS Storage 에 저장되어 있는 32TB 용량의 User Contents Data 의 이관이 큰 문제로 떠오르기 시작하였다. 

용량도 용량이지만, 950만개의 User Contents Data 파일을 BLOB Storage 에 어떻게 이관 하냐는 것이 필자의 머리를 아프게 했다.

 

1. **" 물리적 데이터 이관 : Azure Data Disk Box "**

   : On-Premise 에 있는 User Contents Data 를 Cloud  내의 BLOB Storage 로 이관해야 하는데 그 용량이 대략 34TB 950만건의 파일들이다. Network 으로 전송하기엔 WBS 에 계획된 Data 이관 스케쥴이 터무니 없이 부족하였다. 

   

   따라서, 필자는 고객사의 Data Center 내의 있는 Storage 의 Data 를 물리적 Disk 를 이용하여 Data 복사하여 Cloud 벤더사 Data Center 내로 반입하여 직접 전송 하는 방식을 선택하였다.

   

   위와같은 벤더사의 서비스가 바로 **" Azure Data Disk Box "** 이다.

   

   다음은 벤더사의 " Azure Data Disk Box " 에 관한 자료 이다.

   

   **[ Azure Data Disk Box ]**

   - **서비스 흐름도**

     ![Azure_Data_Disk_Box1](https://github.com/graudis/graudis.github.io/blob/master/_image/data-box-disk-security-1.png?raw=true)

     

   1. Azure에서 호스팅되는 Azure Data Box 서비스 - 디스크 순서를 만들고, 디스크를 구성한 다음 완료에 대한 순서를 추적하는 데 사용하는 관리 서비스입니다.

      

   2. Data Box Disk – Azure로 온-프레미스 데이터를 가져올 수 있도록 함께 제공되는 실제 디스크입니다.

      

   3. 디스크에 연결된 클라이언트/호스트 – USB를 통해 Data Box 디스크에 연결하고 보호되어야 하는 데이터를 포함하는 인프라 내의 클라이언트입니다.

      

   4. 클라우드 저장소 – Azure 클라우드 내에서 데이터가 저장되는 위치입니다. 이것이 일반적으로 사용자가 만든 Azure Data Box 리소스에 연결된 스토리지 계정입니다.

   

   자세한 내용은 다음 URL 의 내용을 참고 할 것.

   https://docs.microsoft.com/ko-kr/azure/databox/data-box-disk-security

   

   - **구성품**

   ![Azure_data_disk_box2](https://github.com/graudis/graudis.github.io/blob/master/_image/data-box-disk-ship-package1.png?raw=true)

   

   1. Data Box Disks는 작은 배송 상자에 넣어 발송됩니다. 상자를 열고 해당 콘텐츠를 제거합니다. 상자에 1~5개의 SSD(반도체 디스크) 및 디스크당 USB 연결 케이블이 있는지 확인합니다. 변조의 증거 또는 기타 명백한 손상에 대해 상자를 검사합니다.

      

   2. 배송 상자가 훼손되었거나 심각하게 손상된 경우 상자를 열지 마십시오. 디스크가 올바른 작업 주문에 있는지 여부를 평가하는 데 도움을 얻고 대체물을 배송 받아야 하는지 Microsoft 지원에 문의합니다.

      

   3. 상자에 반송 배송을 위한 포장용 레이블(현재 레이블 아래)을 포함하는 투명한 케이스가 있는지 확인합니다. 이 레이블이 손실되거나 손상된 경우 Azure Portal에서 항상 새 레이블을 다운로드하고 인쇄할 수 있습니다.

   

   - **디스크에 연결 및 암호 가져오기**

   ![Azure_data_disk_box3](https://github.com/graudis/graudis.github.io/blob/master/_image/data-box-disk-connect-unlock.png?raw=true)

   1. 포함된 케이블을 사용하여 필수 구성 요소에서 설명한 것처럼 지원되는 OS를 실행하는 클라이언트 컴퓨터에 디스크를 연결합니다.

      

      ![Azure_data_disk_box4](https://github.com/graudis/graudis.github.io/blob/master/_image/data-box-disk-get-passkey.png?raw=true)

   2. Azure Portal에서 Data Box Disk Order로 이동합니다. **일반 > 모든 리소스**로 이동하여 검색한 다음, Data Box Disk Order를 선택합니다. 복사 아이콘을 사용하여 암호를 복사합니다. 이 암호는 디스크의 잠금을 해제하는 데 사용됩니다

   

   자세한 내용은 다음 URL 을 참고 할 것

   https://docs.microsoft.com/ko-kr/azure/databox/data-box-disk-deploy-set-up

   

   **[ Azure Data Disk Box : 주의점 ]**

   1. **USB Interface 한계 : Azure Data Disk Box 는 기본적으로 USB 3.0 을 지원**한다.

      **USB Interface 에서 대용량 Data 를 복사 할때 흔히 발생하는 Data 병목 현상으로 인한 데이터 복사 속도가 시간이 가면 갈수록 느려진다.** 

      이는 **직렬 버스 구조인 USB Interface 데이터 전송방식에서 흔하게 볼수 있는 현상**이다.

      여기서 문제가 발생한다. 

      **USB Interface 의 스펙 Speed 속도는 순간 최대 속도를 표시 할 뿐 Concurrent translation data speed 는 아니라는 것**을 대부분 모르고 있다.

      USB 3.0 사용시에는 보조 전원을 같이 사용해야 그나마 어느정도는 Hispeed 를 보장 할 수 있다.

      ( 보조 전원 없이 사용 할 경우 USB 케이블의 길이에 따라 전송 속도가 달라진다.)

      Azure Data Disk Box 의 USB Interface 는 3.0 규격 이지만 보조 전원 케이블이 없다.

      

      **필자 의견 : ** 위의 **"Azure Data Disk Box : 주의점"** 이  Azure 만의 문제점은 아니다. **Public Cloud 벤더사에서 제공하는 Offline Data 이관 Solution 들은 그 명칭만 다를뿐 대부분이 위와 같은 USB Interface 연결 방식으로 물리적 Disk 를 제공하는 서비스**로 알고 있다.

      따라서, 위에서 설명한 주의점은 **Offline 형태로 Data 를 이관할때 공통적으로 생각할 문제점이라고 생각 하여 언급** 하였다.

   

   **[ Contents Data 이관 ]**

   1. On-Premise Storage 에 있는 고객의 Contents Data 를 Azure BLOB Storage 로 이관 해야 한다.  프로젝트에서 Azure 인프라를 담당하기로 하였던 자사 Azure 전문가(?) 인프라 에 Data 이관 업무를 지시하였다. 

      ( 아무것도 안하고 놀고 있으니 최소한 File Copy 는 할줄  알겠지 하는 생각으로 업무를 맏겼으나, 이 생각이 나중에 어마 어마한 후폭풍으로 되돌아와 필자를 고객사의 볼모로 붙잡히게 되는 결과를 낳게 될지는 그때는 알지 못했었다. )

      

   2. 32TB / 4.7TB = 7 개의 Disk 가 소요 되었다. 1 Disk 사용 비용은 정액 비용으로 처리 되어 비용이 지불 되었다. **자사 인프라 엔지니의 전무한 Data 이관 경험능력치 와 Data 이관에 관한 기초 지식의 부제, BLOB Storage 대한 지식 무지로 인하여 고객의 32TB Contents Data 이관하는데 2개월 이상이 소요** 되어 고객의 또다른 불만요인으로 작용하였다.

      

   3. 우여 곡절 끝에 1개월 이상의 Data 이관 작업이 마무리 되어 BLOB Storage 에 고객의 Contents Data 이관이 모두 완료 되었다. 

      

   **[ 악연의 연속 1 : Linux 를 몰라서요?! ]**

   1. Data 이관 작업을 진행 하는 자사 인프라 엔지니어 에게 **Data 이관에 사용할 Host 를 "Linux " 서버를 사용하여 이관 작업을 진행 할 것을 지시** 하였다. 

      이유는 **Data 이관 대상 File 들의 List 를 Log 로 기록하여야 Data 이관 누락 발생시 누락된 파일을 쉽게 찾아 내기 위해서**였다.

      하지만, Data 이관 작업에 사용할 Host 는 " Windows" 서버로 진행 하겠다고 고객에게 PM 과의 상의도 없이 통보를 해버린 것이다. 

      WebApp 사건 이후로 고객에게  프로젝트 수행팀 내부의 불화를 또 보여주는건 프로젝트 진행에 전혀 도움이 되지 않는 판단에 높은 스킬이 필요하지 않는 Data 이관 작업에 대해서 설마, 문제가 발생 하겠어? 라는 생각이 들었다.

      하지만, PM 의 의견을 자기마음대로 무시하고 고객에게 통보한 이유에 대해서는 묻지 않을 수 없었다. 

      이유를 물어 보니, 상상을 뛰어 넘는 대답이 나왔다. **" Linux 를 몰라서요. " 자기는 Windows 엔지니어 라서 Windows 로 작업 하겠다는 말**이다.

      

      물론, Linux 를 모르는 것이 잘못은 아니다. 

      하지만, 고객에게 통보하기 전에 PM 과 사전 협의 후 고객과의 의사 결정이 일의 순서가 아닌가. 경력도 많고 나이도 있는 엔지니어가 도저히 믿기 힘는 행동을 함에 있어 상상 이하의 일들이 발생함에 있어 멘붕이 왔다.

      **에휴, 제발 Data 이관시 Data 만 누락 시키지 않길 바랄뿐.................**

   

   **[ 악연의 연속 2 : 엑박들의 습격( 이관 Data 누락 발생) ]**

   1. Data 이관 작업을 담당한 자사 엔지니어가 작업하는 " Windows " Host Server 에 " RoboCopy " 라는 Tool 을 이용하여 2개월 이상을 Azure Data Disk Box 를 이용한 Data 가 거의 다 끝나 갈쯤 **고객에게서 " Contents Image Data " 가 출력되지 않고 " 엑스박스 " 로 표기 된다는 급한 연락**을 받게 되었다. 

      

      확인 결과 **DB 에는 해당 Contents Image File 을 호출 하고 있으나 이관이 완료된 BLOB Storage 에는 해당 Contents Image File 이 존재하지 않았다.** 

      처음은 단순하게 파일 몇개 만이 누락된 줄 알고 AS-IS Storage 에서 해당 파일을 찾아 복사하여 복구 하였다. 

      

      하지만, **이런 일이 반복적으로 발생하자 고객은 Data 이관 작업을 진행한 자사 엔지니어를 의심하기 시작하여 PM 인 필자에게 " Data 이관 작업에 때른 File 전수 조사 작업" 을 공식적으로 요청**하기 에 이르렀다. 

      

      **필자는 고객의 요청에 " Data 이관 작업 "을 진행한 자사 엔지니어에게 Data 이관 작업에서 이관된 File 들의 List Log 자료를 요청**하였다. 

      AS-IS 의 Data File List 와 비교 하여 누락된 File 을 찾고자 하였다. 

      

      그러나!!! 여기서 기대를 저버리지 않게 **" File List Log 는 없다. RoboCoy Tool 에서는 복사가 완료된 파일 이름을 Log 로 남기지 않는다.!!! " 라고 당당하게 말하는 자사 엔지니어** 모습에 정신이 아득해짐을 느꼈다. 

      나중에 안일이지만 **" RoboCopy 에서 복사된 File Name List Log 를 생성할 수 있다는 것을 사원 엔지니어 작업 내용을 보고 알게 되었다. "** 

      

   **[ 악연의 연속 3 : 950만건의 File 전수조사 ( 950만건 X 2 = 1,900만건 ) ]**

   1. **고객 요청 사항**은 다음과 같았다.

      . **950만건 전수조사 작업은 Data 이관 작업을 진행한 엔지니어가 아니라 필자인 PM 이 직접 전수 조사 하여 누락분을 찾아 낼것**

      **. AS-SI(950만건) , TO-BE(950만건) = 1,900만건 전수 비교 검사 한 List 생성 후 재출 할 것**

      

      위의 고객 요구 사항이 과도한 요구나 갑의 횡포라고 생각 하지 않았다. 

      **Data 이관 작업시 기본적으로 생성해야 하는 File List Log 에 대해서 기본을 지키지 않았기에 발생한 문제**라고 생각한다.

      

      아무튼 필자는 고객의 요구에 따라 **1,900만건에 관한 1:1 전수 조사를 위하여 AS-IS Storage 에 저장되어 있는 고객 Contents Data File 에 대한 List 를 모두 뽑아내기 위하여 Shell Script 를 작성하여 1주만에 뽑아** 내었다. 

      

      **문제는 TO-BE 인 BLOB Storage 내 있는 Contents File List 를 어떻게 뽑아 내느냐 였다.** 

      

      **쉽게는 BLOB Storage URL 주소로 접근 하여 File List 를 뽑아 내는 방법**이 있으나, 

      **이 방법을 사용하게 되면 BLOB Storage 의 OutBound 트레픽에 관한 과금 폭탄**을 맞게되니 이 방법은 사용할 수가 없는 방법이였다. 

      ( BLOB Storage Outbound 트레픽 과금에 대해서는 **" BLOB Storage 의 불편한 진실 "**에서 상세 설명 예정 )

      

      필자는 **Linux Server 에 OpenSource 를 이용하여 " BLOB Storage " 에 META Data 만을 뽑아 내는 방법을 생각해 냈다.** 이 방법을 이용하여 **BLOB Storage 내의 Contents Data File Name 정보를 가지고 있는 META Data RAW 파일 생성 작업을 1주일동안 작업** 한 후 Sh**ell Script 를 이용하여 AS-IS File List 와 TO-BE File List 를 비교 하기 시작** 하였다. 

      

      **File List 비교 결과 누락 및 파일명 변경으로 인하여 Data 이관이 안된 파일 건수가 10만건**이 되었다. **( 아.... 고맙다. 나에게 이런 빅 선물을 줘서... )**

      

      **1주일간 필자는 누락된 10만건의 Contents Data File 을 일일히 AS-IS Storage 에서 찾아 TO-BE BLOB Storage 로 모두 이관 복사** 하였다. 

      

      **고객은 서비스 CutOver 이후에도 돌발적으로 발생할지 모르는 Data 이관 누락 File 분에 대해서 1~2개월간 필자만을 상주 지원 하는 조건으로 프로젝트 완료를 승인** 하였따.

   

   **[ 10만건 Contents Data File 누락 원인은 " Windows " 가 범인 ]** 

   1. **"CONTENTS.DAT" 파일과 " contents.dat " 파일이 같은 파일 이라고 ?**

      : **Windows OS 는 대소문자 파일명을 구별하지 못한다.** 

      따라서, 동일한 파일명의 대문자로된 파일과 소문자로 된 파일이 동시에 존재하면 제일먼저 읽은 파일만 인정하고 다음에 오는 동일 이름의 파일은 무시하는 심각한 버그가 있다.

      **이 버그는 Windows 10 2017 3월 패치에서나 겨우 패치 되었다.**

      Data 이관 업무를 했던 자사 엔지니어가 작업한 Windows 버전이 10 이하에서 작업 하였다. 당연히 위의 버그가 패지 되지 않은 버전이였다. 

      예고된 인재 인것이다. 

      

   2. **UNIX/Linux 타입의 긴파일명이 8자리.3자리 DOS 타입 파일명으로 잘리는 버그.**

      : **Windows OS 에서 Unix/Linux 타입의 긴파일명과 3자리 이상 파일 확장자를 8자라.3자리 로 잘라버리는 버그가 존재함.**

      **여기서도 Windows 의 버그와 AS-IS Data 이관에 관한 기초 지식이나 경험이 전무하여 발생한 인재** 이다.

      **Windows 10 버전 에서는 1607 버전에서 긴파일명 짤림 버그가 패치** 되었는데, 패치 설치 이후에는 레지스트를 일부 수정해줘야 긴파일명이 짤리지 않게 된다.

      ( 그래서 Data 이관 작업 할때 Linux Server 에서 작업 하라고 했건만 )

      

      

   **[ 필자 의견 : Data 이관에 관하여 ]**

   1. **On-Premise 상의 대용량 Data 이관 경험이 많은 엔지니어에게 맏겨라.**

      : 대용량 Data 이관 경험이 없는 엔지니어가 Data 이관 작업 할 경우 언제나 사고가 터지게 된다. 

      **이는 Unix/linux Data 에 대한 이해도와 이관작업의 기본에 대해서 경험이 부족하기 때문이다.**

      

   2. **인프라 엔지니어의 강력한 스킬은 솔직하게 자신의 수준을 인정하는 것이다.**

      : **본인 스스로의 스킬과 경험을 냉정하게 인지 하지 않고 과대/과장 하거나 변명으로 순간을 모면 하려 하다보면 더 큰 거짓말을 하게 된다.** 이러한 자세는 **프로젝트를 진행하는 같은 팀원들에게 엄청한 민폐**로 되돌아 오게 된다.
      **본인이 모른다고 고객에게 " 그건 안됩니다. " 라는 말을 하는건 " 손바닥으로 하늘을 가리고 하늘이 없다고 말하는 일 "** 이라는 것을 왜 모르는지 모르겠다. 

      

   **[ 푸념 ]**

   . 이번 포스트는 두서 내용이 두서 없이 쓴것 같다. 

   필자가 950만건의 파일을 전수 조사 할당시의 심정이 얼마나 멘붕이였지를 글로 표현 하기에 다소 무리가 있다.

   하지만, **팀원의 무지와 변명으로 인하여 구지 하지 않아도될 작업에 대해서 많은 시간과 노력이 소모** 되었으며, **소모된 시간과 노력에도 불구 하고 고객의 불신의 벽이 더 높아졌다**는 것에 대해서 **프로젝트 팀원에 대한 자질 검증에 대해서 많은 생각**을 하게 한 사건이다. 



== 7부 에서 계속 ==
