## PORT와 DNS

**PORT**: 한국어로 '**항구**'라는 뜻임



![](https://lh6.googleusercontent.com/oXlLfqVhUxNvTen4DX-TZIW5cxvlHG7LIkqCh6CazkgcseCOh8KGNuJbw6PgJVEBovt9snkuBilhvt76eNsYw252HrwSiiRwUCqYNyAbd2TU2u5OGELhelJs8DIbUiCJF1fH0s1H)



**문제 상황:** 날라온 패킷이 어떤 애플리케이션에 이용되어야 하는지 알 수가 없다.







![](https://lh3.googleusercontent.com/1XhFFKbhmT2RN23OaerAfJkHDUeYk5kSZBE1SYgiDZSiM9vRixVUaJQN0DrZh58Ho-FTgcDPue7_eN6oWkCeP-QvCUx8lr9XoI7rdheS4ODWwvzMBXK8rslUnn1nF4GEzSASHlu1)



**해결법: **TCP의 **PORT 정보**를 이용









![](https://lh4.googleusercontent.com/BkgnMxdc5UuoEUdjqJ7lLTASyxbeNsIyTomgQpU_SM8e-giN02a1T2b95-ybptsa7-nad5QjXd1Lek7nVJFQjRn9kLl76f8i4BL5_2wLcj_jVl9WL70vMYYZe9a4rEqSWqhse5SF)



**IP**가 아파트 주소면 **PORT**는 몇동 몇호를 나타냄

한 아파트(PC) 안에서 사람들이 사는 집(애플리케이션)을 구분해준다.







### PORT

- 0 ~ 65535: 할당 가능
- 0 ~ 1023: 잘 알려진 포트, 사용하지 않는 것이 좋음
  - FTP: 20, 21
  - TELNET: 23
  - HTTP: 80
  - HTTPS: 443







### IP의 문제점??

- IP는 기억하기 어렵고,
- IP는 변경될 수 있다.



![](https://lh6.googleusercontent.com/MqRMAfLQzd6rt7jeZa3ITKBuFg9CUtp2gPDvOJQgvbvkeR4BKAYEfYRPmBQPNMvpYWpHi6_vaupvkpNjp91rFqf4fk58GUSzLKK9yPdffyXO_9MXZuLA65xnCWcpnarh3Eic4Jha)



![](https://lh6.googleusercontent.com/MxfIjuLoSW21EU0uF1jY07l7p1doquCLtYJNo-inyDXNsSe4A1egwAYb3s5i81PEsfzYbRLmZalRW5ouvLG7-Ha6jnE6Lged-dfsJmNxVrvEX2zku82qahj5IFs54GczGrE3jddc)



**해결:** **DNS(Domain Name System)**









### DNS 도메인 네임 시스템

- 전화번호부 같은 서버
- 도메인 명을 IP 주소로 변환해주는 서버





![](https://lh3.googleusercontent.com/H2pf9mb76jEX_TLIWHnqXvWs0OlhWRzsCp0ForDu42tNs9DK671GTlgR_km96C0A5iJCJf3e7HpGdLU5vWrSE9F0T5Y74PwBxi3S7uuViF3g-7Jku1RVYSKwmUnTlfezo-cHMWHM)