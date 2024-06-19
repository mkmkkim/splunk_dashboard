![header](https://capsule-render.vercel.app/api?type=venom&color=auto&height=200&section=header&text=스플렁크를%20이용한%20공격성%20트래픽%20탐지&fontSize=40&)
---
### 1. 수행목적</br>
Splunk를 이용한 공격성 트래픽 탐지로 실무역량 강화</br>
* 수행주제
  * 시나리오1: admin페이지 내 XSS 취약점을 이용한 피싱사이트 리다이렉트</br>
  * 시나리오2: 대시보드 지표를 활용하여 DDoS공격 탐지</br>
### 2. 수행계획수립</br>
* 수행인원: 시큐리티아카데미 시큐아이트랙 Team.Splunker(4명)</br>
* 담당파트: 대시보드 개발, AES 복호화, 관제 일일보고서 제작, 발표</br>
* 수행일정(WBS)</br>
![image](https://github.com/mkmkkim/splunk_dashboard/assets/74914390/3c50415b-4257-44e3-9439-41a9afc3f7b6)</br>
* 구축 환경</br>
  * 네트워크 구성</br>
![image](https://github.com/mkmkkim/splunk_dashboard/assets/74914390/0462798d-c2ea-4e31-85f1-82e8a0db0ea7)</br>
 > 웹서버 syslog를 Forwarder를 통해 Indexer에 raw 데이터 그대로 저장하며,</br>
 > Master가 Indexer와 Searcher를 중앙에서 컨트롤하므로 분산환경 구축 시 노드 추가가 용이. </br>
  * 작업환경</br>
   * Web - CentOS7.9, GNUBOARD 5.2.6</br>
   * Splunk - CentOS7.9, Splunk 9.2.1</br>
   * AES 복호화 수행 - Anaconda Prompt, Spyder</br>
### 3. 수행과정</br>
__시나리오1 - admin 페이지 내 XSS 취약점을 이용한 피싱사이트 리다이렉트__</br>
* 참고한 페이지</br>
[AES 패스워드 복호화](https://nampill.tistory.com/entry/%ED%81%AC%EB%A1%AC-%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80-%EA%B3%84%EC%A0%95%EC%A0%95%EB%B3%B4-%EC%B6%94%EC%B6%9C%EB%B9%84%EB%B0%80%EB%B2%88%ED%98%B8-%EB%B3%B5%ED%98%B8%ED%99%94,"")</br>
[성심당 해킹](https://m.boannews.com/html/detail.html?idx=129583,"성심당%20해킹")</br>
[유명 제품 크랙버전을 위장한 인포스틸러](https://m.boannews.com/html/detail.html?idx=128626,"유명%20제품%20크랙버전을%20위장한%20인포스틸러")</br>
[인포스틸러로 브라우저 자동로그인 파일 탈취](https://m.boannews.com/html/detail.html?tab_type=1&idx=127372,"인포스틸러로%20브라우저%20자동로그인%20파일%20탈취")</br></br>
* 전개</br>
![image](https://github.com/mkmkkim/splunk_dashboard/assets/74914390/bc847e54-aa38-4a08-bafd-effaf3629ed9)</br>
_브라우저 자동로그인 파일 탈취 과정</br></br>_
![image](https://github.com/mkmkkim/splunk_dashboard/assets/74914390/2de58b5e-3514-4110-9d4b-b7166a9b8032)</br>
_트래블닷컴 관리자 계정 탈취</br>_

</br>
