---
layout: post
title:  "Public Cloud 전환 프로젝트 : 4부 Build 된 Custom Docker Image 적용 이후 들어나기 시작한 문제점들 (1)."
date:   2020-09-21 22:59:13
categories: cloud azure
tags: cloud azure MS
---

​	**필자연역 :** Linux 을 15년 이상 해왔으며, Linux 기반의 Enterprise 인프라 구축 엔지니어 생활을 8년 동안 하였으며, OpenSource 구축 및 기술지원 분야에서 다양한 경험을 하였다.

​	지금까지의 필자의 경력은 주로 On-Premise Enterprise 기업환경의 인프라 구축 및 기술 지원등의 TA 역활을 다년간 진행하였기에 Public Cloud 전환 프로젝트는 또다른 경험을 하는 기회였다.

필자의 주분야는 OpenSource WEB/WAS 분야 이며, 설계/구축/장애 기술지원에 다양한 경험을 가지고 있다.

* **프리랜서 TA 로 프로젝트를 진행 하다보니 이제서야 연재를 작성할 시간적 여유가 생겨 늦게 나마 이야기를 이어 나가고자 한다.**



#### 4. Build 된 Custom Docker Image 적용 이후 들어나기 시작한 문제점들 (1).

  **" 3부 WebApp(PaaS)를 위한 Custom Docker Image Build 삽질. "** 편 에서 build 한 Custom Docker Image 를 WebApp(PaaS) 에 적용하였다.



  Custom Docker Image 로 실행되는 WebApp 는 정상적으로 실행 되는 듯 보였다. 그러나, 그것도 잠시 숨겨 졌던 문제점들이 하나둘씩 나타나기 시작하였다. 

1. **Instance 마다 다른 Main 게시물**

   : Multi Instance 로 구성되어 있는 WebApp 에 각각 다른 사용자가 접속 하였을 경우 공통적인 Main 게시물의 내용이 동일해야 하는데 다른 결과물이 나오기 시작 했다. 

   

   **[ 원인 ]**

   Monolithic Architecture Web Source 에서 Main Page 에 표현되는 게시물들의 빠른 Loading 을 위하여 미리 Main Page 의 Contents 들을 임시 HTML 파일로 만들어서 Main Page 를 Loading 시키는 방법을 사용한 다는 것을 알게 되었다. 

   따라서, Web Source 를 개별적 Instance Local 영역에 Deploy 하게 되어 각각의 Main Page 의 내용이 다르게 표현 되는 현상이 발생 한것이다. 

   

   **[ 해결 방법 ]**

   Multi Instance 구조의 WebApp 에서 공동으로 사용 가능한 Storage Volume 를 Mount 하고 Mount 된 Shared Volume 에 Main Page Index HTML 파일을 저장 하여 바라 보게 한다면 문제가 해결 될 것이라는 생각에 생각대로 진행 하였다.

   

   **[ 결과 ]**

   생각한대로 Shared Volume 에 Main Page Index HTML 파일을 저장하여 공통으로 사용하도록 하니 어떠한 Instance 로 접속을 하여 동일한 내용이 나오게 되었다.

   

   **[ 악연의 시작 1 ]**
   
   Azure WebApp 에서의 문제는 Shared Volume 을 NFS 혹은 SMB 어떠한 것으로 구성 할 것이냐가 문제 였다. 
   필자는 NFS 로 연동하는 것이 안정성 면에서도 그나마 괜찮을 꺼라 생각하여, Azure 서비스에서 찾기 시작 하였으나, 
   Azure 에서는 NFS 서비스가 존재하지 않았다.

   Azure File(SMB) 만 존재 하였다. 비용 또한 Azure BLOB Storage 대비 어마 어마하게 비싼 비용이였다.  
   선택의 여지가 없었다. SMB....훔....  쎄한 느낌이 들었다. 

   ( 이때 이미 SMB 를 버렸어야 했다. SMB 가 두고 두고 필자를 괴롭힐 줄은 이때는 전혀 알수가 없었다.)



2. **ARR 의 저주**

   : WebApp Instance 는 정상이나 WEB URL 접속시 " service not available " 메시지와 함께 서비스 장애 발생

   

   **[ 원인 ]**

   WebApp(PaaS) 는 다이렉트로 Web Requst 를 수신 하는 구조라고 흔히들 알고 있을 것이다. 하지만, 정확히 말하면 WebApp(PaaS) 앞에 숨겨진 Web Server 역활을 하는 Application 이 존재 한다.

   Azure 에는 " ARR : Application Request Routing " 이 WebApp(PaaS) 앞에 숨겨져 있고 Web Server 역활을 하고 있었던 것이다. 

   WebApp 의 Multi Instance 사용시 Multi Instance 중 하나의 Instance 가 재시작면, ARR 과 맺어 있었던 Session 값이 변경되어 새롭게 생성된 Session 값으로 갱신되어야 하는데 이때,

   ARR 이 새로 갱신된 Instance 의 Session 값을 인지 하지 못하고 이미 종료된 Instance 의 Session 값을 유지 하게 되어 발생되는 문제 였다.

   

   **[ 해결 방법 ]**

   ARR 의 구조적인 문제를 개선 할 방법이 없기에 다음과 같은 내용을 벤더사에 문의 하였다.

   . ARR 기능만을 Off 시킬 수 있는가 ? : 불가능 하다.

   . ARR 만 따로 분리 시킬수 있는가 ? : 불가능 하다. 

   결론은 ARR 이 문제를 일으키면 답이 없다는 뜻이다. 

   ARR이 새로 갱신된 Instance 의 Session 을 인식하기 전까지 기다리는 방법 이외에는 

   해결 방법이 없다는 뜻이다.

   ( 여기서 언제 서비스 장애가 발생할지 모르는 불안한 날들의 시작을 예고 하고 있었다. )

   

   **[ 악연의 시작 2 ]**

   처음 " service not available " 서비스 장애 발생시 벤더에서는 벤더에서 제공하는 WebApp(PaaS) 를 사용하지 않고 " Custom Docker Image "의 결함으로 몰고 가려고 하였다.

   필자는 워낙 이런경우에 대하여 경험이 있기에 다음과 같은 내용들의 자료를 수집하였다.

   . " Scouter " Log 및 내부 WebApp Instance 의 정상적인 상태 캡쳐

   . WebApp Instance 내의 Shell 모드에서 " Curl " 명령어를 이용하여 외부 Web Site 통신 내용을 캡처

   . WebApp Instance 내에 Tomcat Log 에서의 외부 인입 Request 발생시 기록되는 Access Log 관련 부분 캡쳐

   위의 내용들을 모두 증거 자료로 캡쳐 하여 벤더 엔지니어 에게 보냈다. 하지만,

   여기서, 예상치 못한 문제가 필자를 멘붕에 빠트렸다. 정말 제대로 멘붕이 왔다.

   필자가 제시한 증거 자료를 보고 나서 벤더 엔지니어의 반응이 " 이게 뭔가요? " 라는 반응 이였다. 뭐라 할말을 잃게 만들었다.

   

   여기서 잠깐, ARR 에 대해서 잠시 알아 보았다.

   **[ Azure ARR 구성도 ]**

   ![ARR](https://github.com/graudis/graudis.github.io/blob/master/_image/ARR_architecture.jpg?raw=true)

   ARR( Application Request Routing ) : MS 의 IIS Server 내에서의 Web Request " Session 처리 " 를 담당하고 있는 기능으로 나온다. 아놔... MS.... IIS 의 향기가..

   MS 이기에 ARR 를 끌수도 뺄수도 없는 구조 인것이 이해가 되었다.

   

  처음부터, 뭔가가 잘못 되었다는 느낌이 강하게 들기 시작하였다. 접근할 수 있는 인프라 영역에서의 장애라면 어떻게 하든 해결해 보겠으나, 접근 할 수 없는 영역의 인프라에서의 장애 발생은 벤더가 해결 해주기 전까지는 손 놓고 있어야 하는 상황이 필자는 너무나 짜증이 나기 시작 했다.

ARR 은 이때 부터 모든 고생을 하고도 고객사에게서 가장 많은 컴플레인을 받게 되어 필자를 끝까지 힘들게 하는 장애 원인으로 자리 잡게 된다.



3. **WebApp Instance 의 무한 재시작 현상을 일으키는 WebApp(PaaS)  Scale Out 의 비밀**

   : MSA 구조가 아닌 Monolithic Architecture Web Source 에서는 다중의 독립적인 WebApp Instance 구성 보다는 단일 WebApp Instance에 " Scale Out " 를 하여 복제 성격의 다중 Instance을 구성 하여야 하는것이 고객의 요구 사항 이였다.

   

   **[ 원인 ]**

   . 단일 Domain 의 " Scale Out " 으로 복제된 Multi Instance 중 일부 Instance 의 " OOM " 현상으로 무한 재시작 현상 발생 원인은 의외로 간단했다. 

   WebApp(PaaS) 의 " App Service Plan " VM 머신의 Resource 부족으로 마지막으로 실행된 Web Instance 의 실행 조건에 맞는 Resource 가 부족하여 발생 된 원인이였다. 

   

   **[ 해결 방법 ]**

   . " App Service Plan " 를 " Scale Up " 하여 문제를 해결

   초기 설정된 " App Serivce Plan " 의 스펙을 프레미엄급의 최상으로 선택하여 Core 및 메모리를 확보하니 무한 재시작 현상이 사라 졌다.

   

   **[ Azure WebApp App Service Plan ]**

   ![SKU](https://github.com/graudis/graudis.github.io/blob/master/_image/app_serice_pan_1.png?raw=true)

   

   **[ 악연의 시작 3 ]**

   MSA 구조에서는 " App Service Plan " 을 높이지 않고 다중으로 내부 서비스별 독립적인 WebApp Instance 로 구성하는 것이 일반적이나, 필자가 PM 및 인프라 엔지니어로 수행한 프로젝트는 " Monolithic Architecture Web Source " 를 Public Cloud WebApp(PaaS) 로 " Migration " 하는 프로젝트 이므로 처음부터 제안 자체가 잘못된 프로젝트라고 개인적 생각이 점차 들기 시작 하였다.
   

   " App Service Plan " 를 " Scale Up " 하여 성능을 높여 WebApp Instance 의 무한 재시작 장애는 해결되었으나, 비용이 2배로 증가하는 부작용이 발생하였다.
   
   AS-IS On-Premise 인프라의 운영/유지 TCO 대비 Public Cloud 전환으로 비용을 절감 할 수 있다는 벤더사의 영업담당의 말이 점점 그 의미를 잃어버리게 되었다.
   
   이러한 불편한 진실이 현실로 나타나기 시작하니 고객사 에서는 " 설마 벤더사 고급 엔지니어의 설계/제안 내용에 문제가 있겠어 ? " 라는 반응으로 비용 증가에 대한 원인을 제안한 벤더사 가 아닌 엉뚱한 프로젝트 수행사에게 불만이 집중 포화 되기 시작 하였다. 

   ( 아놔..... 여긴 누구 나는 어디 ? .... )


   **[ 드러나는 최강의 적 ]**
  
  이러한 상황에서 머리를 스치고 지나가는 의문점이 있었다. 
  
  " 왜 난 PM 롤인데 인프라 문제를 해결 하고 있지? 자사 인프라 담당 엔지니어는 왜 놀고 있지 ? Azure 경력이 많은 전문가 엔지니어라 했는데 ? " 
  
  알고보니 On-Premise Enterprise 분야의 경험치도 없고 사전지식도 없어서 기초적인 설정에도 처리 하지 못하고 손놓고 전문 업체를 섭외 해달라는
  **" 우주최강 업.체.맨! "** 이 바로 자사 인프라 엔지니어 였던 것이 였다. ( 와우! Best ! 정줄 가출 일보 직전 )
  
  업체맨에게 **" 엔지니어가 엔지니어를 호출 하는 엔지니어 재귀호출 신공을 쓰는 도대체 누구냐 넌?! "** 라고 묻고 싶었다.
  
  " 전문가 " 를 자칭하는 엔지니어에 스킬을 신뢰할 수 없다는 선입견이 잘못된것이 아니라 펙트 였구나 라는 생각에 
  " 아.. 제길... " " 그래... 너의 일은 내가 다 할테니 제발 프로젝트 방해만 하지 말고 아무것도 하지 말고 놀아라 그냥!... " 라는 심정 뿐이였다. 
  
  일할줄 아는 사람은 야근에 시달리고 일할줄 모르는 사람은 모른다는 것에 파워풀 하게 당당해 하며, 정시 퇴근과 팀 회식날짜 만을 기다리는 어처구니 없는 상황이 연출되기 시작 하였다.
  
  " 그래 어차피 내가 다 할줄 아니 그냥 내가 하자. " 라는 결심으로 프로젝트 종료까지 퇴근 후 매일 밤 새벽 삽질이 시작 되었다.

( 더이상 나빠저도 이젠 무덤덤 어차피 다... 내꺼다.. 라고 생각 하니 ... 오늘도 생존을 위한 새벽 생존삽질.... )

 

== 5부 에서 계속 ==
