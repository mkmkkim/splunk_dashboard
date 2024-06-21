![header](https://capsule-render.vercel.app/api?type=venom&color=auto&height=200&section=header&text=스플렁크를%20이용한%20공격성%20트래픽%20탐지&fontSize=40&)
---
### 1. 수행목적</br>
Splunk를 이용한 공격성 트래픽 탐지로 실무역량 강화</br>
* 수행주제
  * 시나리오1: admin페이지 내 XSS 취약점을 이용한 피싱사이트 리다이렉트</br>
  * 시나리오2: 대시보드 지표를 활용하여 DDoS공격 탐지</br>
### 2. 수행계획수립</br>
* 수행인원: 시큐리티아카데미 시큐아이트랙 Team.Splunker(4명)</br>
* 담당파트: 대시보드 레이아웃 및 쿼리 개발, AES 복호화, 관제 일일보고서 제작, 발표</br>
* 수행일정(WBS)</br>
![image](https://github.com/mkmkkim/splunk_dashboard/assets/74914390/3c50415b-4257-44e3-9439-41a9afc3f7b6)</br>
* 아키텍처</br>
  * 웹서버 syslog를 Forwarder를 통해 Indexer에 raw 데이터 그대로 저장하며, Searcher는 Master를 통해 Indexer에 있는 데이터를 SPL로 검색</br>
  * Master가 Indexer와 Searcher를 중앙에서 컨트롤하므로 분산환경 구축 시 노드 추가가 용이 </br>
* 네트워크</br>

 | Name  | IP | Port Forwarding |
 |:---------:|:------------:|:------:|
 | Webserver | 192.168.1.16 | 11112 |
 | Forwarder | 192.168.1.11 | 11111 |
 | Indexer | 192.168.1.12 | 22222 |
 | Searcher | 192.168.1.13 | 33333 |
 | Master | 192.168.1.14 | 44444 |

_VMWarePro에서 NAT모드 192.168.1.0대역으로 네트워크 구성_
</br>
* 실습환경</br>
  * Web - CentOS7.9, GNUBOARD 5.2.6</br>
  * Splunk - CentOS7.9, Splunk 9.2.1</br>
  * AES 복호화 수행 - Anaconda Prompt, Spyder</br>
* 대시보드 구축</br>
  * 실시간으로 변하는 대시보드를 구상하였으나, 웹 공격 유입량 테이블 검색 속도가 느려 1분 간격 새로고침으로 설정</br>
  * JOIN을 사용하면 검색 시간이 길어지므로 최대한 사용을 지양하고 최적화할 수 있는 쿼리를 고민</br>

**실시간 접근 시도(1분) 수집**</br>
① 'index=webserver log_type=httpd* |stats count by log_type' 쿼리로는 log_type이 httpd:, httpd.service로 결과값이 여러 개였기에 count를 한 번 더 더하고 Traffic으로 이름 변경함.</br>

<details>
  <summary>쿼리</summary>
  <img src="./assets/query1" width=70%>
</details>

**웹서버 리소스 사용량(1분) 수집**</br>
① 웹서버에 Splunk 설치 후 Splunk Add-on for Linux 앱 설치</br>
② 쉘 스크립트로 실시간으로 명령어 결과값을 Indexer로 전달(CPU는 top, Memory는 free, Disk는 df), Index=resource로 수집</br>
③ Searcher 대시보드에 Horseshoe meter 앱 설치 후, 아래 쿼리를 입력하여 리소스 사용량 출력
```
<query>index=resource sourcetype=cpu | stats avg(cpu_usage) as CPU | eval CPU=round(CPU, 1)</query>
<earliest>-1m@m</earliest>
<latest>now</latest>
``` 
**웹 공격 유입량(1시간) 수집**</br>
① Nikto, Nessus를 웹 서버에 사용 후 웹 공격</br>
② 스캐너가 남긴 웹 공격 시그니처를 바탕으로 쿼리 제작
```
index=webserver src_ip!="*:*" method=*unionselect* OR *select*from* OR *insertinto* OR *deletefrom* OR *droptable* OR *'or1* OR "*%27or%201*"
|stats count by src_ip | rename count as "SQL Injection"
|appendcols
[search earliest=-1h@h latest=now
index=webserver src_ip!="*:*" method=*../*
|stats count by src_ip | rename count as "Path Traversal(../)"]
|appendcols
[search earliest=-1h@h latest=now
index=webserver src_ip!="*:*" method="*script>*" OR method="*onerror*" OR "*iframe*"
|stats count by src_ip | rename count as "XSS"]
|appendcols
[search earliest=-1h@h latest=now
index=webserver src_ip!="*:*" method=*/etc/* =
|stats count by src_ip | rename count as "Broken Access Control"]
|appendcols
[search earliest=-1h@h latest=now
index=webserver src_ip!="*:*" method=*cgi*
|stats count by src_ip | rename count as "Shell Shock"]
|appendcols
[search earliest=-1h@h latest=now
index=webserver src_ip!="*:*" method="*/adm*" status="302" OR status="200"
|stats count by src_ip | rename count as "Admin Page Access"]
```
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
**Python으로 크롬 자동로그인 파일(Local State) 복호화_오픈소스 활용**</br>
**그누보드5.2.6 버전 취약점 검색 후 최신 보안 기사를 레퍼런스로 시나리오 기획**</br>
* 참고한 페이지</br>
[(기사)성심당 해킹](https://m.boannews.com/html/detail.html?idx=129583,"성심당%20해킹")</br>
[(기사)유명 제품 크랙버전을 위장한 인포스틸러](https://m.boannews.com/html/detail.html?idx=128626,"유명%20제품%20크랙버전을%20위장한%20인포스틸러")</br>
[(기사)인포스틸러로 브라우저 자동로그인 파일 탈취](https://m.boannews.com/html/detail.html?tab_type=1&idx=127372,"인포스틸러로%20브라우저%20자동로그인%20파일%20탈취")</br>
[AES 패스워드 복호화](https://nampill.tistory.com/entry/%ED%81%AC%EB%A1%AC-%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80-%EA%B3%84%EC%A0%95%EC%A0%95%EB%B3%B4-%EC%B6%94%EC%B6%9C%EB%B9%84%EB%B0%80%EB%B2%88%ED%98%B8-%EB%B3%B5%ED%98%B8%ED%99%94,"")</br>
[그누보드5.2.6 취약점](https://github.com/gnuboard/gnuboard5/commit/630e39de16e61d6e0cc224028d20efb782436fba,"그누보드%20취약점")</br></br>
__시나리오2 - DDoS__</br>
* 시나리오 전개</br>
1. 대시보드로 대규모 접근 시도 횟수 확인</br>
2. 동시에 경고 트리거로 인해 경고 메일 발송</br>
3. 메일로 1분동안 7,000건 이상의 접근 시도를 요청하는 IP와 요청 횟수 확인</br>
4. ISP 업체와 고객사에 IP차단요청</br>
* 수행과정</br>
