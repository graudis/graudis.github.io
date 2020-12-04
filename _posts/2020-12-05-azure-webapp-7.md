---
layout: post
title: "Public Cloud 전환 프로젝트 : 7부 영업사원이 말하지 않는 Cloud 서비스의 불편한 진실 : Storage 서비스의 IOPS 와 Bandwidth 과금"
date: 2020-12-05 01:40:13
img: azure-webapp-404.png # Add image post (optional)
categories: cloud azure
tags: cloud azure MS
---
[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2Fgraudis%2Fgraudis.github.io%2Fblob%2Fmaster%2F_posts%2F2020-12-05-azure-webapp-7.md&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)


​	**필자연역 :** Linux 을 15년 이상 해왔으며, Linux 기반의 Enterprise 인프라 구축 엔지니어 생활을 8년 동안 하였으며, OpenSource 구축 및 기술지원 분야에서 다양한 경험을 하였다.

​	지금까지의 필자의 경력은 주로 On-Premis Enterprise 기업환경의 인프라 구축 및 기술 지원등의 TA 역활을 다년간 진행하였기에 Public Cloud 전환 프로젝트는 또다른 경험을 하는 기회였다.

필자의 주분야는 OpenSource WEB/WAS 분야 이며, 설계/구축/장애 기술지원에 다양한 경험을 가지고 있다.

* **프리랜서 TA 로 프로젝트를 진행 하다보니 이제서야 연재를 작성할 시간적 여유가 생겨 늦게 나마 이야기를 이어 나가고자 한다.**



#### 7. 영업사원이 말하지 않는 Cloud 서비스의 불편한 진실 ( Storage 서비스의 IOPS 와 Bandwidth 과금 )"

  **" Custom Docker Image Build 작업 이후 누락된 User Contents Data 전수 조사"** 를 하고 있을 때의 시점의 이야기 이다.



  " 6부 "에서의 User Contents Data 누락으로 인하여 "950만건의 File 전수조사" 를 진행을 위해서 이미 BLOB Storage 로 이관된 Data 들의 실제 파일명과 카운트등의 데이터가 필요한 상황 이였다. 



여기서 필자의 뒷통수를 쎄하게 하는 느낌이 Cloud 서비스 영업사원 및 Cloud 벤더사에서  고객에게 알려주지 않는 **" Storage 서비스의 IOPS 와 Out Bound Bandwidth 트래픽 과금 "** 이 문제 였다.



1. **불편한 진실 : Cloud Blob Storage 는 IOPS 를 보장하지 않는다.**

   **Cloud 서비스에서의 Blob Storage 는 AS-IS On-premise 에서 사용하던 고비용의 Storage 장비에 대비 초기 구축비용 없이 사용한 용량만 큼만 비용을 지불하기에 꽤 매력적인 Storage** 로 영업에서는 고객에게 어필 하고 있다.

   

   **다음은 벤더사의 " BLOB Storage " 서비스 계산표 이다.**

   ![blob1](https://github.com/graudis/graudis.github.io/blob/master/_image/azure-blob-1.png?raw=true)

   ![blob2](https://github.com/graudis/graudis.github.io/blob/master/_image/azure-blob-2.png?raw=true)

   ![blob3](https://github.com/graudis/graudis.github.io/blob/master/_image/azure-blob-3.png?raw=true)

   위의 **30TB 의 Data 를 저장하는대 월 70만원 선으로 매우 비용이 저렴**하게 계산된다.

   허나 여기서 불편한 진실이 숨어 있다.

   

   우선 **Blob Storage 원래 용도** 부터 알아 보자.

   **저비용으로 대용량의 Data 들을 백업/보존 하기 위하여 아카이빙( Archiving ) 목적 로 사용하기 위한 Storage 서비스가 바로 Blob Storage 서비스** 이다.

   

   **[ 예고된 사고 : 영업사원 비용절감으로 고객에게 어필 ]**

   . **영업사원의 저비용의 Storage 사용으로 비용절감 이점 어필**

   영업사원은 고객에게 Blob Storage 의 원래 용도를 설명하지 않고 **단지 저비용의 Storage 서비스로 기존 고비용의 AS-IS Storage 서비스를 대치 하면 비용이 절감된다고만 설명**한다.

   . 이러한 영업사원의 단편적인 설명으로 인하여 고객은 Blob Storage 이  **AS-IS 고비용의 Storage 에서의 동일한 File Read/Write 속도를 보장하면서도 비용이 매우 저렴한 것처럼 착각** 하게 된다. 

  
  

   **[ 예고된 사고 : Blob Storage 는 Mount 할 수 없다. ]**

   . 영업사원의 Blob Storage 기**초 지식 무지로 인하여 Blob Storage 로 Data 이관이 AS-IS Storage 의 File System 과 같은 개념으로 Server 에 폴더나 Mount 경로로 Mount 하여 File Copy 작업으로 쉽게 진행 되는 것처럼 고객에게 설명**한다.

   . 이는 영업사원 뿐만 아니라 **필자가 수행했던 프로젝트의 Cloud 전문가 라는 엔지니어 입에서도 영업사원과 동일한 말이 나와 필자를 경악** 하게 만들었다. 

   

   . 결론부터 말하자면, **Blob Storage 는 일반적인 File System  구조가 아니므로 직접적으로 Mount 해서 사용은 불가능** 하다. 

   

   **다음은 Blob Storage 에 대한 간단한 설명이다.**

   

   ## Blob Storage 정보

   Blob Storage는 다음을 위해 설계되었습니다.
   
   - 브라우저에 이미지 또는 문서 직접 제공
   - 분산 액세스용 파일 저장
   - 동영상 및 오디오 스트리밍
   - 로그 파일에 쓰기
- 백업/복원, 재해 복구 및 보관용 데이터 저장
   - 온-프레미스 또는 Azure 호스티드 서비스에 의한 분석용 데이터 저장

   사용자 또는 클라이언트 애플리케이션은 전 세계 어디서든 HTTP/HTTPS를 통해 Blob Storage의 개체에 액세스할 수 있습니다. Blob Storage의 개체는 [Azure Storage REST API](https://docs.microsoft.com/ko-kr/rest/api/storageservices/blob-service-rest-api), [Azure PowerShell](https://docs.microsoft.com/ko-kr/powershell/module/az.storage), [Azure CLI](https://docs.microsoft.com/ko-kr/cli/azure/storage) 또는 Azure Storage 클라이언트 라이브러리를 통해 액세스할 수 있습니다. 클라이언트 라이브러리는 다음을 포함하여 다양한 언어에서 사용할 수 있습니다.
   
   - [.NET](https://docs.microsoft.com/ko-kr/dotnet/api/overview/azure/storage?view=azure-dotnet)
   - [Java](https://docs.microsoft.com/ko-kr/java/api/overview/azure/storage)
   - [Node.JS](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage)
   - [Python](https://docs.microsoft.com/ko-kr/azure/storage/blobs/storage-quickstart-blobs-python)
   - [Go](https://github.com/azure/azure-storage-blob-go/)
   - [PHP](https://azure.github.io/azure-storage-php/)
   - [Ruby](https://azure.github.io/azure-storage-ruby)

   

   **[ 예고된 사고 : Blob Storage 는 IOPS 를 보장하지 않는다. ]**

   . **Blob Storage 는  AS-IS On-premise Storage IOPS 와 같은 속도를 절대로 보장하지 않는다!**

   

   **그 어떠한 Cloud 벤더사에서도 Blob Storage 에서의 IOPS 에 관한 내용은 설명하지 않으며, 어떠한 보장도 하지 않는다.**

   

   **Blob Storage 에 대한 이해도가 있는 분들이라면, 왜 IOPS 에 관하여 설명하지 않고 어떠한 보장도 하지 않는 이유를 잘알고 있을 것이라 생각하여 자세한 설명은 생략** 한다. 

   

   기존 AS-IS Storage 에서 **Business Logic 에 의해 빈번하게 I/O 가 발생 하는 용도 였다면, Blob Storage 는 IOPS 를 절대로 보장하지 않기에 Application 에서 Error 발생** 하기 시작하면서 100% 프로젝트 사고가 나게 된다.

   

   Blob Storage 에 무지한 영업사원의 말로 인하여 위의 예고된 3가지 대형 사고가 예정되고 프로젝트가 시작 하게 된다. 

   


2. **불편한 진실 : Cloud Azure Files(SMB) 는 IOPS 를 보장도 하지 않으면서 비싸기 까지 하다.**

   영업사원 말에 의해서 Blob Storage 로 이관하여 장애 발생후 초기 Storage 선택이 잘못된걸 인지한 이후에 고객에게 엄청난 항의를 받고서 선택 한 Storage 서비스가 바로 Azure Files(SMB) Storage 서비스 입니다.
   
   **Azure Files(SMB) 서비스도 Blob Storage 와 같은 백업/보존 하기 위하여 아카이빙( Archiving ) 목적 로 사용하기를 권장** 합니다. 

   

   **다음은 Azure Files(SMB) 에 관한 비용 계산표 입니다.**

   

   ![azure-file1](https://github.com/graudis/graudis.github.io/blob/master/_image/azure-files-1.png?raw=true)

   ![azure-file2](https://github.com/graudis/graudis.github.io/blob/master/_image/azure-files-2.png?raw=true)



   같은 30TB 용량을 기준으로 계산결과 Blob Storage 비용대비 4배 이상의 차이를 보입니다.

   하지만, Blob Storage 에서는 해결되지 않는 문제점들을 해결 할 수 있다는 희망으로 

   마치, Azure Files(SMB) 로 선택하면 모두 해결될 것처럼 고객을 또한번 설득 하게 되는데, 

   여기에도 함정이 숨어 있습니다.

   

   다음은 Azure Files(SMB) 광고글 입니다.

   

   ## 주요 이점

   - **공유 액세스** Azure 파일 공유는 산업 표준 SMB 및 NFS 프로토콜을 지원합니다. 즉, 애플리케이션 호환성에 대한 걱정 없이 온-프레미스 파일 공유를 Azure 파일 공유로 원활하게 바꿀 수 있습니다. 여러 머신, 애플리케이션/인스턴스 간에 파일 시스템을 공유할 수 있다는 것은 공유성이 필요한 애플리케이션에 Azure Files를 사용하는 중요한 이점입니다.
   - **완벽한 관리** - Azure 파일 공유는 하드웨어 또는 OS를 관리할 필요 없이 만들 수 있습니다. 즉 서버 OS를 중요한 보안 업그레이드로 패치하거나 결함이 있는 하드 디스크를 교체하지 않아도 된다는 것입니다.
   - **스크립팅 및 도구 지원** - PowerShell cmdlet 및 Azure CLI를 사용하여 Azure 애플리케이션 관리의 일부로 Azure File 공유를 만들고, 탑재하고, 관리할 수 있습니다. Azure Portal 및 Azure Storage Explorer를 사용하여 Azure File 공유를 만들고 관리할 수 있습니다.
   - **복원력**. Azure Files는 처음부터 항상 사용할 수 있도록 빌드되었습니다. 온-프레미스 파일 공유를 Azure Files로 바꾸는 경우 로컬 정전 또는 네트워크 문제를 처리하기 위해 더 이상 주의할 필요가 없습니다.
   - **친숙한 프로그래밍** - Azure에서 실행 중인 애플리케이션은 [파일 시스템 I/O API](https://docs.microsoft.com/ko-kr/dotnet/api/system.io.file)를 통해 공유 데이터에 액세스할 수 있습니다. 따라서 개발자는 기존의 코드와 기술을 이용하여 기존 애플리케이션을 마이그레이션할 수 있습니다. 시스템 IO API 외에도 [Azure Storage 클라이언트 라이브러리](https://docs.microsoft.com/ko-kr/previous-versions/azure/dn261237(v=azure.100)) 또는 [Azure Storage REST API](https://docs.microsoft.com/ko-kr/rest/api/storageservices/file-service-rest-api)를 사용할 수 있습니다.

   

   **[ 팩트 체크 ]**

   **. 공유억세스 : NFS 지원 안함 CIFS 만 지원함**

   **. 복원력 : Azure Files(SMB) Storage 와 Network 장애 발생 일어남 Mount 끊어짐**

   **. 비용 : Blob Storage 대비 4배 이상의 고비용**

   **. IOPS : 그 어디에도 Azure Files(SMB) Storage Volume 에 관한 IOPS 에 관한 부분이 명시되거나 언급이 없음** 


   

   **[ 예고된 사고 : Azure Files(SMB) 는 IOPS 를 보장하지 않는다. ]**

   . **Azure Files(SMB) 또한 Blob Storage 와 같이 그 어떠한 기술문서에도 IOPS 관한 부분을 보장하거나 언급 하지 않습니다.**

   

   따라서, **Azure Files(SMB) 또한 IOPS 가 어느정도 나오는지 알길이 없어 I/O 가 많이 발생하는 Application 에서의 Data Storage 로는 사용하기에는 다소 문제가 발생** 할 수 있습니다.

   

   **필자의 경험상 Built-in 된 Azure WebApp 서비스에 제공하는 Azure Files(SMB) 1TB User Data 용 Volume 이 IOPS 가 매우 느리고 불규칙하여 서비스 사용이 불가한 경험**을 가지고 있다. 

   

   **Azure Files(SMB) 는 Network 로 제공되는 Volume Type 이라, IOPS 자체를 따지는 것이 무리 일수도 있다.** 

   

   그렇다면, **I/O 가 빈번하게 발생하는 Image Server 같은 용도에서는 Azure Files(SMB) Storage 도 결국 올바른 선택이 아니게 된다.**

   

   Cloud 환경에서의 Azure Files(SMB) 와 같은 Network File System 들에서 IOPS 및 최소 IOPS 에 관한 언급이 없는 이유는 특별히 설명하지 않아도 Cloud Storage Farm 를 이해 하는 엔지니어라면 알 수 있을 꺼라 생각하여 설명을 생략 한다.

   

   

3. **불편한 진실 : 들어올때는 마음대로 나갈때는 과금청구!**

   영업사원 말중에 고객의 심금을 울리는 말이 바로 " **내부 통신은 무료 입니다.** " 라는 말이다.

   틀린 말은 아니다. 

   맞다. C**loud 내의 Server to Server 와 Server to Storage 간의 네트워크 비용은 거의 무료이거나 아주 저렴한 비용만 청구**된다. 


   

   여기서, 함정카드 발동~! 

   그럼 **인터넷을 통하여 외부 사용자의 요청으로 인한 Data Out Bounding 할때의 사용하는 서비스는 그리고 그 비용은 ? 도데체 얼마 일까 ?**

   

   **정확한 서비스 명칭은 Network " Bandwidth " 서비스** 이다. 

   

   이 서비스의 목적은 **Cloud 서비스를 받고 있는 Data 센터와 해외 Data 센터 혹은 외부 인터넷 망으로 Data Out Bounding 할때 부과되는 비용**을 말한다. 

   

   **기본 5GB 까지는 무료로 과금하지 않는다.** 

   

   **다음은 벤더사의 Network " Bandwidth " 서버의 가격 계산이다.**



   ![bandwidth](https://github.com/graudis/graudis.github.io/blob/master/_image/azure-bandwidth-1.png?raw=true)

   

   30TB 의 고객 User Data 를 타 Cloud 벤더로 이관 할때를 가정하여 " Bandwidth 서비스 "  비용을 계산하였다.

   금액이 무시무시 하다.

   

   **주의 : 위의 5GB 용량은 1개월 누적 Data 전송 용량을 말한다.** 

   이말은 즉, Storage Account 를 생성하고 1개월 안에 외부로 전송되는 Data 의 총합이 5GB 를 넘지 않으면 이용 요금이 무료이며 과금을 하지 않는다는 뜻이다.

   

   **대부분의 상업 서비스들은 1개월 동안 외부로 Data 전송하는 Data 가 5GB 를 훌쩍 넘기게 된다.**

   

   바로 네트워크 Bandwidth 서비스 요금을 별도로 부과하겠다는 뜻이다.

   

   

4. **필자의 정리**

   . **Cloud 벤더사의 효자 종목( 필자 뇌피셜 )**

   **" Blob Storage / Azure Filefs(SMB) / Bandwidth 서비스 " 이렇게 3종목이 각각의 Cloud 벤더사의 이름만 틀릴뿐 효자 종목으로 생각한다.** 

   이말은 **위의 3가지 서비스가 수익 창출의 모델이라고 필자는 생각 한다.**

   **IOPS 를 구지 신경 안쓰고도 고객의 Data 가 있는한 매달 Storage 사용요금과 Network Bandwidth 이용 요금이 부과** 되기 때문이다. 

   Storage 공간을 매달 임대하여 쓰기에 당연히 매달 비용을 낸다고 생각하면 얼추 이해가 되지만, Bandwidth 서비스 과금은 좀 과하지 않나 싶은 생각이 든다.


   

   . **타 Cloud 벤더로 이관할때도 Data Out bounding 으로 인한 요금 부과 발생**

   : **타 Cloud 벤더로 옮길때도 많은 비용을 이관하는 쪽이 아닌 Out Bounding 발생하는 떠나는 벤더사에게 지불**하고 이관해야 한다. **이관 계획을 신중히 생각** 해야 한다.




   . **I/O 가 빈번하게 발생하는 Business Logic 에서는 Blob / Azure File 는 사용을 필자는 권장하지 않는다**

   : I/O 가 빈번하게 발생하는 E-commerce 의 Image 서버 의 Storage 용도 및 동영상 Encoding / Decoding 이 직접 진행되는 Storage Volume 용으로는 사용하지 않는 것을 필자는 권장한다.




   . **필자의 뇌피셜( 흑우가 되지 말자! )**

   위에서 언급된 내용이 사실 필자가 Cloud 프로젝트를 수행하면 직접 몸을 부딛쳐 체험한 불편한 내용들이다.  

   

   지금이야 편히 말할 수 있지만, **프로젝트 수행의 책임을 지는 PM 일때는 무지한 영업사원의 말 한마디에 많이 답답하고 암담한 느낌으로 어떻게 하든 해결책을 모색하며 스트레스를 엄청 받았었다.**

   

   최소한 **본인이 판매하는 또는 고객에게 판매하고자 하는 상품에 대해서 학습**을 했으면 한다. **Cloud 마이그레이션이 편의점에서 단품상품에 바코드 찍고 금액을 청구하는 그런 편의점 알바생 업무 수준의 일이 아니라는 개념**을 가졌으면 한다.

   

   ===== 8부 에서 계속 될까 ? =====
