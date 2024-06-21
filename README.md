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
  * 대시보드 구축</br>
 
### 3. 시나리오 수행</br>
__시나리오1 - admin 페이지 내 XSS 취약점을 이용한 피싱사이트 리다이렉트__</br>
* 시나리오 전개</br>
1. 한글2022 크랙버전을 위장한 인포스틸러 악성코드 다운로드로 웹서버 관리자 PC에 인포스틸러 설치</br>
2. 인포스틸러로 브라우저 자동 로그인 파일이 유출되어 공격자가 관리자 계정 획득</br>
3. 사람이 적은 시간대를 노려 공격자가 관리자 페이지에 접근하여 Stored XSS로 피싱사이트 리다이렉트 코드 삽입</br>
4. 근무 외 시간에 관리자 페이지 접속 경고 메일로 관제실에서 외부 IP가 관리자 페이지 접근 사실 인지</br>
5. 메일에 첨부된 웹 로그로 공격 사실 확인하여 대응</br>
6. 익일 침해사고 관제 보고서 발송</br>
* 수행과정</br>
Python으로 크롬 자동로그인 기억 파일인 Local State 복호화(오픈소스)

* 참고한 페이지</br>
[AES 패스워드 복호화](https://nampill.tistory.com/entry/%ED%81%AC%EB%A1%AC-%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80-%EA%B3%84%EC%A0%95%EC%A0%95%EB%B3%B4-%EC%B6%94%EC%B6%9C%EB%B9%84%EB%B0%80%EB%B2%88%ED%98%B8-%EB%B3%B5%ED%98%B8%ED%99%94,"")</br>
[성심당 해킹](https://m.boannews.com/html/detail.html?idx=129583,"성심당%20해킹")</br>
[유명 제품 크랙버전을 위장한 인포스틸러](https://m.boannews.com/html/detail.html?idx=128626,"유명%20제품%20크랙버전을%20위장한%20인포스틸러")</br>
[인포스틸러로 브라우저 자동로그인 파일 탈취](https://m.boannews.com/html/detail.html?tab_type=1&idx=127372,"인포스틸러로%20브라우저%20자동로그인%20파일%20탈취")</br>
[그누보드5.2.6 취약점](https://github.com/gnuboard/gnuboard5/commit/630e39de16e61d6e0cc224028d20efb782436fba,"그누보드%20취약점")
</br>
__시나리오2 - DDoS__</br>
* 시나리오 전개</br>
1. 대시보드로 대규모 접근 시도 횟수 확인</br>
2. 동시에 경고 트리거로 인해 경고 메일 발송</br>
3. 메일로 1분동안 7,000건 이상의 접근 시도를 요청하는 IP와 요청 횟수 확인</br>
4. ISP 업체와 고객사에 IP차단요청</br>
* 수행과정</br>
