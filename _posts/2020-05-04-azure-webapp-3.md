---
layout: post
title: "Public Cloud 전환 프로젝트 : 3부 WebApp(PaaS)를 위한 Custom Docker Image Build 삽질."
date: 2020-05-04 12:59:13
img: azure-webapp-404.png # Add image post (optional)
categories: cloud azure
tags: cloud azure MS
---

​	**필자연역 :** Linux 을 15년 이상 해왔으며, Linux 기반의 Enterprise 인프라 구축 엔지니어 생활을 8년 동안 하였으며, OpenSource 구축 및 기술지원 분야에서 다양한 경험을 하였다.

​	지금까지의 필자의 경력은 주로 On-Premise Enterprise 기업환경의 인프라 구축 및 기술 지원등의 TA 역활을 다년간 진행하였기에 Public Cloud 전환 프로젝트는 또다른 경험을 하는 기회였다.

필자의 주분야는 OpenSource WEB/WAS 분야 이며, 설계/구축/장애 기술지원에 다양한 경험을 가지고 있다.



#### 3. Public Cloud 전환 프로젝트 : 3부 WebApp(PaaS) 를 위한 Custom Docker Image Build 삽질.

  **" 2부 WebApp(PaaS) 삽질하다. "** 편 에서 테스트 해본 결과 Built-In 으로 제공되는 MS Azure Linux WebApp( Alpine Linux/PaaS ) 으로는 기존의 Monolithic Architecture Web Source 로는 정상 적인 Web Site 서비스가 매우 어렵다는 결론을 얻게 되었다.



  물론, 여기서 무조건 Monolithic Architecture Web Source 의 서비스는 구축이 불가능 하다는 뜻은 아니다. Monolithic Architecture Web Source 라고 해도 단순기능 회사소개 및 제품 소개등의 정적인 컨텐츠를 제공 하는 서비스 라면 Built-In 된 WebApp 서비스 를 이용하여 Web Site 를 구성 하는 것에는 큰 문제가 없다고 판단 된다.



  다만, 필자가 수행 하였던 " Public Cloud 전환 " 프로젝트는 불행히도 사이트 접속 사용자의 요청에 따라 동적 컨텐츠 서비스를 제공하는 특징을 가진 Web Site  서비스 이라 문제가 되었다.



​    **1. MS ACR( Azure Container Registry** )

  필자는 Built-In WebApp 를 사용하지 않고 Monolithic Architecture Web Source 에 부합 하는 Custom Docker Image 를 제작 하기로 고객과 최종 협의 하였다.  고객 또한 Public 한 환경을 고려하여 구성된 Built-In WebApp 으로는 서비스 구성이 안될 것을 충분히 인지 하였다.



  다음은 MS Azure 에서의 Custom Docker Image 사용을 위한 구성도 이다. 

![ACR1](https://github.com/graudis/graudis.github.io/blob/master/_image/webapp-docker-5.png?raw=true)

  또는 

  ![ACR2](https://github.com/graudis/graudis.github.io/blob/master/_image/webapp-docker-4.png?raw=true)



  MS Azure 에는 ACR( Azure Container Registry) 라는 Docker Image 를 저장 하는 저장소 서비스가 위와 같이 존재 하며, 용도는 Docker Image 를 배포 하기 위한 저장소 로 사용한다.



2. **Custom Docker Image Build 하기**

  필자는 Custom Docker Image 를 Build 하기 위해서 Azure Linux WebApp Docker Build 관련 파일들을 Azure 공식 커뮤니티 사이트에서 받아 구성 하였다.

![Azure WebApp](https://github.com/graudis/graudis.github.io/blob/master/_image/webapp-docker-10.png?raw=true)

  위에서 제공 되는 Azure Linux WebApp Docker 관련 파일들의 Base OS 는 모두 Alpine Linux 로 구성되어 있다고 보면 된다.



3. **Alpine Linux 를 버리고 Debian 9 로 갈아 타기**

  필자는 일단 Base OS 가 Alpine Linux 로 구성되어 있는 Docker Build 파일을 받아서 System 안정성이나 Network 안정성에 검증이된 " Debian 9 " 로 Custom Docker Image 를 Build 하기 위하여 다음과 같이 진행 하였다. 

![Azure WebApp](https://github.com/graudis/graudis.github.io/blob/master/_image/webapp-docker-11.png?raw=true)



  GitHub Master ZIP 파일을 받아서 Docker Build 가 가능한 Linux 가상 머신에 다운로드 하여 기본적인 환경을 모두 구성 하였다.

![Azure WebApp](https://github.com/graudis/graudis.github.io/blob/master/_image/webapp-docker-12.png?raw=true)



![Azure WebApp](https://github.com/graudis/graudis.github.io/blob/master/_image/webapp-docker-13.png?raw=true)



  위에서 필자는 " Dockerfile " 파일을 열어 다음과 같이 수정 하였다.



![Azure WebApp](https://github.com/graudis/graudis.github.io/blob/master/_image/webapp-docker-14.png?raw=true)



  Base_Image 는 Base OS 가 되는 Linux OS 의 Image 파일을 정의 하는데 필자는 " Debian 9 " 를 사용하는데 여기서 사용하는 Base OS 의 " Repository " 주소는 MS Azure 가 제공하는 URL 를 사용 하였다. 



![Azure WebApp](https://github.com/graudis/graudis.github.io/blob/master/_image/webapp-docker-15.png?raw=true)



  위의 화면은  " Dockerfile " 이 수정이 완료 된 후 Dockerfile 을 Build 하는 과정이다.



![Azure WebApp](https://github.com/graudis/graudis.github.io/blob/master/_image/webapp-docker-16.png?raw=true)



  필자는 " Dockerfile " 을 Build 하는 과정에서 재미 있는 점을 발견 하였다. MS Azure WebApp 에서는 기본적으로 WebApp Container 의 상태를 모니터링 하는 서비스인 " Application Insights " 를 기본적으로 탑제 하게 되는데, 여기서 MS Azure 가 사용하는 JAVA Base 의 Linux WebApp 에  사용되는 " Application Insights " 의 JAVA Library 를 경쟁사 인 " AWS " 것을 에서 받아 쓴다는 것이다. 이미지 화면을 확대 하여 보면 정확히 알수 있을 것이다.

![azure](https://github.com/graudis/graudis.github.io/blob/master/_image/webapp-docker-18.png?raw=true)

  

짜증의 연속인 삽질 작업에 순간이나마 웃게 만든 사건 이였다. 개인적으로는 MS Azure 서비스가 알면 알수록  짜증이 새록 새록 올라오는 상황에서 좀 웃으면서 삽질 하라는 뜻인것 같다.  순간 어의가 없어서 " 피식 .." 했다.



  다음은, MS Azure Portal 에서의 Custom Docker Image 를 이용하여 WebApp 를 생성하는 과정 이다. 

4. **MS Azure ACR( Custom Docker Image 적용 )**

![Azure WebApp](https://github.com/graudis/graudis.github.io/blob/master/_image/webapp-docker-1.png?raw=true)



![WebSSH](https://github.com/graudis/graudis.github.io/blob/master/_image/webapp-docker-6.png?raw=true)



![WebSSH](https://github.com/graudis/graudis.github.io/blob/master/_image/webapp-docker-7.png?raw=true)

[ Azure Container Registry 메뉴]

![WebSSH](https://github.com/graudis/graudis.github.io/blob/master/_image/webapp-docker-8.png?raw=true)



5. **Custom Docker Image( Debian 9 ) 에 Monolithic Architecture Web Source 테스트**

​	필자가  MS Azure Application Insights 가 적용된 Custom Docker Image 를 적용한 Linux WebApp 에  Monolithic Architecture Web Source 를 테스트 하였다.

​    다음은 고객의 Monolithic Architecture Web Source 테스트 전 마이그레이션 요구 사항들이다.

​	. AS-IS 의 Monolithic Architecture Web Source 구조를 유지 할 것 ( 최소 변경 )

​	: DevOps 구성 및 반영을 위한 최소의 구조 변경 / Contents 경로및 Blob Storage 사용을 위한 구조변경



​	. 1개 WebApp Container Instance 에서 전체 Web Source 가 실행 될 것 ( MSA 구조가 아님 )

​	: WebApp Container Instance 1개로 기동 확인됨



​	. Intro Main Page 의 Page Loading Time 이 1 Sec 전후 일 것 ( AS-IS 와 같거나 빠른 응답속도 )

​	: Custom Docker 적용 후 WebApp User 에게 제공되던 1TB 의 Shared Volume( " /home ") 의  가려 져있떤 IOPS 문제가 밝혀짐, 불규칙 한 IOPS 값으로 인하여 " Intro Main Page "의 Loading 속도가 700ms 에서 4 Sec 로 불규칙하게 반응함

​	. 내부 기능들( 파일 전송 및 기타 기능들)에 대하여 정상 작동 할 것

​	: 대부분 정상 작동 하는 것으로 확인됨



  위의 내용들을 모두 만족하는지 테스트를 진행 하였다. 

[ 테스트 결론 ]

​	. **Custom Docker Image 를 적용하면  향상되고 안정적인 Monolithic Architecture Web Source 서비스 구축**



[ 테스트 상세 내역 ]

​	. **Monolithic Architecture Web Source 실행시 안정적인 Web Service 운영**

​	: **Custom Docker Image 의 Base OS 인 " Debian 9 " 은 안정적으로 검증 받은 OS** 이므로 Monolithic Architecture Web Source 와 같이 프로젝트의 덩치가  큰 어플리케이션 구동시간 20초 내외로 구동시켜 WebApp Container Instance 의 상태 를 정상적으로 체크하여 비정상적인 강제 지시작 현상이 사라졌으며,  WebApp Container Instance 안정적인 실행 상태로 모니터링 됨

 * **Custom Docker Image 제작은 WebApp 의 Migration 프로젝트 인 경우 Custom Docker Image 제작으로 Monolithic Architecture Web Source 서비스의 최소의 구조 변경과 안정적인 환경 구성에 적합**

   

​	. **Custom Docker Image 에 WebApp User Shared Volume 1TB 끄고 사용하기**

​	: **WebApp User Shared Volume 1TB 마운트 방식은 외부 Storage 공간의 SMB 로 연동 하여 사용**하게 된다. 이때, Monolithic Architecture Web Source 를 해당 S**MB Volume 저장하여 사용  불규칙한 IOPS 로 인하여 " 400 ~ 700Mb " 의 Web Source 파일의 Deploy 작업이 매우 불안정 하게 진행됨, 따라서  Web Site 서비스가 불안정** 해지는 것이 확인 되었다.



​	. **WebApp 실시간 모니터링을 위한 Scouter Agent 설치**

​	: Custom Docker Image 사용하면서 **Azure 의 있는 모니터링 데쉬보드 로는 매우 제한적**인 상태 밖엔 그것도 5분전 등의 실시간이라고 하기에는 다소 문제가 있는 모니터링 시스템에 답답함을 느껴 직접 **실시간 모니터링 Open Source 인 Scouter Agent 를 WebApp 에 설치 하여 실시간으로 WebApp의 상태를 확인** 하게 되었다. 



**" 없으면, 만들고, 부족하면 설치 하면 되지! "** 라는 필자의 평소 지론으로 이미 이 프로젝트 시작 부터 많은 충격과 멘붕으로 왠만한 사건에는 별로 놀라지도 않게 되었으며, 충격을 받지 않을 정도의 상태까지 되었다. 



  이쯤에서 필자의 머리 속에 떠오르는 단어는 **" Case Open 의미가 있을까 ? "** 라는 생각으로 벤더 보다 더 많은 내용을 삽질로 알게 되어 이제 Case Open은 의미가 없지 않을까 ? 하는 생각이 들기 시작 했다.



  어느덧 삽질과 쇼킹으로 시작된 프로젝트 PM 역활 및 인프라 엔지니어 역활도 이미 중반을 넘고 있었다. 역시.. 삽질은 새벽에 밥먹듯이 해야 시간과 날짜가 빨리간다는 법칙이 맞는 듯 하다.

( 그래도 더이상 나빠지거나 내꺼가 안나오길 ... 오늘도 삽질.... )



PM 이건 인프라 엔지니어 이건 이미 그 경계는 의미가 없는 삽질 라이프 프로젝트를 정상적으로 끝내자.... 라는 생각뿐.. 
