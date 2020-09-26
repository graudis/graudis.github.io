---
layout: post
title: "Public Cloud 전환 프로젝트 : 1부 The Beginning ( 삽질의 서막 )"
date: 2020-04-16 12:59:13
img: azure-webapp-404.png # Add image post (optional)
categories: cloud azure
tags: cloud azure MS
---
[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2Fgraudis%2Fgraudis.github.io%2Fblob%2Fmaster%2F_posts%2F2020-04-16-azure-webapp-1.md&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)

​	**필자연역 :** Linux 을 15년 이상 해왔으며, Linux 기반의 Enterprise 인프라 구축 엔지니어 생활을 8년 동안 하였으며, OpenSource 구축 및 기술지원 분야에서 다양한 경험을 하였다.

​	지금까지의 필자의 경력은 주로 On-Premise Enterprise 기업환경의 인프라 구축 및 기술 지원등의 TA 역활을 다년간 진행하였기에 Public Cloud 전환 프로젝트는 또다른 경험을 하는 기회였다.

필자의 주분야는 OpenSource WEB/WAS 분야 이며, 설계/구축/장애 기술지원에 다양한 경험을 가지고 있다.



1. #### Public Cloud 전환 프로젝트 : 1부 The Beginning ( 삽질의 서막 )

2019년 On-Premise 에 있는 WEB 서비스를 Public Cloud 로의 전환 Migration 프로젝트의 PM을
맏게 되었다.

​	지금까지의 필자의 경력은 주로 On-Premise 에서의 인프라 구축 및 기술 지원등의 TA 역활을 다년간 진행하였기에 Public Cloud 전환 프로젝트는 또다른 경험을 하는 기회였다.
필자의 그당시 심정은 한마디로 다음과 같았다.

**" 재미 있겠다.  " 라는 생각이였다.**



​	필자는 사전 조사 및 설계 분석 단계 부터 참여 하지 못하고 프로젝트 시작과 함께 투입되었다.  필자는 Public Cloud 전환 프로젝트 또한 On-Premise 구축 프로젝트의 SI 공정과 기본 작업은 동일하다는 판단을 기반으로 이미 사전에 분석 설계 되어 있는 자료등 여러 자료들을 영업팀에게 요청 하였다.

​	다음은 필자가 점검 사항에 대한 요구 사항들이다.



| [ 요청 자료 ]                                                |
| :----------------------------------------------------------- |
| **1. 해당 On-Premise 서비스 와 전환 대상 Pulic Cloud Platform 간의 호환성 검토 자료** |
| : **AS-IS 의 서비스 환경 과 TO-BE 환경간의 호환이 높을 수록 프로젝트 성공률이 높기 때문** |
|                                                              |
| **2. 전환시 고객의 추가 요청 사항 자료 ( JDK 및 Spring Framework 등의 라이브러리 버전 Up 및 부가적인 Build 환경 구성 등등 )** |
| : **TO-BE 환경으로의 전환시 고객의 추가 요청사항을 수용 할 수 있는지 가능성 여부 및 버전 Up 으로 인한 Risk 발생 요인 분석** |
|                                                              |
| **3. 해당 Pulic Cloud Platform 의 지원 가능한 부분 및 성능 부분 자료 ( Storage 부분 및 보안 관련 부분 )** |
| : **TO-BE 환경으로 전환시 지원되지 않는 AS-IS 의 일부 기능에 대한 대체 방안 검토** |



​	혹자 위의 사항들을 보고 **" 너무 오버 아니야 ? "** 혹은 **" 당연히 해야 할 일 "** 라는 말을 하시는 분들도 있을 것이다. 
하지만, **" 당연히 해야 할 일 "** 자체가 **전혀 안되고 있는 것이 현실** 라는 걸 깨닫게 되는데는 그리 오랜 시간이 걸리지 않았다.

​	위의 점검 사항들에 관하여 최대한 사전 분석과 설계를 진행한 협력업처 및 본사 담당 영업에게 문의 하였으나, **사전 분석한 업체와 벤더사의 답변은 매우 간결 했다.** 

**" 벤더의 답변 : 됩니다. 가능 합니다. "** 

​	원하는 자료는 오지 않고 간결하고 심플한 위의 답변이 왔다. 뭔가 좀 이상하다는 느낌은 들었으나, 벤더의 답변이므로 일단 믿고 진행 하기로 하였다.
이런 벤더의 말이 나중에 엄청난 결과를 낳게 된다는 것을 그때는 알지 못했었다.



결국 고객과의 많은 회의로 인하여 현재 진행 하는 " Public Cloud 전환 프로젝트" 의 작업 영역을 알수 있었다.

다음은 대략적인 Public Cloud 전환 프로젝트의 내용이다.



| AS-IS( On-Premise )    |      |     TO-BE( Cloud )     |
| :--------------------- | ---- | :--------------------: |
| OS : UNIX              | ->   |       Linux PaaS       |
| WEB ( IHS : Apache )   | ->   |        ARR(IIS)        |
| WAS ( WebSphare )      | ->   |    WebApp( Tomcat )    |
| Spring Framework 3.2.7 | ->   | Spring Framework 3.2.7 |
| JDK 1.4                | ->   |        JDK 1.8         |
| Storage( Image )       | ->   |       Azure BLOB       |
| Storage( Contents )    | ->   |       Azure File       |
| 형상관리( 없음 )        | ->   |      Azure DevOps      |



위의 내용이 파악되고서는 멘붕이 왔다. 과연 이 프로젝트 잘될까? 라는..
