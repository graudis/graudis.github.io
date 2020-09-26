---
layout: post
title: "Public Cloud 전환 프로젝트 : 2부 WebApp(PaaS) 삽질하다."
date: 2020-05-03 12:59:13
img: azure-webapp-404.png # Add image post (optional)
categories: cloud azure
tags: cloud azure MS
---

​	**필자연역 :** Linux 을 15년 이상 해왔으며, Linux 기반의 Enterprise 인프라 구축 엔지니어 생활을 8년 동안 하였으며, OpenSource 구축 및 기술지원 분야에서 다양한 경험을 하였다.

​	지금까지의 필자의 경력은 주로 On-Premise Enterprise 기업환경의 인프라 구축 및 기술 지원등의 TA 역활을 다년간 진행하였기에 Public Cloud 전환 프로젝트는 또다른 경험을 하는 기회였다.

필자의 주분야는 OpenSource WEB/WAS 분야 이며, 설계/구축/장애 기술지원에 다양한 경험을 가지고 있다.



#### 2. Public Cloud 전환 프로젝트 : 2부 WebApp(PaaS) 삽질하다.

기존의 Cloud 업체들의 Cloud 기반의 인프라 구축은 PaaS( DB 부분은 제외) 서비스 보다는 대부분이 IaaS(VM) 서비스 구축을 많이 하게 된다. 

  이는 기존 On-Premise 환경의 물리서버( 혹은 VM )  를 Cloud 로 이관 할때 PaaS 서비스 환경 대비 난이도가 그리 높지 않으며, Cloud 전문 지식이 없더라도 이관 구축 작업이 용이 하다는 필자의 생각이다.

  위에서 언급한 내용은 나중에 따로 자세히 설명 하기로 하겠다.

이번 **" Public Cloud 전환 프로젝트 " 의 Cloud Platform 은 MS Azure 이며, WEB/WAS 부분은 IaaS 가 아닌 Azure 에서 제공되는 WebApp(PaaS) 서비스 로 구축하는 것이 미션**이였다.

 통상적인 IaaS 서비스 구축이 아닌 **PaaS + IaaS 서비스가 혼합된 Public Cloud 전환 프로젝트**라 하겠다. 

 일단, WEB/WAS 부분을 MS Azure WebApp(PaaS) 서비스로 구연 해야 하는 것이 당장의 미션 이므로, MS Azure WebApp(PaaS) 서비스의 기본 스펙을 다음과 같이 정리 하였다.

1. **MS Azure WebApp(PaaS/Linux)**

   ![Azure WebApp](https://github.com/graudis/graudis.github.io/blob/master/_image/webapp-1.png?raw=true)

   [ WebSSH 접속 ]

   ![WebSSH](https://github.com/graudis/graudis.github.io/blob/master/_image/webapp-2.png?raw=true)

   | 항목                  | 내용                             |
   | --------------------- | -------------------------------- |
   | Container OS          | Alpine Linux                     |
   | Run Type              | Docker Container                 |
   | WebApp WAS            | Tomcat 8.5 / 9                   |
   | JAVA 지원             | OpenJDK Zulu 8/11                |
   | Web Source Volume     | 1TB( Shared Volume / SMB Mount ) |
   | User Data Mount Point | "/home" ( Web Source Volume )    |



2. **MS Azure WebApp 특징**

   . **Container Instance 강제 재시작** : WebApp Container 실행 후 Max 30초 내에 해당 Container Instance의 **HTTP 헤더값이 " 200 "** 이 회신되지 않을 경우 해당 WebApp Container Instance를 강제 재시작함

   

   . **User Data Volume 1TB** : WebApp 에서 사용되는 User Web Page Source 파일들 및 Log / Data 등을 저장하는 Disk Volume을 별도로 제공. Docker Container 특성상 소거 되지 않는 Data Volume 이 필요함.

   ![User Data](https://github.com/graudis/graudis.github.io/blob/master/_image/webapp-4.png?raw=true)



3. **Alpine Linux(MS Azure WebApp) 에 Monolithic Architecture Web Source 테스트**

​	필자는 MS Azure WebApp 에서 Built-In 되어 제공되는 Linux WebApp(Alpine Linux)에 AS-IS 에서 사용하고 있는 Monolithic Architecture Web Source 를 적용하기 위하여 테스트를 진행하였다.

​    다음은 고객의 Monolithic Architecture Web Source 테스트 전 마이그레이션 요구 사항들이다.

​	. AS-IS 의 Monolithic Architecture Web Source 구조를 유지 할 것 ( 최소 변경 )

​	. 1개 WebApp Container Instance 에서 전체 Web Source 가 실행 될 것 ( MSA 구조가 아님 )

​	. Intro Main Page 의 Page Loading Time 이 1 Sec 전후 일 것 ( AS-IS 와 같거나 빠른 응답속도 )

​	. 내부 기능들( 파일 전송 및 기타 기능들)에 대하여 정상 작동 할 것



  위의 내용들을 모두 만족하는지 테스트를 진행 하였다. 

[ 테스트 결론 ]

​	. **Azure 의 Built-In 되어 제공되는 WebApp Container 에서는 AS-IS 의 Monolithic Architecture Web Source 서비스 구축 불가**



[ 테스트 상세 내역 ]

​	. **Monolithic Architecture Web Source 실행시 잦은 강제 재시작 현상 발생**

​	: Monolithic Architecture Web Source 와 같이 프로젝트의 덩치가 너무 커져서 어플리케이션 구동시간 30초 이상이 넘어 가는 경우가 발생 하여 WebApp Container Instance 의 상태 비정상 으로 판단 WebApp Container Instance 의 강제 재시작 현상 발생

 * WebApp 서비스는 MSA 구조를 기반으로 하는  분산 WebApp 서비스 구성에 적합

   

​	. **Alpine Linux 에서의 Glibc 를 기반으로하는 Util 및 3rd Party 제품 사용불가** 

​	: Web Source 내에의 서비스 기능중 Linux OS 제공 Util 및 3rd Party 제품의 Source 컴파일 및 Binary 설치 할 경우 Glibc 와 의존성에서 문제가 발생 하였다. Alpine Linux 는 Glibc 대신 musl libc 를 사용하며, 다양한 쉘 명령어는 GNU util 대신 busybox 를 탑재 하여 사용하기 때문에 Glibc 로 컴파일된 Binary 파일들에서 에러가 발생하였다.



  필자는 벤더에서 Built-In 하여 제공하는 Linux WebApp Container 스펙과 마이그레이션 대상의 Monolithic Architecture Web Source 적용에 에러가 발생되는 것이 확인 되어 심한 충격에 빠졌다. 



  위의 장애 발생에 대하여 벤더사에 " Case Open " 하는 것과 동시에 WebApp 서비스에 대한 유사 케이스를 확인 요청 하였다. 1차 회신을 받고 나서 2차 멘붕에 정신이 혼미해 짐을 느꼈다.



벤더의 답변 : **" 요청하신 Linux WebApp 서비스는 최근에 GA 된 제품이므로 Case Open 된 사례가 거의 없어 정확한 원인을 답변하기 어렵다. "** 라는 답변



  필자의 머리 속에 떠오르는 단어는 **" 마.루.타 "** 더이상의  다른 단어로는 표현은 의미가 없다는 느낌이 들어 한 동안 멍때릴 수 밖엔 없었다. 



  프로젝트 PM 으로써 어떻게 하던 프로젝트는 정상 진행 하고 완료 해야 하기에 어쩔수 없이 Custom Docker 삽질을 할 수 밖엔 없다는 결론에 도달 하였다.

( 그렇다면, Custom Docker Container Build 작업은 인프라 엔지니어 아니면 누가?.... 

언제나 슬픈 예감은 틀린적이 없다. 인프라 엔지니어가 Linux 을 포함한 Docker build를 할줄 모른다고 한다.. 충격.. 에효... 또.. 내꺼구나... 제길.... )



점점 필자는 본인의 스콥에 대한 정체성을 잃어가는...... PM 하라고 하더니만... 
