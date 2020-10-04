---
layout: post
title: "Public Cloud 전환 프로젝트 : 5부 Build 된 Custom Docker Image 적용 이후 들어나기 시작한 문제점들 (2)."
date: 2020-10-04 22:59:13
img: azure-webapp-404.png # Add image post (optional)
categories: cloud azure
tags: cloud azure MS
---


​	**필자연역 :** Linux 을 15년 이상 해왔으며, Linux 기반의 Enterprise 인프라 구축 엔지니어 생활을 8년 동안 하였으며, OpenSource 구축 및 기술지원 분야에서 다양한 경험을 하였다.

​	지금까지의 필자의 경력은 주로 On-Premis Enterprise 기업환경의 인프라 구축 및 기술 지원등의 TA 역활을 다년간 진행하였기에 Public Cloud 전환 프로젝트는 또다른 경험을 하는 기회였다.

필자의 주분야는 OpenSource WEB/WAS 분야 이며, 설계/구축/장애 기술지원에 다양한 경험을 가지고 있다.

* **프리랜서 TA 로 프로젝트를 진행 하다보니 이제서야 연재를 작성할 시간적 여유가 생겨 늦게 나마 이야기를 이어 나가고자 한다.**



#### 5. Build 된 Custom Docker Image 적용 이후 들어나기 시작한 문제점들 (2) / WebApp(PaaS) 성능의 민낯.

  **" 4부 Build 된 Custom Docker Image 적용 이후 들어나기 시작한 문제점들 (1). "** 편 발견된 인프라적 문제점들에 이어서 또 다른 문제점들이 나타나기 시작 했다.



  Custom Docker Image 로 실행되는 WebApp Instance 가 불규칙하게 재시작 되는 현상이 발견 되었다. 정확한 원인파악을 위하여 " Scouter " 의 Host Agent 까지 모두 적용 하였다.

 

1. **" 동접 4 User 의 Heap Memory 4G 사용? "**

   : 프로젝트를 수행하면서 고객 PM으로 부터 예고 없이 WebSite 를 재시작 하였냐는 급한 연락을 받아 확인해 보니 이번이 처음이 아니라 과거에도 WebApp Instance 가 불규칙 하게 빈번하게 재시작 된것을 Log 기록으로 확인 할 수 있었다.

   

   **[ WebApp Instance 재시작 원인 ]**

   . **ARR 과 WebApp Instance 간의 HTTP Request "code 200 OK" 가 30초 동안 확인 되지 않으면 강제로 해당 Web Instance 를 재시작 시킨다.**
   
   . **WebApp Instance 내의 JVM 메모리에서 이상이 발생하여 WebApp Instance 가 응답이 없을 경우에도 해당 WebApp Instance 를 강제 재시작 시킨다.**
   
   . **WebApp Instance Docker 내에 Mount 된 Storage Volume 연결에 장애 발생시 WebApp Instance 의 장애로 이어져 해당 WebApp Instance 를 강제 재시작 시킨다.**

   

   다음은 위의 장애 발생 원인들을 이해하기 쉽도록 구성도로 표현 하였다.

   **[ 장애 포인트 구성도 ]**

   ![app-error](https://github.com/graudis/graudis.github.io/blob/master/_image/ARR-architecture-2.png?raw=true)

   
   
   **[ 장애원인 추측 ]**

   재시작이 발생한 WebApp Instance 의 Log 를 추적하며, **원인을 분석중 "[ WebApp Instance 재시작 원인] " 에서의 " 2번 " Case 에 해당한다는 결론**을 내리게 되었다. 

   다음은 "[ WebApp Instance 재시작 원인] " 에서의 " 2번 " Case 로 추측 되는 자료들이다.
   
   
   
   **[ Scouter 모니터링 화면 ]**
   
   ![scouter](https://github.com/graudis/graudis.github.io/blob/master/_image/scouter-2.png?raw=true)
   
   1. **해당 WebApp Instance 의 비정상적인 JAVA Heap Memory 사용량**
   2. **높은 Heap Memory 사용량 대비 동접수가 2~4 User** 
   3. **높은 heap Memory 사용량 대비 해당 WebApp Instance 의 OS host CPU 사용량이 40% 이하 사용량**
   
   
   
   **[ Portal 화면에서의 생태 모니터링 ]**
   
   ![portal](https://github.com/graudis/graudis.github.io/blob/master/_image/portal-2.png?raw=true)
   
   
   
   **" Case 2번 " 의 발생 하는 과정을 다음과 같이 추측 해 볼수 있다.**
   
   
   
   1. **JAVA Heap Memory 의 적절한 full GC 가 적절히 진행되지 않아 Memory 임계치에 도달**
   2. **JAVA Heap Memory 의 소진과 함께 임계치에 도달하여 "OOM" 발생과 함께 장애 발생**
   3. **JAVA Heap Memory 임계치 발생 이후에 WebSite Request 요청에 Suspend 가 발생**
   4. **Web Request 에 Suspend 현상을 ARR 이 이상 징후로 판단**
   5. **ARR 에게 요청 받은 해당 WebApp Instance 를 Docker Host 에서 강재 재시작**
   
   
   
   대충 추론해 보면 **위의 5단계가 빈번 하게 발생 하여 해당 WebApp Instance 가 자동으로 재시작을 진행 한것으로 추측** 된다.
   
   
   
   **[ 해결 방법 ]**
   
   . **결론을 말하자면, 다른 Platform 으로 이관하지 않는 이상 벤더가 해결 해주 전까지는 해결 방법은 없다.**
   
   
   
   **[ 결과 ]**
   
   . **WebApp Instance 의 " Scale Out" 으로 2중화 시켜 장애 발생시 다른 하나로 운영이 되는 구조로 변경** 하였다. 물론 기존 WebApp Instance 를 1개만 운영 하던 비용대비 비용은 2배로 늘어 나는 결과를 초래 하게 되었다. 여기서 이미 PaaS 의 저비용 에 대한 메리트는 사라진지 오래이고 불편함만 더 늘어 나게 되었다.
   
   

​       **[ 악연의 시작 3 ]**

1. **2018년 11월에 첫 GA 된 Linux WebApp 서비스를 국내 첫 도입한 사례로 기록됨**

   : 벤더사가 고객에게 제안한 PaaS 서비스

2. **동접사용자가 2~4 User 이고 해당 Instance 내의 OS CPU 사용량이 40% 이하 사용량**

   ![cpu](https://github.com/graudis/graudis.github.io/blob/master/_image/scouter-cpu-1.png?raw=true)

   : 동시 접속자가 2~4 User 및 WebApp Instance 내의 OS 의서의 CPU 사용량이 40% 미만

   

3. **WebApp Instance 내의 이유를 알수 없는 JAVA Heap Memory 의 높은 사용량**

   ![heap](https://github.com/graudis/graudis.github.io/blob/master/_image/scouter-heap-1.png?raw=true)

   : 서비스 사용자가 많지 않은 상황에서의 비정상적이고 불안정한  Memory 사용량 발견

   동일한 Monolithic Architecture Web Source 를 적용한 WebApp Instance #1 과 #2 가 각기다른 Heap Momory 사용량을 보이는 이상 증세 발견

   

4. **동일 Web Source 를 사용하는 WebApp Instance 의 불안정한 Request 처리 현상으로 알게된 불안정한 "Loadbalance"**  

   ![tps](https://github.com/graudis/graudis.github.io/blob/master/_image/scouter-tps-1.png?raw=true)

   : WebApp 앞단에 존재하는 **Loadbalance 장비의 SLB 모드가 " Hash " 로 되어 있서 비대칭 적으로 처리 될 수도 있다.** 라고 말할 수도 있다. 

   맞다. SLB 모드가 " Hash " 모드 로 되어 있다면 당연히 비대칭 으로 처리 된다. 

   하지만, Jmeter 로 **부하 테스트를 진행 할때**에도 비대칭으로 처리 하는 결과를 보게 되었다. **SLB 모드가 " Hash " 이라면 2개의 WebApp Instance 중 한쪽에서만 Request 처리를 해야 " Hash" 모드로 처리 하는 것이 맞다고 볼수** 있으나, 

   **결과는 랜덤하게 비대칭으로 2개의  WebApp Instance 에서 각각 처리 하고 있었다.** 

   ( 뭐지? 그럼 SLB 모드는 뭘로 되어 있다는 거지? 아놔.... )



   **[ PaaS 에 대한 필자의 견 ]**

1. **PaaS 는 필요한가 ?**

   : 비즈니스 모델에서의 사용자 접속 요구 범위가 예측 불가능 할 경우 필요에 따라 서비스 접속 요구량을 가변적으로 운영 할 수 있는 장점과 인프라 구축 비용이 별도로 소요 되지 않는 장점이 있다. 따라서, PaaS 서비스는 향후 매우 중요한 Platform 으로 주목 받고 있다.


2. **On-Premise 서비스는 PaaS 로 Migration 이 가능한가 ?**

   : MSA 구조로 되어 있다면 가능하나 **Monolithic Architecture Web Source 라면 Migration 하지 않는 것이 정신 건강에 이롭다.** 
   ( 필자와 같이 어쩔수 없이 PaaS Migration 해야 한다면.... 그냥 하지 말것을 권유 한다. 본인의 정신건강을 학대 하고 싶다면 해도 상관없다. )


3. **PaaS 로 구축하면 TCO 가 절감된다 ?**

   : 신규 구축 프로젝트 혹은 Migration 프로젝트의 성격에 따라 TCO 가 더 높게 나올수 있다. 다양한 인프라 구축 및 지원 경험이 있는 인프라 엔지니어와 협의 하에 신중히 설계 해야 한다. 절대로 벤더사 영업및 Cloud 경력만 있는 엔지니어의  말만 의존해서는 안된다.


4. **PaaS 구축시 DevOps 는 필수 인가 ?**

   : PaaS 구축시 DevOps 의 CI/CD 를 이용하는 것이 훨씬더 생산성이 높다. DevOps 는 이제 기본이다. 

   

   **PaaS 의 구조를 알면 알수록 MSA 구조가 아니면 의미가 없거나 혹은 강제로 Migration 하여도 불안정한 Platform 서비스로 인하여 불안감을 가지고 운영할 필요가 있을까** 하는 생각이 점점 들기 시작 했다. 

( **타 벤더사의 PaaS 서비스들도 현재의 PaaS 서비스와 비슷한 구조와 비슷한 서비스를 할 것 같다는 생각이 점점 들기 시작하였다.**  )

 

== 6부 에서 계속 ==
